---
name: mitochondrial-content-qc-metric-assessment
description: Use when a raw scRNA-seq UMI count matrix (e.g., 10x cellranger filtered_bc_matrix
  or tab-delimited gene-by-cell table) must be quality-filtered and per-cell mitochondrial
  percentage is needed to flag dying cells or doublets.
when_to_use_negative:
- Input is already QC-filtered or provided as a normalized/scaled expression matrix
  — mitochondrial QC would be redundant or invalid
- The assay is scATAC-seq or another modality where mitochondrial-percentage semantics
  differ from scRNA-seq transcript quality
- The organism/dataset is outside the human/mouse scope currently supported by SCALA
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCALA
  role: Interactive web/desktop pipeline whose scRNA-seq QC module computes per-cell
    unique features, UMIs, and mitochondrial percentage and renders QC violin plots
- name: Seurat
  role: 'R library underlying SCALA''s scRNA-seq QC: object creation from the UMI
    count matrix and per-cell mito/UMI/feature metric computation and violin-plot
    visualization'
- name: R
  role: Implementation language in which the QC filtering and metric visualization
    are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:92337894a49291c60ab8e254d97e414dd207dafbfb2acca1c62cee5c80d06278
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  tools:
  - R
  - SCALA
  - Seurat
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Mitochondrial content QC metric assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assessment of per-cell mitochondrial read percentage as a core scRNA-seq quality-control metric, used alongside per-cell unique-feature counts and UMI totals to exclude low-quality/dying cells and putative multiplets before normalization, as implemented in the SCALA/Seurat PBMC3k pipeline.

## When to use

Use when a raw scRNA-seq UMI count matrix (e.g., 10x cellranger filtered_bc_matrix or tab-delimited gene-by-cell table) must be quality-filtered and per-cell mitochondrial percentage is needed to flag dying cells or doublets.

## When NOT to use

- Input is already QC-filtered or provided as a normalized/scaled expression matrix — mitochondrial QC would be redundant or invalid
- The assay is scATAC-seq or another modality where mitochondrial-percentage semantics differ from scRNA-seq transcript quality
- The organism/dataset is outside the human/mouse scope currently supported by SCALA

## Inputs

- Raw scRNA-seq UMI count matrix (10x cellranger filtered_bc_matrix format or tab-delimited gene-by-cell table, e.g., PBMC3k)
- Gene-to-mitochondrial gene list annotation (mito gene prefixes) for per-cell percentage computation

## Outputs

- Filtered Seurat object (RDS) with per-cell QC metadata (nFeature_RNA, nCount_RNA, percent.mt)
- QC violin plots (PNG) showing genes detected, total reads/UMIs, and mitochondrial percentage per cell
- Threshold-selected cell subset retained for downstream normalization and VST variable-feature detection

## How to apply

Load the UMI count matrix (e.g., PBMC3k in 10x cellranger filtered_bc_matrix format) into a Seurat object and compute per-cell metrics: number of unique features, total UMIs, and percentage of mitochondrial content. Filter out low-quality/dying cells (high mitochondrial percentage) and putative multiplets (very high feature/UMI counts), then visualize the three metrics as violin plots for threshold inspection (as in the paper's Fig. 2A). Only after this QC step proceed to global-scaling normalization (counts per barcode scaled to 10,000 and log-transformed), z-score scaling, and VST-based highly variable feature detection, which in the SCALA reanalysis identified 1535 variable genes. The rationale is that high mitochondrial fraction is a proxy for cytoplasmic RNA loss in damaged cells, so filtering on it upstream prevents degraded cells from distorting downstream clustering.

## Related tools

- **SCALA** (Interactive web/desktop pipeline whose scRNA-seq QC module computes per-cell unique features, UMIs, and mitochondrial percentage and renders QC violin plots)
- **Seurat** (R library underlying SCALA's scRNA-seq QC: object creation from the UMI count matrix and per-cell mito/UMI/feature metric computation and violin-plot visualization)
- **R** (Implementation language in which the QC filtering and metric visualization are executed)

## Evaluation signals

- The QC violin plots display all three metrics — genes detected, total reads (UMIs), and mitochondrial percentage — per cell, matching the layout of the paper's Fig. 2A
- Cells with high mitochondrial percentage (dying cells) and cells with very high feature/UMI counts (putative multiplets) are absent from the filtered object
- The retained cell population is strictly a subset of the input barcodes, and per-cell percent.mt values fall within the accepted threshold range
- Downstream VST variable-feature detection on the filtered, normalized matrix returns a well-defined variable-gene table (SCALA's PBMC3k run identified 1535 variable genes)

## Limitations

- SCALA currently supports analysis of only human and mouse datasets
- The online SCALA version restricts dataset size to < 2 GB and allocates no more than 2 CPU threads per session; for datasets larger than 50,000 cells some steps cannot be completed online and the desktop version is recommended
- Software version numbers for Seurat employed in SCALA are not stated, so exact QC-function behavior may differ across Seurat versions

## Evidence
