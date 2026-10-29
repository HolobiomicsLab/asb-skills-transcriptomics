---
name: single-cell-rnaseq-preprocessing
description: Use when raw 10x scRNA-seq FASTQ files (with optional CITE-seq ADT and
  hashing data) must be converted to a filtered, clustered, cell-type-annotated expression
  matrix via a reproducible end-to-end workflow.
when_to_use_negative:
- The goal is antibody/ADT (CITE-seq) or hashing deconvolution analysis only — use
  the dedicated gExcite ADT and hashing modules instead of the GEX preprocessing layer.
- Data is already a filtered, processed count matrix from cellranger/STARsolo with
  QC done — start the pipeline after the resource-intensive cellranger steps rather
  than re-running from raw reads.
- Analysis must run on macOS or other non-Linux Unix systems — gExcite is compatible
  with and tested on Linux only.
- STARsolo is required instead of cellranger — that route is a scAmpi snakefile variant
  (snakefile_basic_starsolo.smk), not the default gExcite workflow.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: gExcite
  role: Start-to-end Snakemake workflow that embeds scAmpi for GEX preprocessing alongside
    ADT and hashing analysis
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: scAmpi
  role: Embedded module performing QC/empty-droplet filtering, normalisation, clustering,
    cell type classification and DE analysis on the GEX matrix
  repo: https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
- name: cellranger
  role: Read mapping and UMI counting from raw FASTQs; must be installed manually
    and added to PATH
- name: snakedeploy
  role: Deploys the gExcite_pipeline workflow from GitHub into the working directory
  repo: https://github.com/ETH-NEXUS/gExcite_pipeline
- name: Snakemake
  role: Workflow manager (>=6.12.1) orchestrating rules, --use-conda environment provisioning,
    and module inclusion of scAmpi
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad329
    title: 'gExcite: a start-to-end framework for single-cell gene expression, hashing,
      and antibody analysis'
  source_content_hash: sha256:8f083d1b15dfc14db1ea3d8e5c20e41925033df370cd02d58ed03851ff45c602
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ETH-NEXUS/gExcite_pipeline
  tools:
  - Cell Ranger (cellranger)
  - Mamba
  - Snakemake
  - cellranger
  - gExcite
  - scAmpi
  - snakedeploy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-rnaseq-preprocessing

## Summary

Reproducible Snakemake-embedded preprocessing of single-cell gene expression data as implemented in gExcite/scAmpi: cellranger count from raw FASTQ, QC with empty-droplet and ambient-RNA filtering, normalisation, clustering and cell type classification.

## When to use

Use when raw 10x scRNA-seq FASTQ files (with optional CITE-seq ADT and hashing data) must be converted to a filtered, clustered, cell-type-annotated expression matrix via a reproducible end-to-end workflow.

## When NOT to use

- The goal is antibody/ADT (CITE-seq) or hashing deconvolution analysis only — use the dedicated gExcite ADT and hashing modules instead of the GEX preprocessing layer.
- Data is already a filtered, processed count matrix from cellranger/STARsolo with QC done — start the pipeline after the resource-intensive cellranger steps rather than re-running from raw reads.
- Analysis must run on macOS or other non-Linux Unix systems — gExcite is compatible with and tested on Linux only.
- STARsolo is required instead of cellranger — that route is a scAmpi snakefile variant (snakefile_basic_starsolo.smk), not the default gExcite workflow.

## Inputs

- Raw single-cell FASTQ files per sample (I1, R1, R2) in input_fastqs_gex/SAMPLENAME/ following SAMPLENAME_S[Number]_L00[LaneNumber]_[ReadType]_001.fastq.gz naming
- config.yaml configured with full input FASTQ path, resource/reference settings, and sample_map (tab-delimited file with 'sample' column)

## Outputs

- Cellranger count outputs (gene-cell count matrices) per sample
- scAmpi GEX processing outputs: QC-filtered, normalised matrices with clustering, cell type assignment and DE/pathway results in the results directory

## How to apply

