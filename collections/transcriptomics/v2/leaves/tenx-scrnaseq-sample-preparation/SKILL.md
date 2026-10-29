---
name: tenx-scrnaseq-sample-preparation
description: Use when starting from raw 10x scRNAseq data from an infected sample
  and you need cellranger-produced host-aligned outputs plus the unaligned read pool
  ready for pathogen alignment and UMI quantification (e.g., HIV, SARS-CoV-2 samples).
when_to_use_negative:
- Input is already a cellranger output folder with unaligned reads extracted — proceed
  directly to pathogen alignment (scPathoQuant) rather than re-running sample preparation
- Data is not 10x single cell scRNAseq output (e.g., bulk RNAseq or plate-based scRNAseq)
  — cellranger count and 10x matrix assumptions do not hold
- The goal is quantifying host transcript expression only, with no pathogen/genome-of-interest
  component — standard cellranger processing suffices without the unaligned-read extraction
  step
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: cellranger
  role: 10x pipeline run first to align reads to the host genome of interest and generate
    the output folder (features.tsv.gz, matrix.mtx.gz) that downstream pathogen quantification
    consumes
  repo: https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/using/tutorial_ct
- name: scPathoQuant
  role: Consumes the prepared cellranger output (-10x) and pathogen genome folder
    (-p2genome), extracts unaligned reads and maps them to the pathogen genome
  repo: https://github.com/galelab/scPathoQuant
- name: docker
  role: Alternative containerized way to build and run the prepared scpathoquant image
    on the prepared sample
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:b1aedc892e0070440c617cfbc40348730c4905e49c333ec9a6955684d4926c5d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - cellranger
  - docker
  - scPathoQuant
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tenx-scrnaseq-sample-preparation

## Summary

Prepares 10x Genomics scRNAseq sample outputs so pathogen-derived reads can be aligned and quantified: run cellranger count against the host genome, retain the unaligned reads (e.g., from possorted_genome_bam.bam), and stage the cellranger output folder plus a pathogen reference genome folder (FASTA and optional GTF) for tools like scPathoQuant.

## When to use

Use when starting from raw 10x scRNAseq data from an infected sample and you need cellranger-produced host-aligned outputs plus the unaligned read pool ready for pathogen alignment and UMI quantification (e.g., HIV, SARS-CoV-2 samples).

## When NOT to use

- Input is already a cellranger output folder with unaligned reads extracted — proceed directly to pathogen alignment (scPathoQuant) rather than re-running sample preparation
- Data is not 10x single cell scRNAseq output (e.g., bulk RNAseq or plate-based scRNAseq) — cellranger count and 10x matrix assumptions do not hold
- The goal is quantifying host transcript expression only, with no pathogen/genome-of-interest component — standard cellranger processing suffices without the unaligned-read extraction step

## Inputs

- Raw 10x scRNAseq sequencing reads (e.g., SRA downloads from Bradley et al. HIV donor samples or Ravindra et al. SARS-CoV-2 airway epithelium samples)
- Human reference genome (hg38), optionally with the pathogen genome integrated (e.g., hg38 + HIV genome)
- Pathogen reference genome folder: pathogen FASTA plus optional pathogen GTF (e.g., SARS-CoV-2 genome and gtf from NCBI GenBank assembly GCF_009858895.2)
- 10x cellranger count output folder containing possorted_genome_bam.bam, filtered_feature_bc_matrix and raw_feature_bc_matrix (features.tsv.gz, matrix.mtx.gz)

## Outputs

- cellranger count output folder per sample (10x format: features.tsv.gz, matrix.mtx.gz) usable as scPathoQuant's -10x input
- Extracted unaligned-read files in the _tmp/ directory (unmmaped.bam, unmmaped.sam, barcodes_umi_read_table.csv, unmapped.fq.gz)
- Prepared pathogen reference genome folder (FASTA, optionally GTF) for the -p2genome argument

## How to apply

