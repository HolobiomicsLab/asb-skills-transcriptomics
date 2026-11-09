---
name: variable-feature-selection-in-single-cell-data
description: Use when a normalized and scaled scRNA-seq expression matrix (e.g. PBMC3k)
  must be reduced to its most informative genes before running PCA and non-linear
  2D/3D embeddings in Seurat or SCALA.
when_to_use_negative:
- Input is scATAC-seq data — SCALA uses LSI dimensionality reduction for chromatin
  accessibility, not VST-based variable gene selection
- PCA or embeddings have already been computed on a fixed feature set and only visualization/export
  of coordinates is needed
- The dataset is not human or mouse — SCALA currently supports only human and mouse
  datasets
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCALA
  role: Interactive pipeline exposing the variable-feature detection step (VST) and
    its downstream PCA/t-SNE/UMAP usage
- name: Seurat
  role: R library in which the scaled variable features are detected and passed to
    RunPCA and non-linear embeddings
  repo: https://github.com/satijalab/seurat
- name: R
  role: Implementation language for the SCALA pipeline and Seurat feature-selection
    step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:d1f536f7664dab64977e7e68d10c7b8159ce1eb056ca5f5c8c8e619aa11bb14c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/satijalab/seurat
  tools:
  - R
  - SCALA
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# variable-feature-selection-in-single-cell-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects the most highly variable genes in a normalized scRNA-seq matrix (e.g. via Seurat's Variance Stabilizing Transformation, default 2000 features) so that downstream PCA, t-SNE and UMAP focus on informative genes rather than noise, as implemented in SCALA's dimensionality-reduction layer.

## When to use

Use when a normalized and scaled scRNA-seq expression matrix (e.g. PBMC3k) must be reduced to its most informative genes before running PCA and non-linear 2D/3D embeddings in Seurat or SCALA.

## When NOT to use

- Input is scATAC-seq data — SCALA uses LSI dimensionality reduction for chromatin accessibility, not VST-based variable gene selection
- PCA or embeddings have already been computed on a fixed feature set and only visualization/export of coordinates is needed
- The dataset is not human or mouse — SCALA currently supports only human and mouse datasets

## Inputs

- Normalized and scaled gene-by-cell scRNA-seq expression matrix (e.g. PBMC3k) in a Seurat object
- Upstream normalization/scaling outputs and metadata (cell barcodes)

## Outputs

- Set of highly variable features (1535 variable genes in the SCALA PBMC3k run; VST default nfeatures = 2000)
- Variable-feature ranking used as input to PCA (25 informative PCs) and downstream t-SNE/UMAP embeddings

## How to apply

Load the normalized and scaled gene-by-cell matrix into a Seurat object together with detected variable features; SCALA uses Seurat's Variance Stabilizing Transformation (VST) with a default of 2000 features to rank genes by variance. In the SCALA PBMC3k run this step led to the identification of 1535 variable genes, which are the only features used for subsequent PCA on the scaled values. Decision point: if few informative genes are recovered, revisit upstream normalization/scaling rather than forcing all genes into PCA. The selected variable features then feed PCA, whose informative PC count (25 PCs for PBMC3k) is set automatically via 10-fold SVD cross-validation, so over- or under-inclusive feature selection directly propagates into the t-SNE/UMAP projections.

## Related tools

- **SCALA** (Interactive pipeline exposing the variable-feature detection step (VST) and its downstream PCA/t-SNE/UMAP usage)
- **Seurat** (R library in which the scaled variable features are detected and passed to RunPCA and non-linear embeddings) — https://github.com/satijalab/seurat
- **R** (Implementation language for the SCALA pipeline and Seurat feature-selection step)

## Examples

```
pbmc <- FindVariableFeatures(pbmc, selection.method = "vst", nfeatures = 2000)
```

## Evaluation signals

- dim_reduction_embeddings.csv exists with one row per cell and coordinate columns for PCA (25 PCs), t-SNE and UMAP
- Number of variable genes reported is finite and plausible relative to the VST cap (SCALA PBMC3k reported 1535 variable genes out of a 2000-feature default)
- Subsequent SVD 10-fold cross-validation selects a stable number of informative PCs (25 PCs in the reference PBMC3k run)
- UMAP/t-SNE computed on PCs from the selected features reproduces the clustered PBMC3k structure shown in Fig. 2B
- Only the detected variable features contribute to the PCA loadings, not the full gene set

## Limitations

- Only human and mouse datasets are currently supported for scRNA-seq and scATAC-seq analysis in SCALA
- The online SCALA version restricts dataset size to < 2 GB with 2 CPU threads; for > 50,000 cells some steps may not complete and the desktop version is advised
- Software version numbers for Seurat employed in SCALA are not stated, so feature-selection behavior may differ from the publication's vignettes

## Evidence
