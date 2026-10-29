---
name: scrtcna-dimensionality-reduction-baseline-comparison
description: Use when a single-cell transcriptomics dataset ships with pre-computed
  dimensionality reductions and manual cell-type annotations, and you must show whether
  a new embedding or clustering method beats those baselines on unsupervised cell-type
  recovery.
when_to_use_negative:
- Datasets requiring batch-effect correction or alignment of disparate batches — the
  source study explicitly did not consider batch-effect correction, so baseline comparisons
  under strong batch structure are out of scope
- Samples with tens of thousands or more cells — the benchmark deliberately restricted
  to tissues with fewer than 10,000 sampled cells, and q-diffused kernel evaluation
  across all cell pairs takes 20 min to 3 h per affinity matrix
- Analyses that need directly interpretable nonlinear structure — the authors note
  q-diffusion's nonlinearities are less directly interpretable than those of dimensionality-reduction
  embeddings
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: QDiffusion.jl
  role: Implements preprocess_cell_counts, estimate_inner_bandwidth, AdaptiveKernel/DeformedKernel
    construction, and the q-diffused kernel evaluated between all pairs of cells
  repo: github.com/marmarelis/QDiffusion.jl
- name: Leiden
  role: Community detection on the symmetrized weighted graph; its clusters are scored
    by ARI against manual annotations for both q-diffusion and baselines
- name: scVI
  role: State-of-the-art dimensionality reduction supplied with the Tabula Sapiens
    datasets, used as the primary supplied-embedding baseline
- name: Seurat
  role: Neighborhood-graph clustering baseline for comparison
- name: Monocle
  role: Neighborhood-graph clustering baseline for comparison
- name: Scanpy
  role: Neighborhood-graph clustering baseline for comparison
- name: SC3
  role: Clustering baseline whose precomputed clusters are included for comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:61fa63311e28643d0456363f517113853627343532401555c1875709140806d2
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# scrtcna-dimensionality-reduction-baseline-comparison

## Summary

Benchmark protocol for comparing an experimental single-cell embedding/clustering method (here, q-diffusion with Leiden) against the dimensionality reductions supplied with a dataset (e.g., scVI in Tabula Sapiens) and neighborhood-graph baselines (Seurat, Monocle, Scanpy, SC3), scored by adjusted Rand index (ARI) against manual cell-type annotations.

## When to use

Use when a single-cell transcriptomics dataset ships with pre-computed dimensionality reductions and manual cell-type annotations, and you must show whether a new embedding or clustering method beats those baselines on unsupervised cell-type recovery.

## When NOT to use

- Datasets requiring batch-effect correction or alignment of disparate batches — the source study explicitly did not consider batch-effect correction, so baseline comparisons under strong batch structure are out of scope
- Samples with tens of thousands or more cells — the benchmark deliberately restricted to tissues with fewer than 10,000 sampled cells, and q-diffused kernel evaluation across all cell pairs takes 20 min to 3 h per affinity matrix
- Analyses that need directly interpretable nonlinear structure — the authors note q-diffusion's nonlinearities are less directly interpretable than those of dimensionality-reduction embeddings

## Inputs

- Gene-by-cell count matrices from Tabula Sapiens organ tissues with fewer than 10,000 sampled cells, PBMCs screened out
- Manually annotated cell phenotype labels per tissue (expert annotations from the Tabula Sapiens Consortium)
- Atlas-supplied dimensionality reductions for the same tissues, including scVI embeddings

## Outputs

- Per-organ ARI comparison table (e.g., tabula_sapiens_ari_comparison.csv) covering q-diffusion vs. scVI, Seurat, Monocle, Scanpy, and SC3
- Leiden cluster assignments on the symmetrized q-diffused kernel graph per organ
- Silhouette-score grid-search optima used to fix clustering parameters per case

## How to apply

