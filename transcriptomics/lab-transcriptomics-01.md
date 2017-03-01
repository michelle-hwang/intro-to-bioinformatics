---
title: Assembly and Assessment Lab
categories:
  - lab
---

# Transcriptomics Lab

<img src="https://cdn.brainpop.com/health/geneticsgrowthanddevelopment/rna/screenshot1.png" width=400>

**TIPS:**
* **THIS is a FANTASTIC guide with explanations for every step and consideration: https://en.wikibooks.org/wiki/Next_Generation_Sequencing_(NGS)**
* **Google is a programmer's best friend.**
* **The best way to learn is to explore and figure out how to accomplish tasks on your own.**
* **There are manuals for every command or software online.**
* **WARNING: DO NOT COPY/PASTE COMMANDS. One, there are invisible characters that can get copied, such as newlines or spaces, and can greatly mess up your command. Two, it's better to learn when you are typing out the command for yourself.**

<br>

Today we will be assembling a transcriptome and assessing its assembly quality via several different strategies. Please login to your Sapelo computing cluster account, start an interactive job, and create a new folder ```transcriptomics-lab```.  Go into this folder and then we can begin!

* Q1. *Why do we need to first start with an interactive job on the cluster?*

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

* *Q2. What command did you use to rename the files?*
* Q3. *What is the difference between *.fastq* and *.fasta* format?*
* Q4. *How do we determine how many sequences are in each file? How many sequences are in each file?*

<br>

## Transcriptome Assembly

The original study that generated this data generated an assembly by aligning to the Arabidopsis genome as a reference. If a high quality referene genome is available, it is advantageous to utilize the reference when creating an assembly. Today, we will be assembling an assembly de novo without a reference. 

* *(BONUS) Q5. It is advantageous to use a reference genome to generate a transcriptome assembly, but what can a de novo assembly reveal that a reference assembly cannot?*

<br>

### Assembly Preparation

#### Assessing Sequencing Data

FASTQC

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

* *Q. Why is there a heavy initial bias in kmer content near the beginning of the reads?*

#### Adapter Removal and Quality Trimming