Deploy the gExcite pipeline with snakedeploy from the ETH-NEXUS/gExcite_pipeline repository into a conda/Mamba environment with Snakemake >=6.12.1, then arrange GEX (and ADT) FASTQs per sample under input_fastqs_gex/input_fastqs_adt using the SAMPLENAME_S[Number]_L00[LaneNumber]_[ReadType]_001.fastq.gz schema, with paths specified relative to the working directory. Install cellranger manually and add its binary to PATH since it cannot come from conda; run 'snakemake --use-conda --printshellcmds --cores 1' so cellranger count processes raw reads and the embedded scAmpi module takes over QC. scAmpi's QC step removes contamination such as likely empty droplets or ambient RNA, then normalises, clusters, assigns cell types and performs pathway/DE analysis on the GEX matrix. Adapt config.yaml (inputOutput, resources sections) and scAmpi-specific entries per the scAmpi_single_cell_RNA repository; validate first with a dry-run of the rulegraph. The quick test run on three hashed human PBMC samples can start after the resource-intensive cellranger count and CITE-Seq steps using dummy FASTQ files.

## Related tools

- **gExcite** (Start-to-end Snakemake workflow that embeds scAmpi for GEX preprocessing alongside ADT and hashing analysis) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **scAmpi** (Embedded module performing QC/empty-droplet filtering, normalisation, clustering, cell type classification and DE analysis on the GEX matrix) — https://github.com/ETH-NEXUS/scAmpi_single_cell_RNA
- **cellranger** (Read mapping and UMI counting from raw FASTQs; must be installed manually and added to PATH)
- **snakedeploy** (Deploys the gExcite_pipeline workflow from GitHub into the working directory) — https://github.com/ETH-NEXUS/gExcite_pipeline
- **Snakemake** (Workflow manager (>=6.12.1) orchestrating rules, --use-conda environment provisioning, and module inclusion of scAmpi)

## Examples

```
snakedeploy deploy-workflow https://github.com/ETH-NEXUS/gExcite_pipeline --tag main . && snakemake --use-conda --printshellcmds --cores 1
```

## Evaluation signals

- Snakemake dry-run ('--dry-run') resolves the rulegraph without missing-input or config errors before the full run
- All Snakemake rules complete without error on the three-sample hashed PBMC test data
- Documented scAmpi GEX outputs (QC filtering, clustering, cell type assignment, pathway enrichment) are present in the results directory for all three samples
- FASTQ files match the SAMPLENAME_S[Number]_L00[LaneNumber]_[ReadType]_001.fastq.gz schema and sit in per-sample subdirectories relative to the working directory; SAMPLENAME contains only [a-zA-Z0-9_-]+ characters
- The workflow completed hashing deconvolution, GEX analysis, and CITE-seq/ADT analysis end to end

## Limitations

- The workflow is compatible with and tested on Linux only; other Unix systems including MacOS are currently not supported.
- Cellranger (and STARsolo) cannot be installed via conda and must be installed manually with the binary added to PATH.
- Snakemake needs internet access for snakedeploy/module set-up; some scAmpi steps also perform online queries.
- Sample names must contain only [a-zA-Z0-9_-]+ characters or cellranger will stop immediately.
- In case of combined GEX and ADT NovaSeq sequencing data, index-hopping cleanup scripts must be run before the pipeline.

## Evidence

- [methods] gExcite is a start-to-end workflow embedded in Snakemake: "gExcite is a start-to-end workflow embedded in Snakemake that provides both, gene expression and CITE-seq analysis, as well as hashing deconvolution"
- [methods] scAmpi QC description: "scAmpi, a workflow that provides basic scRNA processing steps, is included as a module into gExcite"
- [results] QC filter description: [evidence span withheld — anchor status: not_found]
- [readme] FASTQ naming schema: "The pipeline expects the FASTQ files per sample to be in the following folder structure, adhering to the naming schema"
- [readme] cellranger installation requirement: "Follow the instructions on the 10xGenomics installation support page to install cellranger and to include the cellranger binary to your path"
- [readme] sample name restriction: [evidence span withheld — anchor status: not_found]