Select organ tissues with fewer than 10,000 sampled cells and screen out PBMCs, then log-normalize the (genes x cells) matrix without feature selection or dimensionality reduction (QDiffusion.jl's `preprocess_cell_counts(data)` and its `.normalized` element implement this). Estimate the inner bandwidth with `estimate_inner_bandwidth(data)`, build an `AdaptiveKernel` wrapping a `DeformedKernel` (deformation parameter q > 1, e.g. q = 1.2–1.5; use `in_log=true` for numerical stability), evaluate the q-diffused kernel between all cell pairs, and symmetrize the adjacency matrix into a weighted undirected graph. Run Leiden community detection on that graph, optimizing clustering parameters per organ by a grid search over silhouette scores. Compute ARIs of the resulting clusters against the manually annotated cell phenotypes, then repeat the identical clustering-and-ARI evaluation using the atlas-supplied embeddings (including scVI) and the neighborhood-graph methods Seurat, Monocle, Scanpy, and SC3 so all methods are compared on the same ARI criterion.

## Related tools

- **QDiffusion.jl** (Implements preprocess_cell_counts, estimate_inner_bandwidth, AdaptiveKernel/DeformedKernel construction, and the q-diffused kernel evaluated between all pairs of cells) — github.com/marmarelis/QDiffusion.jl
- **Leiden** (Community detection on the symmetrized weighted graph; its clusters are scored by ARI against manual annotations for both q-diffusion and baselines)
- **scVI** (State-of-the-art dimensionality reduction supplied with the Tabula Sapiens datasets, used as the primary supplied-embedding baseline)
- **Seurat** (Neighborhood-graph clustering baseline for comparison)
- **Monocle** (Neighborhood-graph clustering baseline for comparison)
- **Scanpy** (Neighborhood-graph clustering baseline for comparison)
- **SC3** (Clustering baseline whose precomputed clusters are included for comparison)

## Examples

```
inner_bandwidth = estimate_inner_bandwidth(data); kernel = AdaptiveKernel(16, DeformedKernel(1.5f0, data, inner_bandwidth=inner_bandwidth), data, in_log=true)
```

## Evaluation signals

- ARI of q-diffusion's Leiden clusters vs. manual annotations should exceed the atlas-supplied embeddings (incl. scVI) in at least three of the four organs (skin, trachea, uterus pass; liver may degrade)
- Symmetrization invariant: the affinity matrix equals its transpose, A = (Ã + Ãᵀ)/2, forming a valid weighted undirected graph
- Each organ's clustering parameters were chosen by grid search over silhouette scores, and the search grid/results are recorded per case
- Preprocessing shows no feature selection or dimensionality reduction before kernel construction (log-normalized `.normalized` matrix only)
- Identical ARI evaluation applied to every baseline method so the comparison is like-for-like

## Limitations

- The liver showed degraded clusters from q-diffusion relative to the atlas-supplied embeddings, so gains are not uniform across organs
- No algorithm is provided for selecting the deformation parameter q and inner bandwidth φ; both were fixed heuristically (README suggests q = 1.5, k-nn scale 16)
- Runtime of the q-diffused kernel evaluation across all pairs of cells is suboptimal, taking 20 minutes to 3 hours for one affinity matrix
- q-diffusion arguably makes parametric assumptions on the data that might be restrictive in some use cases, and no batch-effect correction was considered

## Evidence

- [results] we selected those organ tissues for which fewer than 10,000 cells were sampled. We also screened out the PBMCs: "we selected those organ tissues for which fewer than 10,000 cells were sampled. We also screened out the PBMCs"
- [results] the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells: "the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells"
- [results] Clustering parameters were optimized by a grid search over the silhouette scores44 for each case.: "Clustering parameters were optimized by a grid search over the silhouette scores44 for each case."
- [readme] The `*.normalized` element is simply log-normalized, sans feature selection or dimensionality reduction, like we do in our study.: "The `*.normalized` element is simply log-normalized, sans feature selection or dimensionality reduction, like we do in our study."
- [results] The Tabula Sapiens datasets were packaged with state-of-the-art dimensionality reductions, including scVI34: "The Tabula Sapiens datasets were packaged with state-of-the-art dimensionality reductions, including scVI34"
- [discussion] there is room for improving the runtime efficiency of the algorithm evaluating the q-diffused kernel across all pairs of cells.: "there is room for improving the runtime efficiency of the algorithm evaluating the q-diffused kernel across all pairs of cells."
