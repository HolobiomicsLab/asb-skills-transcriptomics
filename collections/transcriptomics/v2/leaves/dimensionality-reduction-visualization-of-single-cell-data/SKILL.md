---
name: dimensionality-reduction-visualization-of-single-cell-data
description: Use when a normalized, scaled scRNA-seq matrix with detected variable
  features needs to be compressed for visualization or as input to clustering, and
  2D/3D cell projections (PCA, t-SNE, UMAP) must be produced.
when_to_use_negative:
- Input is raw unnormalized counts or a matrix without prior QC, normalization, scaling,
  and variable-feature detection — run the upstream preprocessing steps first.
- The analysis goal is cell-cell communication, GRN reconstruction, or cluster annotation
  rather than low-dimensional projection and visualization of cells.
- Dataset exceeds 50,000 cells and only SCALA's online version is available, since
  some steps cannot be completed there.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCALA
  role: Interactive R/Shiny pipeline whose dimensionality-reduction layer executes
    PCA selection and t-SNE/UMAP embedding on the scaled matrix
- name: Seurat
  role: 'R library underlying SCALA''s scRNA-seq steps: RunPCA on scaled variable
    features, automatic PC number determination via SVD cross-validation, and RunTSNE/RunUMAP
    on selected PCs'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:ec1000f6b848bf0fc6f26db29c4ff3cb5d0b6ad77ef5daf13ee2d3fb5278ee12
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# dimensionality-reduction-visualization-of-single-cell-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Projects a scaled gene-by-cell scRNA-seq matrix into informative low-dimensional spaces: PCA on the most variable features followed by non-linear t-SNE and UMAP embeddings on the selected PCs, as implemented in SCALA's Seurat-based layer (e.g. 25 PCs for the PBMC3k run).

## When to use

Use when a normalized, scaled scRNA-seq matrix with detected variable features needs to be compressed for visualization or as input to clustering, and 2D/3D cell projections (PCA, t-SNE, UMAP) must be produced.

## When NOT to use

- Input is raw unnormalized counts or a matrix without prior QC, normalization, scaling, and variable-feature detection — run the upstream preprocessing steps first.
- The analysis goal is cell-cell communication, GRN reconstruction, or cluster annotation rather than low-dimensional projection and visualization of cells.
- Dataset exceeds 50,000 cells and only SCALA's online version is available, since some steps cannot be completed there.

## Inputs

- Normalized and scaled gene-by-cell scRNA-seq expression matrix loaded in a Seurat object (e.g. PBMC3k)
- Detected variable features (e.g. ~2000 features identified with Variance Stabilizing Transformation)

## Outputs

- dim_reduction_embeddings.csv: one row per cell with cluster-independent cell barcodes and coordinate columns for 25 PCs, t-SNE, and UMAP
- Elbow plot of incremental per-PC variance and PC loading-score plots for top genes
- 2D/3D t-SNE and UMAP cell projections reproducing the clustered structure (e.g. PBMC3k Fig. 2B)

## How to apply

Load the normalized and scaled scRNA-seq matrix (e.g. PBMC3k) into a Seurat object together with the variable features identified with Variance Stabilizing Transformation (default 2000 features). Run PCA in Seurat on the scaled values of the most variable features and determine the number of informative PCs automatically via 10-fold Singular Value Decomposition (SVD) cross-validation, falling back to inspection of the elbow plot of incremental per-PC variance; in SCALA's PBMC3k analysis this procedure determined 25 informative PCs. Optionally inspect the loading scores of the top genes of a principal component to check PC composition. Then compute t-SNE and UMAP embeddings in Seurat on the selected PCs to obtain 2D cell projections that expose the cluster structure. Finally export a per-cell table with cluster-independent barcodes and their PCA, t-SNE, and UMAP coordinates.

## Related tools

- **SCALA** (Interactive R/Shiny pipeline whose dimensionality-reduction layer executes PCA selection and t-SNE/UMAP embedding on the scaled matrix)
- **Seurat** (R library underlying SCALA's scRNA-seq steps: RunPCA on scaled variable features, automatic PC number determination via SVD cross-validation, and RunTSNE/RunUMAP on selected PCs)

## Examples

```
pbmc <- RunPCA(pbmc, features = VariableFeatures(pbmc)); pbmc <- RunTSNE(pbmc, dims = 1:25); pbmc <- RunUMAP(pbmc, dims = 1:25)
```

## Evaluation signals

- dim_reduction_embeddings.csv exists and contains one row per cell with coordinate columns for 25 PCs, t-SNE, and UMAP
- The number of retained PCs (e.g. 25) matches the output of the 10-fold SVD cross-validation (or is defensible from the elbow plot of incremental per-PC variance)
- Barcodes in the embeddings table are cluster-independent and cover every cell from the upstream preprocessing step
- The UMAP projection reproduces the clustered structure shown in the PBMC3k analysis (Fig. 2B)
- PC loading-score inspection shows biologically coherent top genes for leading principal components

## Limitations

- PC-number selection is sensitive to the cross-validation/elbow inspection; falling back to elbow inspection can change the retained dimensionality.
- Non-linear embeddings (t-SNE/UMAP) preserve local but not global structure and are used for visualization of clusters, not quantitative distance inference.
- In SCALA's online version some steps cannot be completed for datasets larger than 50,000 cells; the desktop version is recommended for such datasets.

## Evidence

- [methods] Run PCA in Seurat on the scaled values of the most variable features and determine the number of informative PCs automatically via 10-fold Singular Value Decomposition (SVD) cross validation: [evidence span withheld — anchor status: not_found]
- [results] This procedure determined the number of most informative principal components (25 PCs): "This procedure determined the number of most informative principal components (25 PCs)"
- [methods] Export a per-cell table containing cluster-independent cell barcodes with their PCA, t-SNE, and UMAP coordinates: [evidence span withheld — anchor status: not_found]
- [other] verify dim_reduction_embeddings.csv exists with one row per cell, contains coordinate columns for PCA (25 PCs), tSNE and UMAP: [evidence span withheld — anchor status: not_found]
- [discussion] users are suggested to prefer the desktop version of SCALA for larger datasets (> 50,000 cells): "users are suggested to prefer the desktop version of SCALA for larger datasets (> 50,000 cells)"
