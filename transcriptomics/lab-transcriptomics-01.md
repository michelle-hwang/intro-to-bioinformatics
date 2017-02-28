---
title: Assembly and Assessment Lab
categories:
  - lab
---

Today we will be assembling a transcriptome for Maize (*Zea mays*) and assessing its assembly quality via several different strategies. 

## Data Acquisition

We will be looking at Arabidopsis thaliana data because it has a smaller genome and will take less time in computational processes. The [data](https://www.ncbi.nlm.nih.gov/Traces/study/?acc=SRP063471) is a single-end Illumina sequencing set with samples treated under salt stress, heat stress, and both. To simplify this, will only download the 3 control samples and the 3 heat stress samples. Each sample is about 3.5GB. You can download the SRA files from NCBI using sratoolkit:

```
module load sratoolkit

for f in SRR2302908 SRR2302909 SRR2302910 SRR2302914 SRR2302915 SRR2302916; do
	fastq-dump $f
done
```
This will take some time, since the files are large. They are currently in *.fastq* format. 


## Transcriptome Assembly

We will be using [Trinity](https://github.com/trinityrnaseq/trinityrnaseq/wiki) to generate a transcriptome assembly. 

## Assembly Assessment

I introduced three different strategies to assess the quality of a transcriptome assembly. 

Strategy 1 | Strategy 2 | Strategy 3
--- | --- | ---
Alignment | TransRate | FASTQC