First run 10x cellranger count to align reads to the host genome of interest, which generates features.tsv.gz and matrix.mtx.gz in the filtered_feature_bc_matrix and raw_feature_bc_matrix folders; a parallel option aligns to a host genome with the pathogen genome integrated (e.g., hg38 with HIV) depending on the analysis. Extract the unaligned reads from possorted_genome_bam.bam into _tmp/ files (unmmaped.bam, unmmaped.sam) so barcodes and UMIs can be pulled for unmapped reads and written to fastq for pathogen mapping. Stage a pathogen reference folder containing at most two files: the pathogen reference genome FASTA (header kept simple, e.g. > HIV_virus, since the header is used to count total pathogen reads) and an optional pathogen GTF for gene-level counts (e.g., SARS-CoV-2 from NCBI GenBank assembly GCF_009858895.2). This prepared sample is then passed to scPathoQuant via -10x (cellranger output path) and -p2genome (pathogen genome folder), with -p specifying processors and bbmap as the default aligner.

## Related tools

- **cellranger** (10x pipeline run first to align reads to the host genome of interest and generate the output folder (features.tsv.gz, matrix.mtx.gz) that downstream pathogen quantification consumes) — https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/using/tutorial_ct
- **scPathoQuant** (Consumes the prepared cellranger output (-10x) and pathogen genome folder (-p2genome), extracts unaligned reads and maps them to the pathogen genome) — https://github.com/galelab/scPathoQuant
- **docker** (Alternative containerized way to build and run the prepared scpathoquant image on the prepared sample)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/pathogen/fastafilefolder
```

## Evaluation signals

- cellranger count completed and the output folder contains features.tsv.gz and matrix.mtx.gz in both filtered_feature_bc_matrix and raw_feature_bc_matrix folders
- _tmp/ directory contains unmmaped.bam, unmmaped.sam, barcodes_umi_read_table.csv, and unmapped.fq.gz, confirming unaligned reads were successfully pulled from possorted_genome_bam.bam
- Pathogen reference folder contains at most two files — a FASTA (with a simplified header such as > HIV_virus) and an optional GTF — as required by -p2genome
- The prepared sample passes end-to-end through a benchmarking script (e.g., CellRangerwithSPQBench.sh from the testSets repo) producing pathogen counts integrated into the 10x matrices

## Limitations

- The pathogen FASTA header is used to quantify total pathogen reads, so a complicated header name should be simplified (e.g., > HIV_virus) or counts will reflect the complicated name
- Temporary files in _tmp/ can be large; use --tmp_removal if space is an issue
- The pathogen GTF is optional — without it, gene-level outputs like pathogen_genes_al_sort_counts.sam are not produced
- Test data ordering matters: codes in the testSets repository are run in numbered order (e.g., 1.DownloadData.sh, then 2a/2b host alignment, then 3.scPathoQuant.sh)

## Evidence

- [readme] This software integrates pathogen UMI counts and pathogen UMI gene counts into 10x files generated by first running cellranger count which aligns reads to your host genome of interest: "This software integrates pathogen UMI counts and pathogen UMI gene counts into 10x files generated by first running [```cellranger count```]"
- [readme] scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files: "scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files."
- [methods] Methods parameter definition for the 10x input: the -10x argument must point at the cellranger count output folder.: "-10x = Path/to/10x/sample/ (this the path to the output folder generated by first running cellranger count )"
- [readme] Pathogen reference folder must contain at most the FASTA (header used for total-read quantification) and optional GTF.: "in this folder should be at most 2 files 1) the fasta file with the pathogen reference genome sequence and 2) pathogen gtf file (not mandatory"
- [readme] Test-set workflow runs cellranger against both hg38 with HIV integrated and hg38 only, in numbered order before scPathoQuant.: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract describes the overall operation of extracting host-unaligned sequences for pathogen mapping and quantification.: "scPathoQuant extracts sequences that were not aligned to the primary host genome"
