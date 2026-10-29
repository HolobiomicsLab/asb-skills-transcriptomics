---
name: single-cell-barcode-oligo-assignment
description: Use when input is hashed single-cell CITE-seq ADT FASTQ data (I1/R1/R2)
  and per-cell hashtag oligo UMI counts are needed as input to hashing demultiplexing.
when_to_use_negative:
- The data are GEX-only (no hashing), where gExcite's gene-expression/ADT branch (cellranger
  count and CITE-Seq) is the relevant path instead
- Hashtag counts already exist (e.g. a ready-made hashtag_counts.csv) and the task
  is demultiplexing, not the initial counting stage
- Input is already processed ADT/CITE-seq output rather than raw ADT FASTQ read pairs
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CITE-seq-Count
  role: Counts hashtag oligonucleotide tags per individual cell from ADT R1/R2 read
    pairs, yielding per-sample cells-by-hashtags UMI matrices
- name: gExcite
  role: Snakemake-embedded start-to-end workflow whose hashing deconvolution branch
    invokes the hashtag-counting stage
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Snakemake
  role: Workflow manager that orchestrates the hashtag-counting rules within gExcite
    for reproducible, scalable runs
  repo: https://snakemake.readthedocs.io
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:f38567a4bf442a0bd9c0222c0924a04f7079d76ec21e37c0c79985d2a30910c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - CITE-seq-Count
  - Snakemake
  - gExcite
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-cell-barcode-oligo-assignment

## Summary

Performs the initial hashtag-counting stage of gExcite's hashing demultiplexing: CITE-seq-Count is run on hashed CITE-seq ADT R1/R2 read pairs against a hashtag oligonucleotide reference to tally hashtag-derived UMIs per cell barcode, producing cells-by-hashtags count matrices for subsequent demultiplexing.

## When to use

Use when input is hashed single-cell CITE-seq ADT FASTQ data (I1/R1/R2) and per-cell hashtag oligo UMI counts are needed as input to hashing demultiplexing.

## When NOT to use

- The data are GEX-only (no hashing), where gExcite's gene-expression/ADT branch (cellranger count and CITE-Seq) is the relevant path instead
- Hashtag counts already exist (e.g. a ready-made hashtag_counts.csv) and the task is demultiplexing, not the initial counting stage
- Input is already processed ADT/CITE-seq output rather than raw ADT FASTQ read pairs

## Inputs

- Per-sample ADT FASTQ files (I1/R1/R2 .fastq.gz) arranged as input_fastqs_adt/SAMPLENAME/SAMPLENAME_S[Number]_L00[Lane]_[Read Type]_001.fastq.gz
- Hashtag oligonucleotide reference (barcode sequences) for CITE-seq-Count
- gExcite config.yaml with ADT FASTQ paths relative to the working directory

## Outputs

- Per-sample cells-by-hashtags UMI count matrix emitted by CITE-seq-Count
- Aggregated hashtag_counts.csv with one row per cell barcode and one column per hashtag oligo spanning all hashed samples

## How to apply

Place each hashed sample's ADT FASTQ files under input_fastqs_adt/SAMPLENAME/ following the SAMPLENAME_S[Number]_L00[Lane Number]_[Read Type]_001.fastq.gz naming schema (e.g. SAMPLENAME_S4_L001_R2_001.fastq.gz). Set up the gExcite Snakemake environment with conda/Mamba plus Snakemake and snakedeploy, and fill in the pipeline config.yaml, specifying FASTQ paths relative to the working directory (mandatory, not merely recommended). For each sample, run CITE-seq-Count on the ADT R1/R2 read pairs against the hashtag oligonucleotide reference so that hashtag-derived UMIs are tallied per cell barcode. Collect the per-sample cells-by-hashtags count matrices and aggregate them across all three hashed samples into a single table used downstream for hashing deconvolution. gExcite provides example test data of three hashed human PBMC samples so the whole hashing, GEX and ADT flow can be validated end to end.

## Related tools

- **CITE-seq-Count** (Counts hashtag oligonucleotide tags per individual cell from ADT R1/R2 read pairs, yielding per-sample cells-by-hashtags UMI matrices)
- **gExcite** (Snakemake-embedded start-to-end workflow whose hashing deconvolution branch invokes the hashtag-counting stage) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Snakemake** (Workflow manager that orchestrates the hashtag-counting rules within gExcite for reproducible, scalable runs) — https://snakemake.readthedocs.io

## Examples

```
snakedeploy deploy-workflow https://github.com/ETH-NEXUS/gExcite_pipeline --tag main . ; snakemake --use-conda --configfile config/config.yaml
```

## Evaluation signals

- hashtag_counts.csv exists after the run
- hashtag_counts.csv contains one row per cell barcode and one column per hashtag oligo for all three hashed samples
- All entries in hashtag_counts.csv are non-negative integer UMI counts
- ADT FASTQs follow the SAMPLENAME_S[Number]_L00[Lane Number]_[Read Type]_001.fastq.gz naming schema in per-sample subdirectories
- A quick test run on the three hashed PBMC example samples reproduces the hashing deconvolution stage

## Limitations

- The gExcite workflow is compatible with and tested on Linux only; other Unix systems including MacOS are currently not supported
- Starting after the resource-intensive cellranger count and CITE-Seq steps requires placing dummy files in place of the FASTQ files
- Specifying input FASTQ paths relative to the working directory in config.yaml is mandatory, not merely recommended
- The example test data cover three hashed human PBMC samples; other tissues/hash panels may need additional configuration

## Evidence

- [other] The first stage of gExcite's hashing demultiplexing applies CITE-seq-Count to count hashtag oligonucleotide tags per individual cell: [evidence span withheld — anchor status: not_found]
- [abstract] start-to-end workflow for gene and antibody analysis with hashing deconvolution: "we designed gExcite, a start-to-end workflow that provides both gene and antibody expression analysis, as well as hashing deconvolution. Embedded in the Snakemake workflow manager, gExcite"
- [methods] gExcite provides CITE-seq analysis and hashing deconvolution: "provides both, gene expression and CITE-seq analysis, as well as hashing deconvolution"
- [readme] FASTQ folder structure and naming schema: "The pipeline expects the FASTQ files per sample to be in the following folder structure, adhering to the naming schema:"
- [readme] Example test data of three hashed PBMC samples: "with three hashed samples of human PBMC cells, so that hashing deconvolution, GEX analysis and ADT analysis can be performed."
