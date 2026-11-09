---
name: tsne-umap-embedding-generation
description: Use when a normalized/scaled scRNA-seq matrix with detected variable
  features already has PCA scores and you need 2D/3D cluster-visualizing cell projections
  (t-SNE and UMAP coordinates per cell).
when_to_use_negative:
- Input has not been normalized, scaled, and reduced by PCA — t-SNE/UMAP should run
  on informative PCs, not raw counts
- Dataset exceeds the online SCALA limits (dataset size > 2 GB or > 50,000 cells),
  where some steps cannot be completed; prefer the desktop version
- Goal is dataset integration across conditions rather than visualization — SCALA's
  RPCA-based integration mode and side-to-side comparison mode are not yet implemented
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCALA
  role: Interactive web/desktop pipeline whose dimensionality-reduction layer runs
    PCA, then t-SNE and UMAP, on scRNA-seq data without coding
- name: Seurat
  role: R library executing PCA, RunTSNE and RunUMAP on the scaled variable features
    of the Seurat object
  repo: https://github.com/scottgigante/seurat
- name: R
  role: Language in which SCALA and the Seurat-based embedding workflow are implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:feb1fec92413a6c18bcfad97c1b5ef995759c6543dedf6b99dbe00813eeacfe6
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tsne-umap-embedding-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates non-linear 2D cell embeddings (t-SNE and UMAP) in Seurat from the top principal components of a scaled scRNA-seq matrix, as performed in SCALA's PBMC3k demonstration where 25 informative PCs fed both embeddings.

## When to use

Use when a normalized/scaled scRNA-seq matrix with detected variable features already has PCA scores and you need 2D/3D cluster-visualizing cell projections (t-SNE and UMAP coordinates per cell).

## When NOT to use

- Input has not been normalized, scaled, and reduced by PCA — t-SNE/UMAP should run on informative PCs, not raw counts
- Dataset exceeds the online SCALA limits (dataset size > 2 GB or > 50,000 cells), where some steps cannot be completed; prefer the desktop version
- Goal is dataset integration across conditions rather than visualization — SCALA's RPCA-based integration mode and side-to-side comparison mode are not yet implemented

## Inputs

- Normalized and scaled gene-by-cell scRNA-seq expression matrix (e.g. PBMC3k) loaded in a Seurat object
- Detected variable features (e.g. ~2000 features via Variance Stabilizing Transformation)
- PCA embeddings with a chosen number of informative PCs (e.g. 25 PCs selected by SVD cross-validation)

## Outputs

- Per-cell t-SNE 2D coordinates
- Per-cell UMAP 2D coordinates
- Exported dim_reduction_embeddings.csv with one row per cell containing barcodes plus PCA (25 PCs), t-SNE, and UMAP coordinate columns

## How to apply

Load the normalized and scaled PBMC3k-style matrix into a Seurat object together with variable features identified by Variance Stabilizing Transformation (default 2000 features). Run PCA on the scaled values of the most variable features and select informative PCs — SCALA determined the count automatically via 10-fold SVD cross-validation, falling back to elbow-plot inspection of incremental per-PC variance; the PBMC3k run settled on 25 PCs. Optionally inspect the loading scores of top genes of a principal component to check PC composition. Then compute RunTSNE and RunUMAP in Seurat on the selected PCs to obtain 2D cell projections, and export a per-cell table of cluster-independent barcodes with PCA (25 PCs), t-SNE, and UMAP coordinates.

## Related tools

- **SCALA** (Interactive web/desktop pipeline whose dimensionality-reduction layer runs PCA, then t-SNE and UMAP, on scRNA-seq data without coding)
- **Seurat** (R library executing PCA, RunTSNE and RunUMAP on the scaled variable features of the Seurat object) — https://github.com/scottgigante/seurat
- **R** (Language in which SCALA and the Seurat-based embedding workflow are implemented)

## Examples

```
pbmc <- RunPCA(pbmc, features = VariableFeatures(object = pbmc)); pbmc <- RunTSNE(pbmc, dims = 1:25); pbmc <- RunUMAP(pbmc, dims = 1:25)
```

## Evaluation signals

- dim_reduction_embeddings.csv exists with exactly one row per cell (cluster-independent barcodes)
- The file contains coordinate columns for all 25 PCA dimensions plus t-SNE and UMAP
- The UMAP projection reproduces the clustered structure shown in the PBMC3k analysis (Fig. 2B)
- The number of PCs fed to t-SNE/UMAP matches the 25 informative PCs determined by 10-fold SVD cross-validation (or is justified by elbow-plot inspection)
- Loading scores of top genes of a principal component were inspected to confirm sensible PC composition

## Limitations

- The number of informative PCs relies on 10-fold SVD cross-validation with elbow-plot inspection as fallback, which is a decision point that can alter downstream embeddings
- SCALA's online version restricts datasets to < 2 GB and 2 CPU threads per session, and cannot complete some steps for datasets > 50,000 cells
- A comparison mode for side-to-side visualization of dimensionality reduction plots across conditions is not yet implemented
- Only human and mouse datasets are currently supported for scRNA-seq and scATAC-seq analysis

## Evidence

- [methods] Workflow description of PCA feeding t-SNE/UMAP: [evidence span withheld — anchor status: not_found]
