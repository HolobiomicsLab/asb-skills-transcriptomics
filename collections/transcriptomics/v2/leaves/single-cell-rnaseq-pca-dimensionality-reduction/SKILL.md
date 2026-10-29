---
name: single-cell-rnaseq-pca-dimensionality-reduction
description: Use when a normalized and scaled gene-by-cell scRNA-seq matrix (e.g.
  PBMC3k with VST-derived variable features) needs PCA followed by t-SNE/UMAP 2D embeddings
  in a Seurat/SCALA workflow.
when_to_use_negative:
- Input is raw count data that has not been normalized and scaled — run upstream normalization/scaling
  and variable-feature detection first
- The modality is scATAC-seq — SCALA uses LSI dimensionality reduction for that modality,
  not PCA on gene expression
- Datasets larger than 50,000 cells in the SCALA online (web-server) version, where
  some steps cannot be completed; prefer the desktop version
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SCALA
  role: Pipeline hosting the dimensionality-reduction module; exported per-cell PCA/t-SNE/UMAP
    embedding tables in the PBMC3k use-case
- name: Seurat
  role: Runs PCA on scaled variable features and computes the t-SNE and UMAP non-linear
    embeddings from the selected PCs
- name: R
  role: Implementation language in which SCALA and the Seurat-based PCA/t-SNE/UMAP
    steps are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:96cfdf38cc498937759fd8af352328a2b3beb1e3d343ec40740819171a4c8b7a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-rnaseq-pca-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs SCALA's scRNA-seq dimensionality-reduction layer: PCA on the scaled variable-feature matrix (Seurat, PBMC3k), selection of informative PCs via SVD cross-validation, and t-SNE/UMAP embeddings of the top PCs for 2D visualization.

## When to use

Use when a normalized and scaled gene-by-cell scRNA-seq matrix (e.g. PBMC3k with VST-derived variable features) needs PCA followed by t-SNE/UMAP 2D embeddings in a Seurat/SCALA workflow.

## When NOT to use

- Input is raw count data that has not been normalized and scaled — run upstream normalization/scaling and variable-feature detection first
- The modality is scATAC-seq — SCALA uses LSI dimensionality reduction for that modality, not PCA on gene expression
- Datasets larger than 50,000 cells in the SCALA online (web-server) version, where some steps cannot be completed; prefer the desktop version

## Inputs

- Normalized and scaled PBMC3k gene-by-cell scRNA-seq expression matrix (from upstream preprocessing)
- Detected variable features (VST, default 2000 features; 1535 identified in the SCALA run)
- Seurat object containing the scaled data

## Outputs

- dim_reduction_embeddings.csv with one row per cell and coordinate columns for PCA (25 PCs), t-SNE, and UMAP
- Elbow plot of incremental per-PC variance (informative-PC selection)
- PC loading-score plots of top genes per principal component
- 2D t-SNE and UMAP cell projections reproducing the clustered structure (Fig. 2B)

## How to apply

Load the normalized and scaled PBMC3k matrix into a Seurat object together with variable features detected via Variance Stabilizing Transformation (default 2000 features; the SCALA run identified 1535 variable genes). Run PCA in Seurat on the scaled values of the most variable features and determine the number of informative PCs automatically via 10-fold Singular Value Decomposition (SVD) cross-validation, falling back to elbow-plot inspection of incremental per-PC variance — the SCALA PBMC3k run determined 25 informative PCs. Optionally inspect PC composition by plotting loading scores of the PC's top genes. Compute t-SNE and UMAP on the selected 25 PCs to obtain 2D cell projections, and export a per-cell table of cluster-independent barcodes with PCA, t-SNE, and UMAP coordinates.

## Related tools

- **SCALA** (Pipeline hosting the dimensionality-reduction module; exported per-cell PCA/t-SNE/UMAP embedding tables in the PBMC3k use-case)
- **Seurat** (Runs PCA on scaled variable features and computes the t-SNE and UMAP non-linear embeddings from the selected PCs)
- **R** (Implementation language in which SCALA and the Seurat-based PCA/t-SNE/UMAP steps are executed)

## Examples

```
pbmc <- RunPCA(pbmc, features = VariableFeatures(pbmc), npcs = 25); pbmc <- RunTSNE(pbmc, dims = 1:25); pbmc <- RunUMAP(pbmc, dims = 1:25)
```

## Evaluation signals

- dim_reduction_embeddings.csv exists with exactly one row per cell (cluster-independent barcodes)
- Embedding table contains coordinate columns for PCA (25 PCs), t-SNE, and UMAP
- SVD 10-fold cross-validation (or elbow-plot inspection) converges on the expected number of informative PCs (25 in the PBMC3k run)
- UMAP projection visually reproduces the clustered structure shown in the PBMC3k analysis (Fig. 2B)
- Variable-feature detection upstream yields a plausible count (e.g. 1535 variable genes in the SCALA PBMC3k run; VST default 2000)

## Limitations

- Only human and mouse datasets are currently supported for both scRNA-seq and scATAC-seq analysis
- Online SCALA version restricts dataset size to < 2 GB with no more than 2 CPU threads per session, and cannot complete some steps for datasets larger than 50,000 cells (desktop version recommended)
- Software version numbers for Seurat employed in SCALA are not stated, which may cause differences vs. the published results
- A side-to-side comparison mode for dimensionality-reduction/feature-expression plots is not yet implemented (promised in the near future)

## Evidence
