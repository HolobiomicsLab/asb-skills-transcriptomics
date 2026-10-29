---
name: adt-fastq-sample-multiplexing-setup
description: Use when ADT FASTQs from cell-hashing CITE-seq libraries must be organized
  per sample and converted into a cells-by-hashtags UMI count matrix before sample
  demultiplexing.
when_to_use_negative:
- The dataset is not cell-hashed (no hashtag oligonucleotide multiplexing) — this
  setup and demultiplexing step does not apply.
- A per-cell hashtag count matrix already exists and you only need downstream demultiplexing
  rather than recounting ADT FASTQs.
- You are only processing GEX FASTQs without ADT data — use the gene-expression module
  instead.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CITE-seq-Count
  role: Counts hashtag oligonucleotide-derived UMIs per cell barcode from ADT R1/R2
    read pairs against the hashtag reference
- name: gExcite
  role: Snakemake-embedded pipeline that expects the ADT FASTQ folder structure and
    aggregates the per-sample hashtag count matrices
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: snakedeploy
  role: Deploys the gExcite pipeline into the working directory before configuring
    the ADT hashing run
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:4baca1196292d1fe031ea9ed3f643d60a3c38e963de35cb68ada49ea212fec1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - CITE-seq-Count
  - gExcite
  - snakedeploy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# adt-fastq-sample-multiplexing-setup

## Summary

Prepares hashed ADT FASTQ inputs (I1/R1/R2) in the gExcite-expected folder and naming schema and runs CITE-seq-Count against the hashtag oligo reference to produce per-cell hashtag UMI count matrices, the entry point for hashing demultiplexing in CITE-seq ADT data.

## When to use

Use when ADT FASTQs from cell-hashing CITE-seq libraries must be organized per sample and converted into a cells-by-hashtags UMI count matrix before sample demultiplexing.

## When NOT to use

- The dataset is not cell-hashed (no hashtag oligonucleotide multiplexing) — this setup and demultiplexing step does not apply.
- A per-cell hashtag count matrix already exists and you only need downstream demultiplexing rather than recounting ADT FASTQs.
- You are only processing GEX FASTQs without ADT data — use the gene-expression module instead.

## Inputs

- ADT FASTQ files per sample (I1 index, R1 and R2 reads) as .fastq.gz following SAMPLENAME_S[Number]_L00[Lane Number]_[Read Type]_001.fastq.gz
- Hashtag oligonucleotide reference for CITE-seq-Count
- gExcite config.yaml with sample FASTQ paths relative to the working directory
- gExcite example test data (three hashed samples of human PBMC cells) for test runs

## Outputs

- Per-sample cells-by-hashtags UMI count matrices from CITE-seq-Count
- Aggregated hashtag_counts.csv (one row per cell barcode, one column per hashtag oligo) spanning all hashed samples, feeding gExcite's hashing deconvolution

## How to apply

Place each sample's ADT FASTQ files under input_fastqs_adt/SAMPLENAME/ using the schema SAMPLENAME_S[Number]_L00[Lane Number]_[Read Type]_001.fastq.gz (SAMPLENAME restricted to [a-zA-Z0-9_-]+); configure the pipeline config.yaml with FASTQ paths relative to the working directory (mandatory, not merely recommended). Deploy gExcite with snakedeploy inside a Mamba/Snakemake/Snakedeploy conda environment. For each hashed sample, run CITE-seq-Count on the ADT R1/R2 read pairs against the hashtag oligonucleotide reference so hashtag-derived UMIs are tallied per cell barcode. Collect the per-sample cells-by-hashtags matrices and aggregate them across samples into a single table (hashtag_counts.csv) used downstream by gExcite's hashing deconvolution. If you want to skip the resource-intensive cellranger/CITE-Seq steps, place dummy files in place of the FASTQs as described in the testdata README.

## Related tools

- **CITE-seq-Count** (Counts hashtag oligonucleotide-derived UMIs per cell barcode from ADT R1/R2 read pairs against the hashtag reference)
- **gExcite** (Snakemake-embedded pipeline that expects the ADT FASTQ folder structure and aggregates the per-sample hashtag count matrices) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **snakedeploy** (Deploys the gExcite pipeline into the working directory before configuring the ADT hashing run) — https://github.com/ETH-NEXUS/gExcite_pipeline

## Examples

```
snakedeploy deploy-workflow https://github.com/ETH-NEXUS/gExcite_pipeline --tag main .
```

## Evaluation signals

- hashtag_counts.csv exists and contains one row per cell barcode and one column per hashtag oligo
- hashtag_counts.csv covers cell barcodes from all three hashed PBMC samples
- All entries in the hashtag count table are non-negative integer UMI counts
- Every sample's ADT FASTQ set (I1/R1/R2) is present under input_fastqs_adt/SAMPLENAME/ and matches the SAMPLENAME_S[Number]_L00[Lane Number]_[Read Type]_001.fastq.gz naming schema (SAMPLENAME restricted to [a-zA-Z0-9_-]+)
- config.yaml FASTQ paths resolve correctly relative to the working directory

## Limitations

- SAMPLENAME can only contain characters [a-zA-Z0-9_-]+ — a dot in the sample name will stop cellranger-type tools immediately.
- Starting the pipeline after the resource-intensive cellranger count and CITE-Seq steps requires dummy files in place of the FASTQ files.
- Specifying input paths relative to the working directory in config.yaml is mandatory, not merely recommended; the workflow is tested on Linux only (MacOS/other Unix unsupported).

## Evidence

- [readme] The pipeline expects the FASTQ files per sample to be in the following folder structure, adhering to the naming schema:: "The pipeline expects the FASTQ files per sample to be in the following folder structure, adhering to the naming schema:"
- [methods] provides both, gene expression and CITE-seq analysis, as well as hashing deconvolution: "provides both, gene expression and CITE-seq analysis, as well as hashing deconvolution"
- [readme] example data for a test run ... with three hashed samples of human PBMC cells, so that hashing deconvolution, GEX analysis and ADT analysis can be performed: "example data for a test run ... with three hashed samples of human PBMC cells, so that hashing deconvolution, GEX analysis and ADT analysis can be performed"
- [readme] Starting processing after the resource-intensive cellranger count and CITE-Seq steps requires the presence of dummy files in place of the FASTQ files: "Starting processing after the resource-intensive cellranger count and CITE-Seq steps requires the presence of dummy files in place of the FASTQ files"
- [readme] SAMPLENAME can only contain the following characters [a-zA-Z0-9_-]+: [evidence span withheld — anchor status: external_doc]
