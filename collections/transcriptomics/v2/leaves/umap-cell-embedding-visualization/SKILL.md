---
name: umap-cell-embedding-visualization
description: Use when a scRNA-seq or scATAC-seq dataset has already been quality-controlled
  and dimension-reduced (PCA or iterative LSI) and a 2D visualization of cells colored
  by Louvain/Seurat cluster identity is needed.
when_to_use_negative:
- The dataset is from an organism other than human or mouse, since SCALA supports
  analysis of only human and mouse datasets
- The input is raw sequencing data that has not yet undergone QC, normalization, and
  dimensionality reduction — UMAP should not be the first step
- Only a numeric summary table is needed without spatial cell embedding — dimensionality-reduction
  plotting adds no value
edam_operation: http://edamontology.org/operation_3935
tools:
- name: ArchR
  role: Computes the 2D UMAP embedding of cells in LSI space for the scATAC-seq pipeline
    (SNN graph + Louvain + UMAP)
  repo: https://github.com/GreenleafLab/ArchR
- name: Seurat
  role: Provides the UMAP embedding of scRNA-seq cells on selected PCs inside SCALA
    (patched build for PHATE compatibility)
  repo: https://github.com/scottgigante/seurat
- name: SCALA
  role: Shiny application that orchestrates the UMAP run and renders publication-ready,
    interactive UMAP plots colored by cluster
  repo: https://github.com/PavlopoulosLab/SCALA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:e43feaafbba073505714f5c61970c7dda0a13fcb577737508629800cb99d5d55
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/GreenleafLab/ArchR
  tools:
  - ArchR
  - R
  - SCALA
  - Seurat
  edam_operation: http://edamontology.org/operation_3935
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# umap-cell-embedding-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Embed single-cell data in 2D with UMAP on top of a reduced-dimension representation (PCA for scRNA-seq, iterative LSI for scATAC-seq) within SCALA's Seurat/ArchR-based pipeline, and render the embedding colored by cluster identity as publication-ready PNG/plotly figures.

## When to use

Use when a scRNA-seq or scATAC-seq dataset has already been quality-controlled and dimension-reduced (PCA or iterative LSI) and a 2D visualization of cells colored by Louvain/Seurat cluster identity is needed.

## When NOT to use

- The dataset is from an organism other than human or mouse, since SCALA supports analysis of only human and mouse datasets
- The input is raw sequencing data that has not yet undergone QC, normalization, and dimensionality reduction — UMAP should not be the first step
- Only a numeric summary table is needed without spatial cell embedding — dimensionality-reduction plotting adds no value

## Inputs

- ArchR Arrow file / ArrowProject with an IterativeLSI reduced-dimension matrix (scATAC-seq, e.g. BMMC fragments-derived)
- Seurat object with PCA (or patched Seurat with PHATE) reduction computed on variable genes
- Louvain/Seurat cluster assignments per cell

## Outputs

- 2D UMAP cell coordinates per cell
- UMAP projection PNG colored by cluster identity (Fig. 3C style)
- Interactive plotly/ggplot2 UMAP visualization in the SCALA Shiny interface

## How to apply

For the BMMCs scATAC-seq use-case, run UMAP on the iterative LSI reduction (LSI with resolution 0.6, 30 dimensions, 4 iterations on genome-wide 500-bp tile counts) by building a Shared-Nearest-Neighbor graph in LSI space, partitioning it into Louvain communities, and embedding cells in 2D via ArchR within SCALA's create_arrow_file workflow. For scRNA-seq, UMAP is computed on the selected principal components (e.g., the 25 PCs determined by SVD k-fold cross-validation) through Seurat, optionally using the patched Seurat build that supports extra embeddings. Export the UMAP projection colored by cluster identity as a PNG (Fig. 3C style), alongside the QC plots (Fig. 3A style) and cluster abundance bar plot (Fig. 3B style); SCALA's Shiny/plotly front end renders the embedding interactively. Validate that clusters are spatially coherent in the 2D projection and consistent with the SNN/Louvain partitioning before using the plot for downstream annotation.

## Related tools

- **ArchR** (Computes the 2D UMAP embedding of cells in LSI space for the scATAC-seq pipeline (SNN graph + Louvain + UMAP)) — https://github.com/GreenleafLab/ArchR
- **Seurat** (Provides the UMAP embedding of scRNA-seq cells on selected PCs inside SCALA (patched build for PHATE compatibility)) — https://github.com/scottgigante/seurat
- **SCALA** (Shiny application that orchestrates the UMAP run and renders publication-ready, interactive UMAP plots colored by cluster) — https://github.com/PavlopoulosLab/SCALA

## Examples

```
ArchRProj <- addUMAP(ArchRProj = ArchRProj, reducedDims = "IterativeLSI", name = "UMAP", plotUMAP = TRUE)
```

## Evaluation signals

- UMAP PNG output exists and is colored by cluster identity in the Fig. 3C style, accompanied by Fig. 3A-style QC plots and a Fig. 3B-style cluster abundance bar plot
- Cells of the same Louvain community form spatially contiguous islands in the 2D UMAP, consistent with the SNN graph partitioning in LSI space
- The number of visually distinct UMAP islands matches the number of clusters reported in the cluster abundance bar plot
- The embedding is computed from the LSI reduction with the specified settings (resolution 0.6, 30 dimensions, 4 iterations), not from raw tile counts
- Cluster boundaries are stable across re-runs of the reduction+UMAP on the same Arrow file / Seurat object

## Limitations

- In the online version, some steps cannot be completed for datasets larger than 50,000 cells; the desktop SCALA version is recommended for larger datasets
- The comparison mode for side-to-side visualization of dimensionality reduction plots or feature expression plots when more than one condition is present is not yet implemented
- SCALA requires modifications to the Seurat package (PHATE patch), and the GitHub version of phateR; the CRAN phateR version WILL NOT WORK

## Evidence

- [methods] Build a Shared-Nearest-Neighbor graph in the LSI space, partition it into communities with the Louvain algorithm, and embed the cells in 2D with UMAP: [evidence span withheld — anchor status: not_found]
- [methods] Save the QC plots (Fig. 3A style), the cluster abundance bar plot (Fig. 3B style), and the UMAP projection colored by cluster identity (Fig. 3C style) as PNG files: [evidence span withheld — anchor status: not_found]
