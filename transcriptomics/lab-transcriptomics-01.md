---
title: Assembly and Assessment Lab
categories:
  - lab
---

Today we will be assembling a transcriptome for Maize (*Zea mays*) and assessing its assembly quality via several different strategies. 

## Data Acquisition

We will be looking at Arabidopsis thaliana data because it has a smaller genome and will take less time in computational processes. The data set has two conditions: control and heat stress. You can copy the files from 

```cp  ```

Unzip the package containing the data.

```gunzip NAME.fasta.gz```

## Transcriptome Assembly

We will be using [Trinity](https://github.com/trinityrnaseq/trinityrnaseq/wiki) to generate a transcriptome assembly. 

## Assembly Assessment

I introduced three different strategies to assess the quality of a transcriptome assembly. 

Strategy 1 | Strategy 2 | Strategy 3
--- | --- | ---
Alignment | TransRate | FASTQC

