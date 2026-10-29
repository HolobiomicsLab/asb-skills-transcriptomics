---
name: cite-seq-hashtag-umi-counting
description: Use when hashed CITE-seq ADT FASTQs (I1/R1/R2 per sample) must be converted
  into per-cell hashtag UMI count matrices prior to demultiplexing sample identities
  in a gExcite/Snakemake workflow.
when_to_use_negative:
- Input is already a cells-by-hashtag count matrix or an aggregated hashtag_counts.csv
  — skip straight to demultiplexing
- The goal is gene expression processing (GEX FASTQs), which is handled by cellranger
  count via the scAmpi module, not ADT hashtag counting
- The data are unhashed CITE-seq ADT libraries without hashtag oligos — there is nothing
  to deconvolve or count per hashtag
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0121
tools:
- name: CITE-seq-Count
  role: Core tool that counts hashtag-derived UMIs per cell barcode from ADT R1/R2
    reads against the hashtag oligo reference
- name: gExcite
  role: Start-to-end Snakemake workflow whose hashing deconvolution branch invokes
    this counting step on ADT FASTQs
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Snakemake
  role: Workflow manager that orchestrates the CITE-seq hashtag counting rules reproducibly
    and scalably
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: conda
  role: Environment basis for installing Mamba/Snakemake/Snakedeploy used to deploy
    and run the pipeline with per-rule environments
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:048e7b10f2a5d0d55e22d2dfb2fa6650e0bddcb6d7b48203073c7313f489043a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - CITE-seq-Count
  - Snakemake
  - conda
  - gExcite
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cite-seq-hashtag-umi-counting

## Summary

First stage of gExcite's hashing demultiplexing: run CITE-seq-Count on hashed CITE-seq ADT FASTQs (I1/R1/R2) to tally hashtag oligonucleotide-derived UMIs per cell barcode, producing the per-sample cells-by-hashtags count matrices that feed downstream sample demultiplexing.

## When to use

Use when hashed CITE-seq ADT FASTQs (I1/R1/R2 per sample) must be converted into per-cell hashtag UMI count matrices prior to demultiplexing sample identities in a gExcite/Snakemake workflow.

## When NOT to use

- Input is already a cells-by-hashtag count matrix or an aggregated hashtag_counts.csv — skip straight to demultiplexing
- The goal is gene expression processing (GEX FASTQs), which is handled by cellranger count via the scAmpi module, not ADT hashtag counting
- The data are unhashed CITE-seq ADT libraries without hashtag oligos — there is nothing to deconvolve or count per hashtag

## Inputs

- Hashed-sample ADT FASTQ files (I1, R1, R2), gzipped, per sample, named SAMPLENAME_S[Number]_L00[Lane Number]_[Read Type]_001.fastq.gz
- Hashtag oligonucleotide reference list (hashtag barcodes/tags for the antibody panel)

## Outputs

- Per-sample cells-by-hashtags count matrix of non-negative integer UMI counts (hashtag_counts.csv)
- Aggregated multi-sample hashtag count table spanning all hashed samples

## How to apply

Place each hashed sample's ADT FASTQs under input_fastqs_adt/SAMPLENAME/ following the SAMPLENAME_S[Number]_L00[Lane Number]_[Read Type]_001.fastq.gz schema, then run CITE-seq-Count on each sample's ADT R1/R2 read pairs against the hashtag oligonucleotide reference so that hashtag-derived UMIs are tallied per cell barcode. Collect the per-sample cells-by-hashtags count matrices CITE-seq-Count emits and aggregate them across all samples (e.g. the three hashed human PBMC example samples) into a single table used for the subsequent hashing deconvolution step. In gExcite this step is embedded in the Snakemake workflow and configured through the pipeline config.yaml, with FASTQ paths specified relative to the working directory (mandatory).

## Related tools

- **CITE-seq-Count** (Core tool that counts hashtag-derived UMIs per cell barcode from ADT R1/R2 reads against the hashtag oligo reference)
- **gExcite** (Start-to-end Snakemake workflow whose hashing deconvolution branch invokes this counting step on ADT FASTQs) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Snakemake** (Workflow manager that orchestrates the CITE-seq hashtag counting rules reproducibly and scalably) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **conda** (Environment basis for installing Mamba/Snakemake/Snakedeploy used to deploy and run the pipeline with per-rule environments)

## Examples

```
CITE-seq-Count -R1 SAMPLENAME_S4_L001_R1_001.fastq.gz -R2 SAMPLENAME_S4_L001_R2_001.fastq.gz -t hashtags.csv -o output/SAMPLENAME -U 10 -cells 5000
```

## Evaluation signals

- hashtag_counts.csv exists and contains one row per cell barcode and one column per hashtag oligo for every hashed sample
- All entries in the hashtag count matrices are non-negative integer UMI counts
- Per-sample matrices from CITE-seq-Count can be aggregated into a single table spanning all samples (e.g. the three example PBMC samples)
- Input ADT FASTQs follow the SAMPLENAME_S[Number]_L00[Lane Number]_[Read Type]_001.fastq.gz naming schema under input_fastqs_adt/SAMPLENAME/
- The demultiplexing stage downstream receives a populated cells-by-hashtags matrix rather than empty or malformed output

## Limitations

- FastQ paths must be specified relative to the working directory — this is mandatory, not merely recommended
- Starting processing before/during cellranger count and CITE-Seq steps requires manually installed software (e.g. cellranger binary on PATH); the quick test run instead starts after these resource-intensive steps
- The workflow is compatible with and tested on Linux only; other Unix systems (including MacOS) are currently not supported

## Evidence

- [abstract] start-to-end workflow with gene and antibody analysis plus hashing deconvolution: "we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution"
- [methods] CITE-Seq steps precede the quick test run start point: "starts after the resource-intensive cellranger count and CITE-Seq steps"
- [readme] gExcite test data are three hashed human PBMC samples for hashing deconvolution and ADT analysis: "with three hashed samples of human PBMC cells, so that hashing deconvolution, GEX analysis and ADT analysis can be performed"
- [readme] FASTQ input naming schema required per sample: "The pipeline expects the FASTQ files per sample to be in the following folder structure, adhering to the naming schema"
