---
name: hashing-deconvolution-count-generation
description: Use when hashed CITE-seq ADT FASTQ files (I1/R1/R2 per sample) need to
  be converted into per-cell hashtag UMI count matrices before hashtag deconvolution,
  e.g. for multiplexed PBMC samples in gExcite.
when_to_use_negative:
- Input is gene expression (GEX) FASTQ data — that path is handled by cellranger/scAmpi
  in gExcite, not by hashtag counting
- A cells-by-hashtags count matrix already exists and the goal is sample demultiplexing
  itself rather than count generation
- ADT reads are not hashtag-derived (e.g. antibody-derived tags without hashing) —
  CITE-seq-Count should then be run against an ADT antibody reference, not a hashtag
  oligo reference
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CITE-seq-Count
  role: Counts hashtag oligonucleotide UMIs per cell barcode from ADT R1/R2 read pairs
    against the hashtag reference, producing cells-by-hashtags matrices
- name: gExcite
  role: Snakemake-embedded pipeline whose hashing stage drives the per-sample CITE-seq-Count
    runs and aggregation into hashtag_counts.csv
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Snakemake
  role: Workflow manager that schedules and executes the per-sample hashtag counting
    rules reproducibly
  repo: https://snakemake.readthedocs.io/en/stable/
- name: snakedeploy
  role: Deploys the gExcite pipeline repository into the working directory before
    configuring the ADT hashing run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:2f869aaae7d4364de749ab638d54c859f6034f7a326fc4de0cb4df1761babeaf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - CITE-seq-Count
  - Snakemake
  - gExcite
  - snakedeploy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# hashing-deconvolution-count-generation

## Summary

Generates per-cell hashtag oligo UMI count matrices from hashed CITE-seq ADT FASTQs by running CITE-seq-Count per sample, as the first stage of gExcite's hashing deconvolution. These cells-by-hashtags matrices are the prerequisite input for subsequent sample demultiplexing.

## When to use

Use when hashed CITE-seq ADT FASTQ files (I1/R1/R2 per sample) need to be converted into per-cell hashtag UMI count matrices before hashtag deconvolution, e.g. for multiplexed PBMC samples in gExcite.

## When NOT to use

- Input is gene expression (GEX) FASTQ data — that path is handled by cellranger/scAmpi in gExcite, not by hashtag counting
- A cells-by-hashtags count matrix already exists and the goal is sample demultiplexing itself rather than count generation
- ADT reads are not hashtag-derived (e.g. antibody-derived tags without hashing) — CITE-seq-Count should then be run against an ADT antibody reference, not a hashtag oligo reference

## Inputs

- ADT FASTQ files per sample (I1, R1, R2 .fastq.gz) in input_fastqs_adt/SAMPLENAME/ with gExcite's naming schema
- Hashtag oligonucleotide reference sequences

## Outputs

- Per-sample cells-by-hashtags UMI count matrices from CITE-seq-Count
- Aggregated hashtag_counts.csv (one row per cell barcode, one column per hashtag oligo, non-negative integer UMI counts)

## How to apply

Place each sample's ADT FASTQ files under input_fastqs_adt/SAMPLENAME/ following the SAMPLENAME_S[Number]_L00[Lane Number]_[Read Type]_001.fastq.gz schema, with paths specified relative to the working directory (mandatory per gExcite). For each sample, run CITE-seq-Count on the ADT R1/R2 read pairs against the hashtag oligonucleotide reference so hashtag-derived UMIs are tallied per cell barcode; gExcite orchestrates this via Snakemake with parameters set in config.yaml. Collect the per-sample cells-by-hashtags count matrices and aggregate them into a single table spanning all samples (e.g. the three hashed human PBMC example samples). The result feeds directly into the downstream hashtag demultiplexing stage of gExcite.

## Related tools

- **CITE-seq-Count** (Counts hashtag oligonucleotide UMIs per cell barcode from ADT R1/R2 read pairs against the hashtag reference, producing cells-by-hashtags matrices)
- **gExcite** (Snakemake-embedded pipeline whose hashing stage drives the per-sample CITE-seq-Count runs and aggregation into hashtag_counts.csv) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Snakemake** (Workflow manager that schedules and executes the per-sample hashtag counting rules reproducibly) — https://snakemake.readthedocs.io/en/stable/
- **snakedeploy** (Deploys the gExcite pipeline repository into the working directory before configuring the ADT hashing run)

## Examples

```
snakedeploy deploy-workflow https://github.com/ETH-NEXUS/gExcite_pipeline --tag main .
```

## Evaluation signals

- hashtag_counts.csv exists in the results directory
- hashtag_counts.csv contains one row per cell barcode and one column per hashtag oligo for all three hashed samples
- all entries in the count table are non-negative integer UMI counts
- input FASTQs conform to the SAMPLENAME_S[Number]_L00[Lane]_[Read Type]_001.fastq.gz naming schema in input_fastqs_adt/SAMPLENAME/ folders
- the aggregated table spans all samples listed in config.yaml, not just a single sample

## Limitations

- FASTQ inputs must be specified relative to the working directory (mandatory, not merely recommended), otherwise the pipeline cannot resolve the files
- For a quick test run starting after cellranger count and CITE-Seq steps, dummy FASTQ files must be placed where the real FASTQs would be
- The gExcite workflow is compatible with and tested on Linux only; other Unix systems including MacOS are not supported
- For combined GEX and ADT NovaSeq sequencing data, index-hopping cleanup scripts must be run before the pipeline

## Evidence

- [other] gExcite first-stage hashtag counting: [evidence span withheld — anchor status: not_found]
- [readme] gExcite workflow scope: "gExcite is a start-to-end workflow embedded in Snakemake that provides both, gene expression and CITE-seq analysis, as well as hashing deconvolution."
- [readme] Example hashed PBMC test data: "with three hashed samples of human PBMC cells, so that hashing deconvolution, GEX analysis and ADT analysis can be performed."
- [readme] FASTQ folder structure requirement: "The pipeline expects the FASTQ files per sample to be in the following folder structure, adhering to the naming schema"
- [methods] config.yaml configuration step: "the pipeline config.yaml is configured to your data"
- [discussion] Relative input paths mandatory: "specifying the input relative to the working directory is now mandatory, not merely recommended"
