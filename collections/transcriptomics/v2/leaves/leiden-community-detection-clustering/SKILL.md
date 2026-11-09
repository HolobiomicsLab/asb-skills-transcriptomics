---
name: leiden-community-detection-clustering
description: Use when cells have already been embedded via a kernel-induced affinity
  graph (e.g., the symmetrized q-diffused kernel matrix) and you need unsupervised
  cluster assignments scored against ground-truth annotations or stimulation labels.
when_to_use_negative:
- When only a dimensionality-reduced embedding is available and no cell-by-cell affinity/adjacency
  graph has been computed — Leiden here is applied to the symmetrized kernel graph,
  not raw expression.
- When the task is batch-effect-corrected integration of disparate datasets; the article
  explicitly did not consider batch-effect correction upstream of this clustering.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: q-Diffusion.jl
  role: builds the q-diffused kernel (AdaptiveKernel over DeformedKernel, q=1.2, k=16,
    in_log=true) whose symmetrized affinity matrix is the graph Leiden clusters
  repo: github.com/marmarelis/QDiffusion.jl
- name: q-diffusion
  role: the q-diffused kernel framework whose graph Leiden community detection partitions,
    benchmarked against Seurat, Monocle, Scanpy, SC3, and scVI
  repo: github.com/marmarelis/QDiffusion.jl
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:4d6512b4c49529498ec6e30f488fb6f3052bb2be95436d02ef6db12df5818834
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leiden
  - Monocle
  - PHATE
  - SC3
  - Scanpy
  - Seurat
  - q-Diffusion.jl
  - q-diffusion
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# leiden-community-detection-clustering

## Summary

Cluster cells by running Leiden community detection (optimizing modularity) on the symmetrized weighted undirected graph induced by the q-diffused kernel affinity matrix, then quantify cluster quality with adjusted Rand indices (ARI) against reference labels.

## When to use

Use when cells have already been embedded via a kernel-induced affinity graph (e.g., the symmetrized q-diffused kernel matrix) and you need unsupervised cluster assignments scored against ground-truth annotations or stimulation labels.

## When NOT to use

- When only a dimensionality-reduced embedding is available and no cell-by-cell affinity/adjacency graph has been computed — Leiden here is applied to the symmetrized kernel graph, not raw expression.
- When the task is batch-effect-corrected integration of disparate datasets; the article explicitly did not consider batch-effect correction upstream of this clustering.

## Inputs

- log-normalized (genes x cells) scRNAseq matrix produced by preprocess_cell_counts in QDiffusion.jl
- symmetrized cell-by-cell q-diffused affinity matrix A=(Ã+Ãᵀ)/2 (diagonal subtracted)
- reference cell labels (e.g., IFN-γ stimulation vs control, or Tabula Sapiens manual phenotype annotations)

## Outputs

- Leiden cluster assignment per cell
- adjusted Rand index (ARI) table against reference labels (e.g., cond_qd_pbmc_ari.csv, tabula_sapiens_ari_comparison.csv)

## How to apply

Evaluate the q-diffused kernel (DeformedKernel with q=1.2 wrapped in an AdaptiveKernel scaled to the k=16 nearest neighbor, inner bandwidth from estimate_inner_bandwidth at the 50% quantile of per-gene average pairwise squared distances) between all pairs of cells, subtract the diagonal, and symmetrize arithmetically as A=(Ã+Ãᵀ)/2 to produce a weighted undirected graph. Run Leiden community detection optimizing modularity on this graph to assign cells to clusters. When reference phenotypes exist, compute adjusted Rand indices (ARIs) of the Leiden assignments against those labels — e.g., within each of the eight PBMC subtypes for the IFN-γ stimulation benchmark, or against the expert manual annotations of Tabula Sapiens organ tissues (skin, trachea, uterus, liver; <10,000 cells, PBMCs screened out). For local distributional segmentation of MERFISH cortical tissue, Leiden is one of two accepted algorithms (the other being hierarchical clustering with Ward's linkage), with clustering parameters selected by a grid search over silhouette scores.

## Related tools

- **q-Diffusion.jl** (builds the q-diffused kernel (AdaptiveKernel over DeformedKernel, q=1.2, k=16, in_log=true) whose symmetrized affinity matrix is the graph Leiden clusters) — github.com/marmarelis/QDiffusion.jl
- **q-diffusion** (the q-diffused kernel framework whose graph Leiden community detection partitions, benchmarked against Seurat, Monocle, Scanpy, SC3, and scVI) — github.com/marmarelis/QDiffusion.jl

## Examples

```
kernel = AdaptiveKernel(16, DeformedKernel(1.2f0, data, inner_bandwidth=estimate_inner_bandwidth(data)), data, cuda_devices=[0], in_log=true); A = (evaluate(kernel, data) .+ transpose(evaluate(kernel, data)))./2; clusters = leiden(A, optimize_modularity=true)
```

## Evaluation signals

- The affinity matrix must be symmetrized (A=(Ã+Ãᵀ)/2) with the diagonal subtracted before Leiden runs — running Leiden on an asymmetric or self-weighted matrix is incorrect.
- ARI against the known labels should reproduce the paper's direction: higher for q-diffusion than Seurat/Monocle/Scanpy/SC3 baselines on most PBMC subtypes (with documented exceptions for CD8 T cells and some NK comparisons).
- On Tabula Sapiens, ARI should exceed the atlas-supplied embeddings (including scVI) for skin, trachea, and uterus while liver may degrade — liver ARI below baselines is an expected edge case, not necessarily a bug.
- Clustering parameters should be chosen at the maximum silhouette score from the grid search for each case (e.g., per organ).

## Limitations

- No algorithm is provided for selecting the q-diffusion parameters q and φ; there is no selection algorithm for the non-Euclidean deformation q and inner bandwidth φ, the two vital parameters.
- Exact all-pairs q-diffused kernel evaluation is slow (20 minutes to 3 hours per n-by-n affinity matrix).
- No batch-effect correction or alignment of disparate datasets was considered before clustering.

## Evidence

- [results] Evaluate q-diffused kernel between all pairs of cells and symmetrify the adjacency matrix into a weighted undirected graph: "the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells"
- [results] Leiden clustering with ARI evaluation: "Adjusted Rand indices (ARIs)31 of Leiden community detection42 in the PBMC dataset."
- [results] Parameter selection by silhouette grid search: "Clustering parameters were optimized by a grid search over the silhouette scores44 for each case."
- [results] Leiden as segmentation algorithm for LDS: "the segmentation was performed by two popular algorithms42: Leiden community"
- [methods] Community Detection methods section: "## Methods > Community Detection"
