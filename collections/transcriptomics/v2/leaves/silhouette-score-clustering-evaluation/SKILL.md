---
name: silhouette-score-clustering-evaluation
description: Use when you need to quantitatively compare clustering performance of
  DR methods (e.g., scLENS vs Seurat/Scanpy PCs) on single-cell datasets with known
  ground-truth labels across varying sparsity or TGC variability.
when_to_use_negative:
- When ground-truth cell labels are unavailable — SIL can be computed but ECS-vs-truth
  and the label-optimized resolution sweep cannot be performed as specified
- When the goal is selecting a signal-dimension threshold rather than comparing clustering
  outcomes — scLENS's RMT-based noise filtering and signal robustness test serve that
  purpose instead
- When data have not passed QC filtering (cells <200 expressed genes, genes expressed
  in <15 cells, cells with >5% mitochondrial reads) — scores would be dominated by
  low-quality cells
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scLENS
  role: Produces the low-dimensional embedding whose clustering is scored by SIL and
    ECS in the benchmark
  repo: github.com/Mathbiomed/scLENS
- name: Seurat
  role: Baseline DR method (50 PCs) whose embedding is scored with SIL/ECS for comparison
- name: Scanpy
  role: Baseline DR method (50 PCs) whose embedding is scored with SIL/ECS for comparison
- name: scVI
  role: Baseline DR method (10,000 highly variable genes) whose embedding is scored
    with SIL/ECS
- name: UMAP
  role: Provides the 2D embedding (15 neighbors, min_dist 0.01, cosine metric) on
    which hierarchical clustering and SIL computation are performed
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:a84b1f6cdb1eb394840ae1a9169c90055bc919a602e46b5d1664895964f94b82
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ACTIONet
  - Monocle3
  - ParallelPCA
  - Randomly
  - SHARP
  - Scanpy
  - Seurat
  - UMAP
  - ZINB-WaVE
  - scDHA
  - scLENS
  - scTransform
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 24
---

# Silhouette-score clustering evaluation

## Summary

Evaluates clustering quality of dimensionality-reduction embeddings from scRNA-seq data by computing the average silhouette (SIL) score and element-centric similarity (ECS) against ground-truth cell-type labels, as used to benchmark scLENS against 11 DR packages.

## When to use

Use when you need to quantitatively compare clustering performance of DR methods (e.g., scLENS vs Seurat/Scanpy PCs) on single-cell datasets with known ground-truth labels across varying sparsity or TGC variability.

## When NOT to use

- When ground-truth cell labels are unavailable — SIL can be computed but ECS-vs-truth and the label-optimized resolution sweep cannot be performed as specified
- When the goal is selecting a signal-dimension threshold rather than comparing clustering outcomes — scLENS's RMT-based noise filtering and signal robustness test serve that purpose instead
- When data have not passed QC filtering (cells <200 expressed genes, genes expressed in <15 cells, cells with >5% mitochondrial reads) — scores would be dominated by low-quality cells

## Inputs

- Low-dimensional embedding (PCA or UMAP coordinates) from a DR method such as scLENS, Seurat, Scanpy, Monocle3, ParallelPCA, ACTIONet, SHARP, scDHA, scVI, ZINB-WaVE, or scTransform
- Vector of cluster assignments (from hierarchical UMAP clustering or Leiden SNN-graph clustering)
- Ground-truth cell-type labels (e.g., ten T-cell subtypes from scDesign2-simulated data, or independently assigned labels from Koh/Kumar/Trapnell datasets)

## Outputs

- Average silhouette (SIL) score per package-dataset pair
- Element-centric similarity (ECS, damping factor 0.975) score per package-dataset pair
- Benchmark table of SIL/ECS scores across sparsity levels and CVs of TGC (e.g., Fig. 4a,c style)
- Relative performance metric (scLENS ECS minus best competitor ECS) per dataset

## How to apply

Run each DR method (scLENS, Seurat/Scanpy/Monocle3 with 50 PCs, scVI with 10,000 highly variable genes, etc.) on the QC-filtered expression matrix, then derive two cluster assignments per method: hierarchical clustering on the 2D UMAP embedding and Leiden clustering on an SNN graph (20 neighbors, edge cutoff 1/15, cosine similarity), sweeping cut-tree heights or resolutions to maximize ECS against ground truth. Compute the average silhouette (SIL) score on the embedding with the resulting labels, and compute element-centric similarity (ECS, damping factor 0.975) between predicted and ground-truth partitions. Average SIL/ECS per package across datasets (e.g., the 13 simulated Zheng-mixture T-cell datasets and the 39-dataset extended benchmark) to build the comparison table. A correct application reproduces the pattern where scLENS attains the highest SIL/ECS with the smallest decline as sparsity increases.

## Related tools

- **scLENS** (Produces the low-dimensional embedding whose clustering is scored by SIL and ECS in the benchmark) — github.com/Mathbiomed/scLENS
- **Seurat** (Baseline DR method (50 PCs) whose embedding is scored with SIL/ECS for comparison)
- **Scanpy** (Baseline DR method (50 PCs) whose embedding is scored with SIL/ECS for comparison)
- **scVI** (Baseline DR method (10,000 highly variable genes) whose embedding is scored with SIL/ECS)
- **UMAP** (Provides the 2D embedding (15 neighbors, min_dist 0.01, cosine metric) on which hierarchical clustering and SIL computation are performed)

## Examples

```
sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev); scLENS.apply_umap!(sclens_embedding)  # then compute SIL/ECS on the resulting UMAP coordinates vs ground-truth labels
```

## Evaluation signals

- scLENS attains the highest average SIL score among all 12 packages on the sparsity/TGC-variance benchmark (Fig. 4a pattern)
- scLENS ranks first in ECS for both hierarchical (UMAP) and Leiden graph-based clustering across the 39-dataset extended benchmark (Fig. 5a pattern)
- Performance degradation of scLENS with increasing sparsity (up to ~0.84) is smaller than for fixed-50-PC packages
- Sweeping Leiden resolutions / cut-tree heights converges to assignments maximizing ECS against ground truth (damping factor 0.975)
- Delta-SIL = SIL − SILp (shuffled non-zeros, scLENS re-run) quantifies non-binary information retained per dataset

## Limitations

- Optimal Leiden resolution or hierarchical cut height is chosen by maximizing ECS against ground truth, which requires known labels and is not applicable to unlabeled data
- RMT-based steps inside scLENS demand significant memory — computing all eigenvalues of the large cell-to-cell similarity matrix requires high-specification hardware for large datasets with whole genes
- SIL/ECS on 2D UMAP embeddings can under-represent local structure for methods relying on highly variable gene selection, which may compromise low-variance genes' local structure

## Evidence

- [abstract] SIL benchmark finding: "Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets with high sparsity and variability"
- [methods] ECS and SIL computation in benchmark workflow: [evidence span withheld — anchor status: not_found]
- [methods] Clustering variants before scoring: [evidence span withheld — anchor status: not_found]
- [readme] QC filtering prior to scoring: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality"
