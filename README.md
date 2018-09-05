# Colaptes_genome
Genome assembly with PacBio, Illumina long-linked (chromium 10x) and Hi-C reads 

Objective: Assemble higher quality genome by mapping Illumina and Hi-C reads to SMRT (PacBio) data. 

All steps run on TTU HPCC Quanah cluster unless otherwise specified. 

Step 1 : PacBio data assembled using Canu

/lustre/work/jmanthey/canu-1.7.1/Linux-amd64/bin/canu -p colaptes -d colaptes-pb genomeSize=1600000000 -pacbio-raw 01_input/colaptes_part?.pb.fasta gnuplotTested=true gridEngineMemoryOption="-l h_vmem=MEMORY" gridEngineThreadsOption="-pe sm THREADS" gridOptions="-P quanah -q omni" correctedErrorRate=0.065 corMhapSensitivity=normal
 
Output saved as colaptes.contigs.fasta.

Step 2 : Clean raw chromium 10x data using bbduk

module load java

/lustre/work/johruska/bbmap/bbduk.sh in1=/lustre/scratch/johruska/colaptes/chromium_10x/raw/5469-JDM-0003_S1_L001_R1_001.fastq.gz in2=/lustre/scratch/johruska/colaptes/chromium_10x/raw/5469-JDM-0003_S1_L001_R2_001.fastq.gz out1=/lustre/scratch/johruska/colaptes/chr$

Step 3: Index PacBio genome using bwa, samtools and gatk.  

module load intel bwa samtools java

cd /lustre/scratch/johruska/colaptes/canu_assembly/
bwa index colaptes.contigs.fasta
samtools faidx colaptes.contigs.fasta
/lustre/work/johruska/gatk-4.0.8.1/gatk --java-options "-Xmx10g" CreateSequenceDictionary -R colaptes.contigs.fasta

Step 4: Divide up Chromium 10x cleaned data into 5 separate files of equal size (for R1 AND R2). Original size of cleaned chromium 10x fastq files was 139 GB. Initially, bwa jobs run on these files timed out on the HPCC TTU cluster (48 hr walltime). 

split -n 5 5469-JDM-0003_S1_L001_R1_001.fastq

split -n 5 5469-JDM-0003_S1_L001_R2_002.fastq


