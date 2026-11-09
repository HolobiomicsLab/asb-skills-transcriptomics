---
name: louvain-community-detection-on-cell-graphs
description: Use when a QC-filtered, normalized scRNA-seq object already carries variable
  features and PCA coordinates (e.g. PBMC3k with 25 PCs from 10-fold SVD cross-validation)
  and cell-population cluster labels are needed before marker-gene testing.
when_to_use_negative:
- Input is scATAC-seq reduced dimensions (LSI) rather than scRNA-seq PCA space — SCALA
  handles the modality separately
- The object lacks normalized expression and PCA coordinates (no variable features
  detected), so no meaningful SNN graph can be built
- The goal is marker identification for pre-existing cluster labels rather than discovering
  the clusters themselves
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCALA
  role: Wraps the SNN-graph construction and Louvain clustering step in an interactive
    pipeline ending in marker identification
- name: Seurat
  role: Builds the SNN graph from PCA-space Euclidean distances and runs the Louvain
    partitioning plus one-vs-rest Wilcoxon/MAST/DESeq2 marker tests
  repo: https://github.com/scottgigante/seurat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:3e5313791c6c733b6c45250bb3d598926044680f19d86d0df9b65d61f789b3de
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# louvain-community-detection-on-cell-graphs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partitions cells into transcriptionally coherent clusters by building a shared-nearest-neighbour (SNN) graph over PCA-space embeddings and running Louvain community detection, as implemented in Seurat and used by SCALA on the PBMC3k scRNA-seq dataset.

## When to use

Use when a QC-filtered, normalized scRNA-seq object already carries variable features and PCA coordinates (e.g. PBMC3k with 25 PCs from 10-fold SVD cross-validation) and cell-population cluster labels are needed before marker-gene testing.

## When NOT to use

- Input is scATAC-seq reduced dimensions (LSI) rather than scRNA-seq PCA space — SCALA handles the modality separately
- The object lacks normalized expression and PCA coordinates (no variable features detected), so no meaningful SNN graph can be built
- The goal is marker identification for pre-existing cluster labels rather than discovering the clusters themselves

## Inputs

- QC-filtered, normalized, scaled scRNA-seq expression matrix (e.g. PBMC3k) with ~2000 variable features
- PCA coordinates per cell (25 informative PCs identified via 10-fold SVD cross-validation)
- Seurat object with variable-feature and dimensionality-reduction slots populated

## Outputs

- Per-cell barcode-to-Louvain-cluster assignment table (cluster_assignments.csv)
- Per-cluster marker-gene tables ranked by Log2FC (top 10 markers per cluster, marker_genes_per_cluster.csv)
- Cluster labels overlaid on UMAP/t-SNE dimensionality-reduction plots

## How to apply

Start from the QC-filtered, normalized, scaled Seurat object holding its variable features (default 2000) and PCA coordinates determined by 10-fold SVD cross-validation (25 informative PCs in the PBMC3k use-case). Construct the shared-nearest-neighbour (SNN) graph on Euclidean distances in PCA space with Seurat's FindNeighbors, then partition the graph with the Louvain algorithm (FindClusters) to obtain highly interconnected communities and a per-cell cluster label. Treat resolution as the main tunable parameter and iterate if communities are over- or under-merged. Downstream, run one-vs-rest differential expression per cluster in Seurat (Wilcoxon rank sum test on normalized expression, with MAST and DESeq2 as alternatives) to characterize each community, exporting cluster_assignments.csv and per-cluster marker tables ranked by Log2FC. This graph-based route is what SCALA automates in its 'Clustering' and 'Markers' Identification' modules.

## Related tools

- **SCALA** (Wraps the SNN-graph construction and Louvain clustering step in an interactive pipeline ending in marker identification)
- **Seurat** (Builds the SNN graph from PCA-space Euclidean distances and runs the Louvain partitioning plus one-vs-rest Wilcoxon/MAST/DESeq2 marker tests) — https://github.com/scottgigante/seurat

## Examples

```
pbmc <- FindNeighbors(pbmc, dims = 1:25); pbmc <- FindClusters(pbmc, algorithm = 1); markers <- FindAllMarkers(pbmc, test.use = "wilcox", only.pos = TRUE)
```

## Evaluation signals

- cluster_assignments.csv exists with non-zero rows and every barcode carries a Louvain cluster label
- marker_genes_per_cluster.csv contains per-cluster entries ranked by Log2FC matching the paper's per-cluster marker depiction (Fig. 2E)
- Number of PCs used to build the graph matches the SVD cross-validation choice (25 PCs for PBMC3k)
- Marker tests ran each cluster's cells against all remaining cells (one-vs-rest contrast)

## Limitations

- In SCALA's online version the supported dataset size is restricted to < 2 GB with no more than 2 CPU threads per session, and some steps cannot be completed for datasets larger than 50,000 cells — use the desktop version
- Only human and mouse datasets are currently supported
- Software version numbers for Seurat employed in SCALA are not stated in the article, so exact reproduction may depend on version differences

## Evidence

- [methods] SNN graph construction and Louvain partitioning workflow: [evidence span withheld — anchor status: not_found]
- [readme] Presto's Wilcoxon rank-sum for fast marker discovery: [evidence span withheld — anchor status: not_found]
