---
name: single-cell-rna-seq-global-scaling-normalization
description: Use when raw UMI count matrices from a droplet-based scRNA-seq experiment
  (e.g. 10x cellranger filtered_bc_matrix or gene-by-cell tables) need QC, normalization,
  scaling and variable-feature detection prior to PCA/clustering.
when_to_use_negative:
- Input is already normalized/scaled or an HVG feature table rather than raw UMI counts
- Data are scATAC-seq epigenomes (e.g. BMMCs) — use the LSI-based SCALA module instead
- Non-human/non-mouse datasets — SCALA currently supports only human and mouse
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCALA
  role: Interactive pipeline that exposes the QC, normalization/scaling and variable-feature
    detection steps as GUI-based modules
- name: Seurat
  role: R library whose NormalizeData/ScaleData/FindVariableFeatures (VST) functions
    underlie SCALA's scRNA-seq preprocessing, including the mean-variance-plot (MVP)
    HVG method
  repo: https://github.com/scottgigante/seurat
- name: R
  role: Implementation language for SCALA's scRNA-seq preprocessing module
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:35b5818ad65565a4d2f6f794fb0a540be8892a4aefcd6bd4334c08e40832bb94
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/scottgigante/seurat
  tools:
  - R
  - SCALA
  - Seurat
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-rna-seq-global-scaling-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the SCALA/Seurat scRNA-seq preprocessing module for the PBMC3k dataset: QC filtering on genes detected, UMI counts and mitochondrial percentage, followed by global-scaling normalization (counts per barcode scaled to 10,000 and log-transformed), z-score scaling, and VST-based highly variable feature detection that identified 1535 variable genes.

## When to use

Use when raw UMI count matrices from a droplet-based scRNA-seq experiment (e.g. 10x cellranger filtered_bc_matrix or gene-by-cell tables) need QC, normalization, scaling and variable-feature detection prior to PCA/clustering.

## When NOT to use

- Input is already normalized/scaled or an HVG feature table rather than raw UMI counts
- Data are scATAC-seq epigenomes (e.g. BMMCs) — use the LSI-based SCALA module instead
- Non-human/non-mouse datasets — SCALA currently supports only human and mouse

## Inputs

- scRNA-seq UMI count matrix in 10x cellranger filtered_bc_matrix format
- Tab-delimited gene-by-cell count table
- Per-cell metadata covariates for optional regression during scaling

## Outputs

- Preprocessed Seurat object saved as RDS
- Highly variable feature table saved as CSV
- QC violin plots (genes detected, total UMIs, mitochondrial percentage) saved as PNG

## How to apply

Load the UMI count matrix into a Seurat object, then filter cells on the three standard QC metrics — number of unique features, total UMIs, and mitochondrial content percentage — dropping high-mito (dying) cells and very-high-feature/UMI (putative multiplet) cells while visualizing metrics as violin plots (as in the paper's Fig. 2A). Normalize each barcode by dividing gene counts by total barcode counts, multiplying by 10,000, and log-transforming (global-scaling normalization). Scale normalized counts to z-scores (mean 0, variance 1 per feature), optionally regressing out metadata covariates. Detect highly variable features with Seurat's Variance Stabilizing Transformation (VST), which fits a local polynomial regression to the log-variance vs log-mean relationship, standardizes features by observed mean and expected variance, clips to a maximum, and returns a fixed number of features (default 2000); SCALA's PBMC3k run returned 1535 variable genes. Persist the preprocessed Seurat object (RDS), variable-feature table (CSV), and QC violin plots (PNG).

## Related tools

- **SCALA** (Interactive pipeline that exposes the QC, normalization/scaling and variable-feature detection steps as GUI-based modules)
- **Seurat** (R library whose NormalizeData/ScaleData/FindVariableFeatures (VST) functions underlie SCALA's scRNA-seq preprocessing, including the mean-variance-plot (MVP) HVG method) — https://github.com/scottgigante/seurat
- **R** (Implementation language for SCALA's scRNA-seq preprocessing module)

## Examples

```
Rscript -e 'pbmc <- CreateSeuratObject(counts); pbmc <- NormalizeData(pbmc, normalization.method="LogNormalize", scale.factor=10000); pbmc <- ScaleData(pbmc); pbmc <- FindVariableFeatures(pbmc, method="vst", nfeatures=2000)'
```

## Evaluation signals

- The HVG table contains the expected number of features — 1535 variable genes for the PBMC3k run (default 2000 under VST)
- QC violin plots display all three metrics: genes detected, total reads/UMIs, and mitochondrial percentage, matching the paper's Fig. 2A
- Normalized values correspond to log10(total counts per 10,000 per barcode), i.e. counts × 10,000 followed by log transformation
- Scaled expression matrix has column-wise mean 0 and variance 1 per feature (z-scores)
- Preprocessed artifacts (RDS object, CSV HVG table, PNG violin plots) are all present and loadable

## Limitations

- Only human and mouse datasets are currently supported in both modalities
- The online SCALA version restricts dataset size to < 2 GB and 2 CPU threads per session; datasets > 50,000 cells should use the desktop version
- Seurat/ArchR software version numbers used by SCALA are not stated, complicating exact reproduction

## Evidence

- [methods] each barcode's gene counts are normalized by total barcode counts, multiplied by 10,000, and log-transformed: [evidence span withheld — anchor status: not_found]
