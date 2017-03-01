---
title: Assembly and Assessment Lab
categories:
  - lab
---

# Transcriptomics Lab

<img src="https://cdn.brainpop.com/health/geneticsgrowthanddevelopment/rna/screenshot1.png" width=400>

**TIPS:**
* **Google is a programmer's best friend.**
* **The best way to learn is to explore and figure out how to accomplish tasks on your own.**
* **There are manuals for every command or software online.**

<br>

Today we will be assembling a transcriptome and assessing its assembly quality via several different strategies. Please login to your Sapelo computing cluster account, start an interactive job, and create a new folder ```transcriptomics-lab```.  Go into this folder and then we can begin!

* Q1. Why do we need to first start with an interactive job on the cluster?

<br>

## Data Acquisition

<img src="https://www.kuleuven-kulak.be/kulakbiocampus/lage%20planten/Arabidopsis%20thaliana%20-%20Zandraket/Arabidopsis_thaliana-zandraket02.jpg" width=400>

We will be looking at Arabidopsis thaliana data because it has a smaller genome and will take less time in computational processes. The [data](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=SRP063471) is a **single-end** Illumina sequencing set with leaf tissue samples treated under salt stress, heat stress, and both. To simplify this, will only download one replicate each of the control samples and the heat stress samples. Each sample is about 3.5GB. You can download the SRA files from NCBI using sratoolkit:

```
module load sratoolkit

for f in SRR2302908 SRR2302914; do
	fastq-dump $f
done
```
This will take some time, since the files are large. They are currently in *.fastq* format. The first two files are the control samples, and the last two files are the samples under heat stress. Please rename the files to: "Ctrl" and "Heat".

* Q2. What command did you use to rename the files?
* Q3. What is the difference between *.fastq* and *.fasta* format?
* Q4. How do we determine how many sequences are in each file? How many sequences are in each file?

<br>

## Transcriptome Assembly

### Assembly Preparation

#### Adapter Removal and Quality Trimming

Before assembling, the adapters must be removed from sequences. A popular program used for this is [trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) or [cutadapt](https://cutadapt.readthedocs.io/en/stable/), which are both available on the comuputing cluster. Our data set is already trimmed, so we can skip this step.

#### Removal of Artifacts



### Running the Assembly

We will be using [Trinity](https://github.com/trinityrnaseq/trinityrnaseq/wiki) to generate a transcriptome assembly. You can find instructions on how to use Trinity on Sapelo [here](https://wiki.gacrc.uga.edu/wiki/Trinity-Sapelo). 

The assembly itself may take several hours, so we want to submit a shell script to the computing cluster queue instead of having it run interactively. Create a shell script file ```run_trinity.sh```.

```
#!/bin/bash

#PBS -N trinity
#PBS -q batch
#PBS -l nodes=1:ppn=16:HIGHMEM
#PBS -l walltime=480:00:00
#PBS -l mem=100gb

cd $PBS_O_WORKDIR
module load trinity/2.0.6-UGA      
time Trinity --seqType fq --max_memory 100G --CPU 12 --normalize_reads --normalize_by_read_set --output Trinity --single Ctrl.fq,Heat.fq 1>trinity.out 2>trinity.err 
```

<br>


## Assembly Assessment

During lecture I introduced three different strategies to assess the quality of a transcriptome assembly. 

Strategy 1 | Strategy 2 | Strategy 3
--- | --- | ---
Alignment | TransRate | FASTQC

### Strategy 1 - Alignment

```
#!/bin/bash

#PBS -N AnE
#PBS -q batch
#PBS -l nodes=1:ppn=8:HIGHMEM
#PBS -l walltime=480:00:00
#PBS -l mem=100gb

cd $PBS_O_WORKDIR
module load trinity/2.0.6-UGA     
align_and_estimate_abundance.pl --transcripts Trinity.fasta --seqType fq --est_method RSEM --output_dir AnE --aln_method bowtie2 --thread_count 8 --trinity_mode --prep_reference -single READS
```

### Strategy 2 - TransRate

```
#!/bin/bash

#PBS -N transrate
#PBS -q batch
#PBS -l nodes=1:ppn=4
#PBS -l walltime=480:00:00

cd $PBS_O_WORKDIR
module load transrate/1.0.1
time transrate --threads=4 --assembly=Trinity.fasta --output=raw-transrate
```

### Strategy 3 - FASTQC

```
#!/bin/bash
#PBS -q fastqc
#PBS -l nodes=1:ppn=1:AMD
#PBS -l walltime=480:00:00
#PBS -l mem=80gb

cd $PBS_O_WORKDIR
module load java/jdk1.8.0_20 fastqc
time fastqc Trinity.fasta
```

The analysis will spit out a .html file and a .zip file. In order to view the .html file, you must transfer it back to your own computer and double click it. 

* Q. Why is there a heavy initial bias in kmer content near the beginning of the reads?

<br>

## Resources



