---
name: single-cell-snn-graph-construction
description: Use when a QC-filtered, normalized, scaled scRNA-seq object with variable
  features and PCA coordinates must be converted into a cell-to-cell SNN graph to
  enable graph-based (Louvain) clustering and downstream marker identification.
when_to_use_negative:
- Input is raw count data that has not passed QC, normalization, scaling, and PCA
  — the graph must be built on scaled PCA coordinates, not raw expression
- Input is scATAC-seq data analyzed via LSI dimensionality reduction (SCALA uses LSI
  for the epigenomic modality, not PCA-based SNN on gene expression)
- The goal is only cluster annotation/label transfer (e.g., CIPR) rather than de novo
  graph-based partitioning of cells
edam_operation: http://edamontology.org/operation_3626
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: Seurat
  role: Constructs the SNN graph from Euclidean distances in PCA space and runs the
    Louvain partitioning that produces cluster labels
- name: SCALA
  role: Pipeline embedding this graph-construction and clustering step in its scRNA-seq
    analysis mode and exporting cluster assignments and markers
  repo: https://github.com/PavlopoulosLab/SCALA
- name: R
  role: Implementation language of SCALA and Seurat in which the SNN graph construction
    is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:66a6acac41c7700498889b7a53ceb9eaeb28688ec658bd91f17ec534496b8f1e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/PavlopoulosLab/SCALA
  tools:
  - R
  - SCALA
  - Seurat
  edam_operation: http://edamontology.org/operation_3626
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-snn-graph-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a shared-nearest-neighbour (SNN) graph over single-cell transcriptomes in the informative PCA space (e.g., 25 PCs of PBMC3k) using Seurat, as SCALA does, so that graph-based Louvain partitioning can identify distinct cell populations.

## When to use

Use when a QC-filtered, normalized, scaled scRNA-seq object with variable features and PCA coordinates must be converted into a cell-to-cell SNN graph to enable graph-based (Louvain) clustering and downstream marker identification.

## When NOT to use

- Input is raw count data that has not passed QC, normalization, scaling, and PCA — the graph must be built on scaled PCA coordinates, not raw expression
- Input is scATAC-seq data analyzed via LSI dimensionality reduction (SCALA uses LSI for the epigenomic modality, not PCA-based SNN on gene expression)
- The goal is only cluster annotation/label transfer (e.g., CIPR) rather than de novo graph-based partitioning of cells

## Inputs

- QC-filtered, normalized, and scaled Seurat scRNA-seq object (e.g., PBMC3k)
- Variable feature set (default 2000 variable genes)
- PCA coordinates / informative PC selection (e.g., 25 PCs determined by 10-fold SVD cross-validation)

## Outputs

- SNN graph over cells (k-nearest-neighbour graph in PCA space)
- Per-cell Louvain cluster assignments (exported as cluster_assignments.csv)
- Per-cluster marker table ranked by Log2FC, top 10 markers per cluster (marker_genes_per_cluster.csv, as in Fig. 2E)

## How to apply

Start from the QC-filtered, normalized, scaled Seurat object carrying its variable features (default 2000) and PCA coordinates determined by 10-fold SVD cross-validation (SCALA selected 25 PCs). Compute Euclidean distances between cells in this PCA space and embed the cells in a shared-nearest-neighbour (SNN) graph with Seurat's neighbour-detection step, a k-nearest-neighbour approach followed by shared-neighbour linkage construction. Partition the SNN graph into highly interconnected communities with the Louvain algorithm to obtain a per-cell cluster label. Rationale: SNN graphs in reduced PCA space capture transcriptomic similarity robustly and give the discrete partition that marker testing (one cluster vs. the rest) requires downstream.

## Related tools

- **Seurat** (Constructs the SNN graph from Euclidean distances in PCA space and runs the Louvain partitioning that produces cluster labels)
- **SCALA** (Pipeline embedding this graph-construction and clustering step in its scRNA-seq analysis mode and exporting cluster assignments and markers) — https://github.com/PavlopoulosLab/SCALA
- **R** (Implementation language of SCALA and Seurat in which the SNN graph construction is executed)

## Examples

```
FindNeighbors(pbmc, reduction = "pca", dims = 1:25); pbmc <- FindClusters(pbmc, algorithm = 1)
```

## Evaluation signals

- cluster_assignments.csv exists with non-zero rows and every barcode carries a Louvain cluster label
- marker_genes_per_cluster.csv exists with non-zero rows and contains per-cluster entries ranked by Log2FC (top 10 markers per cluster) matching the paper's Fig. 2E depiction
- The number of informative PCs fed to the graph (25 in the SCALA PBMC3k use-case) matches the SVD cross-validation result
- Marker calls from the Wilcoxon rank sum test on each cluster vs. all remaining cells are consistent with, or explicitly compared against, MAST and DESeq2 alternatives

## Limitations

- The number of Louvain clusters and their resolution can differ across Seurat software versions, as SCALA's reanalysis noted version differences versus the original publication
- The online SCALA version restricts dataset size to < 2 GB with 2 CPU threads per session, and some steps cannot be completed for datasets larger than 50,000 cells — the desktop version is recommended for larger datasets

## Evidence

- [methods] Embed the cells in a shared-nearest-neighbour (SNN) graph based on Euclidean distances in the PCA space using Seurat.: [evidence span withheld — anchor status: not_found]
- [methods] Partition the SNN graph into highly interconnected communities with the Louvain algorithm to obtain cluster labels per cell.: [evidence span withheld — anchor status: not_found]
- [methods] SCALA identifies cluster marker genes by comparing each cluster's transcriptomes against the rest of the cells' transcriptomes through the Wilcoxon rank sum test on the normalized expression values.: [evidence span withheld — anchor status: not_found]
