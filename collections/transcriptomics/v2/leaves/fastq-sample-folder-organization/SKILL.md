---
name: fastq-sample-folder-organization
description: Use when preparing raw 10x-style GEX and ADT (CITE-Seq) FASTQs for the
  gExcite or scAmpi Snakemake pipelines, which require one sub-directory per sample
  and SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz naming before cellranger
  count.
when_to_use_negative:
- The data are already processed past FASTQ stage (e.g. Cell Ranger count matrices
  or feature-barcode outputs) — the folder schema only applies to raw sequencing reads
- The raw data are un-demultiplexed (e.g. raw BCL or combined NovaSeq runs) that first
  require the provided index-hopping removal scripts rather than direct FASTQ placement
- Running outside Linux — the gExcite workflow is only tested on Linux and other Unix
  systems including MacOS are not supported
tools:
- name: Cell Ranger (cellranger)
  role: Consumer of the organized folder structure; expects one sub-directory per
    sample and the SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz naming for
    cellranger count
- name: gExcite
  role: Defines the required input_fastqs_gex / input_fastqs_adt folder schema and
    FASTQ naming convention before running the Snakemake workflow
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: scAmpi
  role: Included module whose cellranger rules consume the same per-sample FASTQ structure;
    documents the SAMPLENAME character restrictions and sample_map requirement
  repo: https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:1f121b6015710a5fd31feaa9ceb53f4e2224529454079590eb9c1ae18cb4dd06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - Cell Ranger (cellranger)
  - Mamba
  - gExcite
  - scAmpi
  - snakedeploy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# fastq-sample-folder-organization

## Summary

Organizes raw single-cell FASTQ files into the exact per-sample folder structure and naming schema that the gExcite pipeline and its Cell Ranger/scAmpi module expect, so that cellranger count, GEX, CITE-Seq/ADT, and hashing deconvolution steps can run without input errors.

## When to use

Use when preparing raw 10x-style GEX and ADT (CITE-Seq) FASTQs for the gExcite or scAmpi Snakemake pipelines, which require one sub-directory per sample and SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz naming before cellranger count.

## When NOT to use

- The data are already processed past FASTQ stage (e.g. Cell Ranger count matrices or feature-barcode outputs) — the folder schema only applies to raw sequencing reads
- The raw data are un-demultiplexed (e.g. raw BCL or combined NovaSeq runs) that first require the provided index-hopping removal scripts rather than direct FASTQ placement
- Running outside Linux — the gExcite workflow is only tested on Linux and other Unix systems including MacOS are not supported

## Inputs

- Raw gzipped single-cell FASTQ files (I1/I2/R1/R2 reads) per sample for gene expression (GEX)
- Raw gzipped single-cell FASTQ files (I1/I2/R1/R2 reads) per sample for ADT / CITE-Seq
- Optional dummy FASTQ files in place of real FASTQs for a quick test run that skips cellranger count
- Tab-delimited sample_map file listing each sample (with header 'sample') when using the scAmpi module

## Outputs

- Directory trees input_fastqs_gex/SAMPLENAME/ and input_fastqs_adt/SAMPLENAME/ with schema-compliant per-sample FASTQ files
- Path configuration in pipeline config.yaml pointing to the organized FASTQ folders relative to the working directory

## How to apply

Arrange the per-sample gzipped FASTQ files (read types I1, I2, R1, R2) into two parallel trees, input_fastqs_gex/SAMPLENAME/ and input_fastqs_adt/SAMPLENAME/, so each sample has its own sub-directory as Cell Ranger requires. Name every file SAMPLENAME_S[Number]_L00[LaneNumber]_[ReadType]_001.fastq.gz and restrict SAMPLENAME to the characters [a-zA-Z0-9_-]+, since sample names containing e.g. a dot make cellranger stop. Specify the input paths relative to the working directory — this is mandatory, not merely recommended, for gExcite. If a quick test run starting after the resource-intensive cellranger count and CITE-Seq steps is intended, place dummy FASTQ files at the paths where the real FASTQs would sit. For combined GEX and ADT NovaSeq sequencing data, first run the provided index-hopping cleanup scripts on the data before organizing and launching the pipeline.

## Related tools

- **Cell Ranger (cellranger)** (Consumer of the organized folder structure; expects one sub-directory per sample and the SAMPLENAME_S[Number]_L00[Lane]_[ReadType]_001.fastq.gz naming for cellranger count)
- **gExcite** (Defines the required input_fastqs_gex / input_fastqs_adt folder schema and FASTQ naming convention before running the Snakemake workflow) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **scAmpi** (Included module whose cellranger rules consume the same per-sample FASTQ structure; documents the SAMPLENAME character restrictions and sample_map requirement) — https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA

## Examples

```
mkdir -p input_fastqs_gex/SAMPLENAME input_fastqs_adt/SAMPLENAME
```

## Evaluation signals

- Every FASTQ filename matches the pattern SAMPLENAME_S[Number]_L00[LaneNumber]_[ReadType]_001.fastq.gz with ReadType in {I1, I2, R1, R2}
- Each SAMPLENAME appears as its own sub-directory under both input_fastqs_gex and input_fastqs_adt, as Cell Ranger expects one sub-directory per sample
- SAMPLENAME uses only characters [a-zA-Z0-9_-]+ (no dots), otherwise cellranger stops immediately
- FASTQ input paths in config.yaml are specified relative to the working directory (mandatory for gExcite)
- For a quick test run, dummy FASTQ files are present at the documented paths so Snakemake rules downstream of cellranger count and CITE-Seq can execute

## Limitations

- SAMPLENAME can only contain [a-zA-Z0-9_-]+; a dot or other character causes cellranger to stop right away
- Specifying FASTQ inputs relative to the working directory is mandatory for gExcite, not merely recommended
- Combined GEX and ADT NovaSeq sequencing data require optional preprocessing (index hopping removal scripts) before the run
- Quick test runs require dummy FASTQ files in place of the real FASTQs to start after cellranger count and CITE-Seq steps

## Evidence

- [readme] README prepare-FASTQ folder schema: "The pipeline expects the FASTQ files per sample to be in the following folder structure, adhering to the naming schema"
- [readme] SAMPLENAME character restriction: "NOTE: SAMPLENAME can only contain the following characters [a-zA-Z0-9_-]+  
  E.g., if the SAMPLENAME contains a dot, cellranger will stop right away."
- [methods] Index hopping removal prerequisite: "In case of combined GEX and ADT NovaSeq sequencing data, scripts are provided to clean up the data before a run"
- [discussion] Relative paths mandatory: "specifying the input relative to the working directory is now mandatory, not merely recommended"
