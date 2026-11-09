---
name: tabula-sapiens-cell-type-annotation-matching
description: Use when evaluating whether an embedding or kernel method improves unsupervised
  cell-type recovery on Tabula Sapiens organ tissues (<10,000 cells, PBMCs screened
  out), with ARI against expert manual annotations.
when_to_use_negative:
- The tissue sample exceeds 10,000 cells — the paper restricted the benchmark to organs
  with fewer than 10,000 sampled cells
- The input is PBMC data — this benchmark explicitly screens out PBMCs (the PBMC IFN-γ
  benchmark is a separate analysis)
- Batch-effect correction or alignment of disparate datasets is required — the authors
  state 'we did not consider batch-effect correction or alignment of disparate datasets'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: QDiffusion.jl
  role: Julia package providing preprocess_cell_counts, estimate_inner_bandwidth,
    DeformedKernel/AdaptiveKernel, and GPU-accelerated kernel evaluation used to build
    the q-diffused affinity graph
  repo: github.com/marmarelis/QDiffusion.jl
- name: q-diffusion
  role: The q-diffused kernel method whose full-transcriptome affinities are clustered
    and benchmarked
  repo: github.com/marmarelis/QDiffusion.jl
- name: Leiden
  role: Community detection optimizing modularity on the symmetrized q-diffused kernel
    graph to produce clusters
- name: scVI
  role: Atlas-supplied dimensionality reduction used as the state-of-the-art comparison
    embedding
- name: Seurat
  role: Baseline neighborhood-graph clustering method for comparison
- name: Monocle
  role: Baseline neighborhood-graph clustering method for comparison
- name: Scanpy
  role: Baseline neighborhood-graph clustering method for comparison
- name: SC3
  role: Baseline clustering method whose computed clusters are included for comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:f65ecc92fdecdb82cbe22a3f9b1db58ab82d130dae2c64316409ecb2e0871d10
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leiden
  - Monocle
  - QDiffusion.jl
  - SC3
  - Scanpy
  - Seurat
  - q-Diffusion.jl
  - q-diffusion
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# tabula-sapiens-cell-type-annotation-matching

## Summary

Reproduces the Tabula Sapiens unsupervised clustering benchmark from the q-diffusion paper: cluster non-PBMC small-tissue cells from four human organs via Leiden on a q-diffused affinity graph, then score cluster-to-annotation agreement against the atlas's expert manual cell phenotypes using adjusted Rand index.

## When to use

Use when evaluating whether an embedding or kernel method improves unsupervised cell-type recovery on Tabula Sapiens organ tissues (<10,000 cells, PBMCs screened out), with ARI against expert manual annotations.

## When NOT to use

- The tissue sample exceeds 10,000 cells — the paper restricted the benchmark to organs with fewer than 10,000 sampled cells
- The input is PBMC data — this benchmark explicitly screens out PBMCs (the PBMC IFN-γ benchmark is a separate analysis)
- Batch-effect correction or alignment of disparate datasets is required — the authors state 'we did not consider batch-effect correction or alignment of disparate datasets'

## Inputs

- Tabula Sapiens organ-tissue gene-by-cell count matrices (skin, trachea, uterus, liver; <10,000 cells each, PBMCs screened out)
- Manually annotated cell phenotype labels supplied by the Tabula Sapiens Consortium
- Atlas-supplied dimensionality reductions (e.g., scVI embeddings) for baseline comparison

## Outputs

- Leiden cluster assignments per organ on the q-diffused affinity graph
- Per-organ ARI table comparing q-diffusion vs. scVI and baseline methods, written to tabula_sapiens_ari_comparison.csv

## How to apply

