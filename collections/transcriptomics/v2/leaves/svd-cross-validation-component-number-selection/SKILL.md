---
name: svd-cross-validation-component-number-selection
description: Use when a normalized, scaled scRNA-seq matrix has been PCA-transformed
  in Seurat and the number of informative PCs feeding t-SNE/UMAP or clustering must
  be chosen objectively rather than by eye.
when_to_use_negative:
- Input is scATAC-seq chromatin accessibility data, where SCALA uses Latent Semantic
  Indexing (LSI) dimensionality reduction instead of PCA
- Input is already a reduced-dimension embedding or a fixed PC set supplied by the
  user, so re-selecting component numbers would be redundant
- The analysis goal is a fixed-dimensionality embedding for direct comparison, not
  choosing an intrinsic dimensionality
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCALA
  role: Interactive pipeline that wraps the SVD cross-validation PC-number selection
    as its PCA Dimensionality Reduction step
- name: Seurat
  role: R library in which PCA is run on scaled variable features and the number of
    informative PCs is determined via 10-fold SVD cross-validation before t-SNE/UMAP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:35b8213a1756250f16ec45f06f7c861c97633cb55e4a6a468439eeefef061cf1
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# SVD cross-validation component number selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automatically selects the number of informative principal components after PCA by 10-fold Singular Value Decomposition (SVD) cross-validation in Seurat, with an elbow plot of incremental per-PC variance as fallback; SCALA's PBMC3k run selected 25 PCs to feed t-SNE and UMAP.

## When to use

Use when a normalized, scaled scRNA-seq matrix has been PCA-transformed in Seurat and the number of informative PCs feeding t-SNE/UMAP or clustering must be chosen objectively rather than by eye.

## When NOT to use

- Input is scATAC-seq chromatin accessibility data, where SCALA uses Latent Semantic Indexing (LSI) dimensionality reduction instead of PCA
- Input is already a reduced-dimension embedding or a fixed PC set supplied by the user, so re-selecting component numbers would be redundant
- The analysis goal is a fixed-dimensionality embedding for direct comparison, not choosing an intrinsic dimensionality

## Inputs

- Seurat object with normalized, scaled gene-by-cell expression matrix from PBMC3k scRNA-seq data (or equivalent)
- Detected variable features (e.g. ~2000 features from Variance Stabilizing Transformation)
- PCA embedding computed on the scaled variable features

## Outputs

- Number of informative principal components (25 PCs for SCALA's PBMC3k run)
- Per-cell PCA coordinates (25 PCs) exported in dim_reduction_embeddings.csv with cluster-independent cell barcodes
- Optional PC loading scores / elbow plot of incremental per-PC variance

## How to apply

Run PCA in Seurat on the scaled values of the detected variable features (e.g. ~2000 features identified by Variance Stabilizing Transformation in the PBMC3k workflow). Score PCs with 10-fold SVD cross-validation, which estimates how well each PC generalizes, and take the number of informative components from the cross-validation optimum — SCALA's PBMC3k run determined 25 PCs this way. Optionally inspect loading scores of top genes of a principal component to check PC composition. If the automatic criterion is ambiguous, fall back to inspection of the elbow plot of incremental per-PC variance. The chosen PCs then serve as input for non-linear dimensionality reduction (t-SNE, UMAP) and downstream clustering.

## Related tools

- **SCALA** (Interactive pipeline that wraps the SVD cross-validation PC-number selection as its PCA Dimensionality Reduction step)
- **Seurat** (R library in which PCA is run on scaled variable features and the number of informative PCs is determined via 10-fold SVD cross-validation before t-SNE/UMAP)

## Examples

```
pbmc <- RunPCA(pbmc, features = VariableFeatures(pbmc))
```

## Evaluation signals

- dim_reduction_embeddings.csv exists with one row per cell and contains coordinate columns for PCA (25 PCs), t-SNE and UMAP
- The selected PC count is a positive integer reported with its selection rationale (SVD cross-validation optimum, or elbow plot if fallback was used)
- The UMAP projection computed on the selected PCs reproduces the clustered structure shown in the PBMC3k analysis (Fig. 2B)
- PCA was computed on the scaled values of the most variable features only (e.g. the ~2000 VST-selected features), not all genes

## Limitations

- The SVD cross-validation criterion is dataset-dependent: 25 PCs was determined for the PBMC3k dataset and will not necessarily transfer to other datasets
- Software version numbers for Seurat employed in SCALA are not stated, so exact reproducibility of the PC-number determination may vary
- SCALA supports only human and mouse datasets; the online version is restricted to < 2 GB datasets and cannot complete some steps for datasets larger than 50,000 cells

## Evidence

- [methods] PCA on scaled variable features with 10-fold SVD CV and elbow-plot fallback: [evidence span withheld — anchor status: not_found]
- [results] Selected PCs feed t-SNE and UMAP embeddings: [evidence span withheld — anchor status: not_found]
- [methods] Variable features from VST, default 2000 features: [evidence span withheld — anchor status: not_found]
