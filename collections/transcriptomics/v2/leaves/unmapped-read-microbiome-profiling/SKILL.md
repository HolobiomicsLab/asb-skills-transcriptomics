---
name: unmapped-read-microbiome-profiling
description: 'Use when you have small RNA-seq FASTQ data processed through QC and
  human-genome alignment (e.g., STAR) and need to quantify microbial RNA species from
  the unmapped reads in the four supported taxons: archaea, bacteria, fungi and viruses.'
when_to_use_negative:
- Your reads have not first been filtered and mapped to the human genome — the Microbe
  module specifically consumes qualified reads that were NOT mapped to hg19/hg38
- You need microbial profiling beyond the four supported taxons (archaea, bacteria,
  fungi, viruses); COMPSRA's Microbe module only supports these four
- You lack disk/memory resources for the BLAST nt database and prebuilt microbial
  databases, or cannot afford the ~3.5 hours per sample runtime reported for full
  four-taxon profiling
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3168
tools:
- name: COMPSRA
  role: Hosts the Microbe module (-mic) that runs BLAST on unmapped reads, parses
    hits, and aggregates taxon-level read counts
  repo: https://github.com/cougarlj/COMPSRA
- name: BLAST
  role: Aligns qualified unmapped reads to the UCSC nt database for microbial sequence
    comparison
- name: STAR
  role: Default aligner in the Alignment module; produces the unmapped BAM/FASTA output
    that feeds the Microbe module
- name: UCSC nt database
  role: Nucleotide reference database against which unmapped reads are BLAST-aligned
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:3f675b1cd5c151155733f71961ab65d96e35e8ec01d1faf2f889a30f082dfe47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - BLAST
  - COMPSRA
  - STAR
  - UCSC nt database
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# unmapped-read-microbiome-profiling

## Summary

Profiles fragmental microbial RNA (archaea, bacteria, fungi, viruses) in small RNA-seq data by BLAST-aligning reads that failed to map to the human genome against the UCSC nt database, as implemented in COMPSRA's Microbe module.

## When to use

Use when you have small RNA-seq FASTQ data processed through QC and human-genome alignment (e.g., STAR) and need to quantify microbial RNA species from the unmapped reads in the four supported taxons: archaea, bacteria, fungi and viruses.

## When NOT to use

- Your reads have not first been filtered and mapped to the human genome — the Microbe module specifically consumes qualified reads that were NOT mapped to hg19/hg38
- You need microbial profiling beyond the four supported taxons (archaea, bacteria, fungi, viruses); COMPSRA's Microbe module only supports these four
- You lack disk/memory resources for the BLAST nt database and prebuilt microbial databases, or cannot afford the ~3.5 hours per sample runtime reported for full four-taxon profiling

## Inputs

- Unmapped reads from the Alignment module (e.g. sample01_17to50_FitRead_STAR_Aligned_UnMapped.bam), FASTQ or SAM format accepted by -in
- A sample list file (-inf file.list) for multiple samples, one file per line
- Prebuilt BLAST microbial databases (blast_archaea, blast_bacteria, blast_fungi, blast_viruses) plus BLAST taxonomy information from UCSC nt

## Outputs

- Per-sample microbial read counts per taxon (archaea, bacteria, fungi, viruses) tabulated from BLAST hits via COMPSRA's fast text parsing
- Taxon-level counts aggregated across samples using the phylogenetic tree to account for interspecies homology

## How to apply

First run the COMPSRA QC module (adapter trimming with e.g. -ra TGGAATTCTCGGGTGCCAAGG, -rb 4, quality thresholds -rh 20 -rt 20 -rr 20) and the Alignment module with STAR against hg19/hg38; the reads not mapped to the human genome are saved (e.g. *_Aligned_UnMapped.bam / FASTA). Install BLAST, taxonomy information, and the prebuilt microbial databases via the COMPSRA ToolKit (java -jar COMPSRA.jar -tk -dr -ck blast, blast_taxonomy, blast_archaea, blast_bacteria, blast_fungi, blast_viruses). Then run the Microbe module with -mic -mtool Blast -mdb archaea,bacteria,fungi,viruses so unmapped reads are aligned to the UCSC nucleotide (nt) database using BLAST. Because of interspecies homology, aggregate BLAST hits by listing all potential taxa with their read counts according to the phylogenetic tree, restricted to the four supported taxons. Validation: on the article's 12 healthy human serum samples the module reproduced totals of 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses, with an estimated runtime of about 3.5 hours per sample on average.

## Related tools

- **COMPSRA** (Hosts the Microbe module (-mic) that runs BLAST on unmapped reads, parses hits, and aggregates taxon-level read counts) — https://github.com/cougarlj/COMPSRA
- **BLAST** (Aligns qualified unmapped reads to the UCSC nt database for microbial sequence comparison)
- **STAR** (Default aligner in the Alignment module; produces the unmapped BAM/FASTA output that feeds the Microbe module)
- **UCSC nt database** (Nucleotide reference database against which unmapped reads are BLAST-aligned)

## Examples

```
java -jar COMPSRA.jar -mic -mtool Blast -mdb archaea,bacteria,fungi,viruses -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned_UnMapped.bam -out ./example_out/
```

## Evaluation signals

- The Microbe module command completes with -mtool Blast and -mdb covering archaea,bacteria,fungi,viruses and produces taxon-level read-count output for each sample
- On the reference benchmark (12 healthy human serum samples), totals reproduce 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses
- Output counts are aggregated across all potential taxa per the phylogenetic tree rather than assigning each read to a single best hit, reflecting the homology-aware counting rule
- Per-sample average runtime across the four taxons is on the order of ~3.5 hours, consistent with the reported estimate
- Input file is the STAR *_Aligned_UnMapped output of qualified (QC-passed) reads, not raw FASTQ

## Limitations

- Only four microbial taxons are supported: archaea, bacteria, fungi and viruses
- Interspecies homology means reads can map to multiple taxa, so counts are reported per potential taxon rather than as definitive species assignments
- Full four-taxon profiling is slow — an estimated ~3.5 hours per sample on average in the article's evaluation
- Microbe module requires extra resources downloaded via the ToolKit (BLAST, blast_taxonomy, and each blast_<taxon> database) and is optional rather than part of the default three-module pipeline

## Evidence

- [methods] Microbe module: BLAST alignment of unmapped reads to UCSC nt database for microbial profiling: "The qualified reads that were not mapped to the human genome in the Alignment Module are aligned to the nucleotide (nt) database27 from UCSC using BLAST"
- [results] Detected taxon counts in 12 serum samples: "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs, as well as 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses"
- [results] Microbe module runtime on the 12 serum samples: "For the 12 test samples, the estimated time on average is 3.5 hours, including viruses, archaea, bacteria and fungi (see Supplementary Table S1)"
- [readme] Four supported taxons: "The four major microbial taxons archaea, bacteria, fungi and viruses are supported."
- [readme] Microbe module invocation: "java -jar COMPSRA.jar -mic -mtool Blast -mdb archaea -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned_UnMapped.bam -out ./example_out/"
