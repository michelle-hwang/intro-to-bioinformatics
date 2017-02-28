---
title: Assembly and Assessment Lab
categories:
  - lab
---

# Transcriptomics Lab

**TIPS:**
* **Google is a programmer's best friend.**
* **The best way to learn is to explore and figure out how to accomplish tasks on your own.**
* **There are manuals for every command or software online.**

<br>

Today we will be assembling a transcriptome and assessing its assembly quality via several different strategies. Please login to your Sapelo computing cluster account, start an interactive job, and create a new folder ```transcriptomics-lab```.  Go into this folder and then we can begin!

* Q1. Why do we need to first start with an interactive job on the cluster?

## Data Acquisition

We will be looking at Arabidopsis thaliana data because it has a smaller genome and will take less time in computational processes. The [data](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=SRP063471) is a **single-end** Illumina sequencing set with samples treated under salt stress, heat stress, and both. To simplify this, will only download one replicate each of the control samples and the heat stress samples. Each sample is about 3.5GB. You can download the SRA files from NCBI using sratoolkit:

```
module load sratoolkit

for f in SRR2302908 SRR2302909 SRR2302914 SRR2302915; do
	fastq-dump $f
done
```
This will take some time, since the files are large. They are currently in *.fastq* format. The first two files are the control samples, and the last two files are the samples under heat stress. Please rename the files to: "Ctrl" and "Heat".

* Q2. What command did you use to rename the files?
* Q3. What is the difference between *.fastq* and *.fasta* format?
* Q4. How do we determine how many sequences are in each file? How many sequences are in each file?

## Transcriptome Assembly

### Assembly Preparation

Before assembling, the adapters must be removed from sequences. We will use [trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) software to do this. Please visit the GACRC Wiki page for Trimmomatic [here](https://wiki.gacrc.uga.edu/wiki/Trimmomatic-Sapelo) to figure out what module(s) must be loaded to run this software.

```
time java -jar /usr/local/apps/trimmomatic/0.33/trimmomatic-0.33.jar SE -threads 4 
```

### Running the Assembly

We will be using [Trinity](https://github.com/trinityrnaseq/trinityrnaseq/wiki) to generate a transcriptome assembly. 


## Assembly Assessment

During lecture I introduced three different strategies to assess the quality of a transcriptome assembly. 

Strategy 1 | Strategy 2 | Strategy 3
--- | --- | ---
Alignment | TransRate | FASTQC

