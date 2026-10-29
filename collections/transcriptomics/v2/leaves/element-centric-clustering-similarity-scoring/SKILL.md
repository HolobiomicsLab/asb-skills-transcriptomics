---
name: element-centric-clustering-similarity-scoring
description: Use when benchmarking DR embeddings (e.g., scLENS vs Seurat/Scanpy/scVI
  PCs) against known cell-type labels and you need per-element clustering agreement
  (ECS) plus silhouette scores to rank methods across sparsity levels.
when_to_use_negative:
- The dataset has no independent ground-truth labels (ECS and SIL here are computed
  against reference assignments, so unsupervised-only data cannot be scored this way)
- The input is still a raw count matrix — QC filtering (cells with <200 expressed
  genes, genes expressed in <15 cells, cells with >5% mitochondrial reads) and dimensionality
  reduction must precede ECS scoring
- The goal is biological interpretation of a single clustering rather than comparative
  benchmarking of DR methods
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scLENS
  role: Produces the DR embedding whose clusterings (hierarchical on UMAP, Leiden
    on SNN) are scored with ECS/SIL; benchmarked as the top-performing method
  repo: github.com/Mathbiomed/scLENS
- name: UMAP
  role: UMAP.jl generates the 2D embeddings (15 neighbors, min_dist 0.01, cosine metric)
    on which hierarchical clustering is performed before ECS scoring
- name: scDesign2
  role: Simulates the T-cell and Tabula Muris datasets (version 1.0.0) whose ground-truth
    subtype labels serve as the ECS/SIL reference
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:b597e1f9cc38599cb966acbc3b88bdc77749dbe260907b548eb2ac67f6d43843
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
  - scDesign2
  - scLENS
  - scTransform
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9130434782608695
n_claims: 23
---

# element-centric-clustering-similarity-scoring

## Summary

Scores dimensionality-reduction clusterings by element-centric similarity (ECS, damping factor 0.975) and average silhouette (SIL) against ground-truth cell labels, as used in the scLENS benchmark of 12 DR packages across 13 simulated T-cell and 16 real datasets.

## When to use

Use when benchmarking DR embeddings (e.g., scLENS vs Seurat/Scanpy/scVI PCs) against known cell-type labels and you need per-element clustering agreement (ECS) plus silhouette scores to rank methods across sparsity levels.

## When NOT to use

- The dataset has no independent ground-truth labels (ECS and SIL here are computed against reference assignments, so unsupervised-only data cannot be scored this way)
- The input is still a raw count matrix — QC filtering (cells with <200 expressed genes, genes expressed in <15 cells, cells with >5% mitochondrial reads) and dimensionality reduction must precede ECS scoring
- The goal is biological interpretation of a single clustering rather than comparative benchmarking of DR methods

## Inputs

- Dimensionality-reduction embeddings (scLENS output, or 50 PCs from Seurat/Scanpy/Monocle3, ParallelPCA, Randomly, ACTIONet, SHARP, scDHA, scVI, ZINB-WaVE, scTransform outputs)
- 2D UMAP embeddings of each DR output (UMAP.jl, 15 neighbors, min_dist 0.01, cosine metric)
- Ground-truth cell-type labels for each dataset (e.g., ten simulated T-cell subtypes from scDesign2, Zheng-mixture, Koh, Kumar, Trapnell labels)

## Outputs

- Per-dataset, per-package ECS scores (damping factor 0.975) for hierarchical-UMAP and Leiden-SNN clusterings
- Average silhouette (SIL) scores against ground-truth labels
- Delta-SIL (SIL - SILp after shuffling non-zero values) quantifying non-binary information
- Benchmark score tables and relative-performance rankings (Fig. 4a,c; Fig. 5a,b)

## How to apply

For each package's embedding, derive two cluster assignments: (1) hierarchical clustering on the 2D UMAP embedding (UMAP.jl, 15 neighbors, min_dist 0.01, cosine metric) and (2) Leiden clustering on an SNN graph (20 neighbors, edge cutoff 1/15, cosine similarity). Sweep cut-tree heights and Leiden resolutions, keeping the assignment that maximizes ECS against the ground-truth labels. Then compute the element-centric similarity with damping factor d=0.975 and the average silhouette (SIL) score for every dataset-package pair, compiling score tables (Fig. 4a,c; Fig. 5a). Also compute Delta-SIL = SIL - SILp by shuffling non-zero values and re-running scLENS to quantify non-binary information per dataset. Rationale: SIL/ECS jointly capture how well the embedding preserves cluster separation and element-wise label agreement, letting sparsity-driven degradation (e.g., fixed-50-PC packages) be quantified.

## Related tools

- **scLENS** (Produces the DR embedding whose clusterings (hierarchical on UMAP, Leiden on SNN) are scored with ECS/SIL; benchmarked as the top-performing method) — github.com/Mathbiomed/scLENS
- **UMAP** (UMAP.jl generates the 2D embeddings (15 neighbors, min_dist 0.01, cosine metric) on which hierarchical clustering is performed before ECS scoring)
- **scDesign2** (Simulates the T-cell and Tabula Muris datasets (version 1.0.0) whose ground-truth subtype labels serve as the ECS/SIL reference)

## Evaluation signals

- ECS values are reported with damping factor 0.975 explicitly recorded, and every dataset-package pair has both an ECS and an SIL score
- Cluster assignments were selected by sweeping cut-tree heights / Leiden resolutions and keeping the ECS-maximizing assignment, so reported ECS is the best achievable per method-dataset pair
- Reproduced tables should show scLENS attaining the highest SIL and ECS among the 12 packages with the smallest performance decline as sparsity increases
- Fixed-50-PC packages (Seurat, Scanpy, Monocle3) should degrade substantially relative to scLENS as sparsity rises, and relative performance of scLENS should increase with increasing sparsity (Fig. 5b)

## Limitations

- Because cut heights and resolutions are selected to maximize ECS against ground truth, scores represent an optimistic upper bound that requires independent labels
- ECS/SIL scores depend on upstream embedding quality and clustering hyperparameters fixed by the paper (20 neighbors, edge cutoff 1/15, cosine similarity), so results may not transfer to other settings
- scLENS itself (whose embeddings are being scored) requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources on large whole-gene datasets

## Evidence

- [other] Compute the average silhouette (SIL) score and the element-centric similarity (ECS, damping factor 0.975) against ground-truth labels for every dataset-package pair: [evidence span withheld — anchor status: not_found]
- [other] hierarchical clustering on the 2D UMAP embedding (15 neighbors, min distance 0.01, cosine metric) and Leiden graph-based clustering on an SNN graph (20 neighbors, edge cutoff 1/15, cosine similarity): [evidence span withheld — anchor status: not_found]
- [abstract] Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets with high sparsity and variability: "Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets with high sparsity and variability"