Log-normalize the gene-by-cell matrices with preprocess_cell_counts in QDiffusion.jl (no feature selection or dimensionality reduction), select organ tissues with fewer than 10,000 sampled cells, and screen out the PBMCs. Estimate the inner bandwidth via estimate_inner_bandwidth, wrap a DeformedKernel (deformation parameter q = 1.2) in an AdaptiveKernel, evaluate the q-diffused kernel between all pairs of cells, and symmetrize the anisotropic affinity matrix arithmetically, A = (Ã + Ãᵀ)/2, into a weighted undirected graph. Run Leiden community detection optimizing modularity, with clustering parameters optimized per organ by a grid search over silhouette scores. Compute adjusted Rand indices (ARI) of the clusters against the manually annotated cell phenotypes per organ, and repeat with the atlas-supplied dimensionality reductions (including scVI) and baseline neighborhood-graph methods (Seurat, Monocle, Scanpy, SC3) for comparison.

## Related tools

- **QDiffusion.jl** (Julia package providing preprocess_cell_counts, estimate_inner_bandwidth, DeformedKernel/AdaptiveKernel, and GPU-accelerated kernel evaluation used to build the q-diffused affinity graph) — github.com/marmarelis/QDiffusion.jl
- **q-diffusion** (The q-diffused kernel method whose full-transcriptome affinities are clustered and benchmarked) — github.com/marmarelis/QDiffusion.jl
- **Leiden** (Community detection optimizing modularity on the symmetrized q-diffused kernel graph to produce clusters)
- **scVI** (Atlas-supplied dimensionality reduction used as the state-of-the-art comparison embedding)
- **Seurat** (Baseline neighborhood-graph clustering method for comparison)
- **Monocle** (Baseline neighborhood-graph clustering method for comparison)
- **Scanpy** (Baseline neighborhood-graph clustering method for comparison)
- **SC3** (Baseline clustering method whose computed clusters are included for comparison)

## Examples

```
kernel = AdaptiveKernel(16, DeformedKernel(1.2f0, data, inner_bandwidth=estimate_inner_bandwidth(data)), data, in_log=true)
```

## Evaluation signals

- ARI against expert manual annotations should exceed the atlas-supplied scVI embeddings in skin, trachea, and uterus; the liver is expected to degrade
- Symmetrized affinity matrix A = (Ã + Ãᵀ)/2 must be symmetric and non-negative before Leiden runs
- Per-organ clustering parameters should be chosen by grid search maximizing silhouette score, not fixed defaults
- The baseline ARI table must include Seurat, Monocle, Scanpy, SC3, and scVI for a complete comparison
- Output CSV (tabula_sapiens_ari_comparison.csv) must contain one row per organ per method with its ARI

## Limitations

- Q-diffusion showed degraded liver clusters in this benchmark, so organ-level failure is a known mode
- Runtime of the q-diffused kernel evaluation is suboptimal (20 min to 3 h for one affinity matrix)
- No algorithm for selecting the q-diffusion parameters q and φ is provided; the paper notes a careful selection algorithm is still needed
- No batch-effect correction or dataset alignment was considered in this benchmark
- The AdaptiveKernel scales bandwidth relative to k-nn distances (k=16 used in related analyses) and the kernel should be evaluated in_log=true for numerical stability

## Evidence

- [results] Selecting Tabula Sapiens organ tissues with fewer than 10,000 cells and screening out PBMCs: "we selected those organ tissues for which fewer than 10,000 cells were sampled. We also screened out the PBMCs"
- [results] Symmetrizing the q-diffused kernel adjacency into a weighted undirected graph: "the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells"
- [results] Grid search over silhouette scores to optimize clustering parameters per case: "Clustering parameters were optimized by a grid search over the silhouette scores44 for each case."
- [readme] preprocess_cell_counts convenience method in QDiffusion.jl: "There is also a convenience mathod called `preprocess_cell_counts(data)` that returns a named tuple containing different versions of the matrix"
- [readme] estimate_inner_bandwidth heuristic from README: "inner_bandwidth = estimate_inner_bandwidth(data)"
- [discussion] Tabula Sapiens benchmarked q-diffusion against atlas-supplied reductions: "The Tabula Sapiens benchmark evaluated q-diffusion against the dimensionality reductions supplied by the authors of the atlas"