Before assembling, the adapters must be removed from sequences. A popular program used for this is [trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) or [cutadapt](https://cutadapt.readthedocs.io/en/stable/), which are both available on the comuputing cluster. Each has extensive documentation so there are plenty of nice instructions to guide you through. Our data set is already trimmed, so we can skip this step.

### Running the Assembly

We will be using [Trinity](https://github.com/trinityrnaseq/trinityrnaseq/wiki) to generate a transcriptome assembly, which is an extremely popular softare for RNA-Seq assembly and analysis with extensive tools and documentation. You can find instructions on how to use Trinity on Sapelo [here](https://wiki.gacrc.uga.edu/wiki/Trinity-Sapelo). 

The assembly itself may take several hours, so we want to submit a shell script to the computing cluster queue instead of having it run interactively. Create a shell script file ```run_trinity.sh```.

#### NOTE: Normalization

As discussed in lecture, RNA-Seq experiences a unique problem over DNA sequencing because there is variation in expression of transcripts. Therefore, there will be the existence of lowly expressed/sequenced transcripts alongside very highly expressed/sequenced transcripts, which can cause a problem when estimating expression. This becomes more of a problem with increased number of reads. A solution to this is **digital normalization**, which sets a hard threshold for highly expressed transcripts at a specified cut-off, e.g. 5x coverage. Trinity has an in-silico normalization procedure that is built-in to the program and can be specified with the ```---normalize_reads``` flag so you don't need to do this yourself. Just be sure you know what it is doing. You can read more about it [here](https://github.com/trinityrnaseq/trinityrnaseq/wiki/Trinity%20Insilico%20Normalization).

```
#!/bin/bash
#PBS -N trinity
#PBS -q batch
#PBS -l nodes=1:ppn=12:HIGHMEM
#PBS -l walltime=480:00:00
#PBS -l mem=100gb

cd $PBS_O_WORKDIR
module load trinity/2.0.6-UGA      
time Trinity --seqType fq --max_memory 100G --CPU 12 --normalize_reads --output Trinity --single Ctrl.fq,Heat.fq 1>trinity.out 2>trinity.err 
```

To check on the status of your job, enter the command ```qstat```. If you see a "Q" for its status, that means the job is in the queue waiting to begin. If you see a "R", that means the job is currently running. If you see a "C", this means that the job is completed, but does not necessarily mean the job was completed correctly. 

Once the computing job is complete, there will be a bunch of Trinity outfiles and also two outfiles related to your job submission: ```trinity.out``` and ```trinity.err```. The .out will show you anything that Trinity would have printed out onto the terminal for you to see. The .err file will show any warnings or errors associated with the computing job. If the job was not completed correctly or was completed instantly, it is likely that this file will help you troubleshoot. 

Your completed assembly will be the file ```Trinity.fasta```. Take a look at the file by typing ```head Trinity.fasta``` or ```less Trinity.fasta``` to ensure that it is indeed a FASTA file. 

Congratulations! You've just generated an assembly!

<br>

* Q6. What does it mean to set the CPU at 12 for the computing job? 
* Q7. What is a Trinity 'gene'? What is a Trinity 'isoform'?
* Q8. If you ran the Trinity assembly over again using all the same files and parameters, would the assembly be different? Why?

<br>


## Assembly Assessment

During lecture I introduced three different strategies to assess the quality of a transcriptome assembly. 

Strategy 1 | Strategy 2 | Strategy 3
--- | --- | ---
Alignment | TransRate | Orthology

### Strategy 1 - Alignment

There are two alignment methods we will use to assess the assembly: read mapping rate and alignment to reference genome.

#### Alignment to Reference Genome

```
#!/bin/bash
#PBS -N blat
#PBS -q batch
#PBS -l nodes=1:ppn=1:HIGHMEM
#PBS -l walltime=48:00:00
#PBS -l mem=80gb

cd $PBS_O_WORKDIR
time /usr/local/apps/blat/latest/bin/blat genome.fa Trinity.fasta -t=dna -q=rna -fine Trinity.blat
```


#### Read Mapping Rate

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

Bioinformatics is constantly improving with new advances in software in technology. A recent new software called TransRate was introduced as a software to assess the quality of a transcriptome assembly. You can read the published paper on it [here](http://genome.cshlp.org/content/early/2016/06/01/gr.196469.115). Otherwise, instructions on how to use and interpret it can be found on its [website](http://hibberdlab.com/transrate/index.html). 

TransRate is also on Sapelo, so you can find instructions on the GACRC wiki [here](https://wiki.gacrc.uga.edu/wiki/Transrate-Sapelo). The default shell script to run TransRate can be found below, but please see the TransRate manual to customize your results and explore what it can do/tell you. 

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

### Strategy 3 - Orthology

[BUCSO](http://busco.ezlab.org/) is a software to assess the completeness of a assembly or gene set based on evolutionarily-informed expectations of gene content from near-universal single-copy orthologs from OrthoDB. In short terms, you will be aligning your assembly to a highly conserved set of proteins among plants. 

```
#!/bin/bash
#PBS -N busco
#PBS -q batch
#PBS -l walltime=128:00:00
#PBS -l nodes=1:ppn=8:jlmnode
#PBS -l mem=10gb

cd $PBS_O_WORKDIR
module load miniconda2/4.0.5
export PATH="/usr/local/apps/augustus/latest/bin:$PATH"
export PATH="/usr/local/apps/augustus/latest/scripts:$PATH"
export AUGUSTUS_CONFIG_PATH="/usr/local/apps/augustus/latest/config/"

source activate busco
time python /usr/local/apps/miniconda2/4.0.5/envs/busco/bin/BUSCO_v1.2.py -in Trinity.fasta --mode trans -l embryophyta_odb9 -o Trinity.BUSCO.out -c 8
source deactivate
```

<br>


