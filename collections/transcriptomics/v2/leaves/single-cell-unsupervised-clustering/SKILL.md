---
name: single-cell-unsupervised-clustering
description: Use when clustering unlabeled single-cell transcriptomes (e.g. small
  organ tissue samples from the Tabula Sapiens atlas with manually annotated phenotypes)
  and you want to capture high-order gene coexpression interactions rather than rely
  on PCA/scVI reductions.
when_to_use_negative:
- Datasets requiring batch-effect correction or alignment of disparate datasets —
  the paper explicitly did not consider batch-effect correction
- Very large samples where the O(n²) kernel evaluation is prohibitive — q-diffused
  kernel runtime is 20 min to 3 h for one affinity matrix
- Analyses needing directly interpretable nonlinearities, since q-diffusion's nonlinearities
  are less directly interpretable in general
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: QDiffusion.jl
  role: Julia package providing preprocess_cell_counts, estimate_inner_bandwidth,
    AdaptiveKernel/DeformedKernel construction, and GPU-accelerated (cuda_devices)
    kernel evaluation
  repo: github.com/marmarelis/QDiffusion.jl
- name: Leiden
  role: Community detection on the symmetrized q-diffused weighted undirected graph
    to produce cell clusters
- name: scVI
  role: State-of-the-art dimensionality reduction supplied with Tabula Sapiens, used
    as a benchmark embedding for ARI comparison
- name: Seurat
  role: Neighborhood-graph clustering baseline for ARI comparison
- name: Scanpy
  role: Neighborhood-graph clustering baseline for ARI comparison
- name: Monocle
  role: Neighborhood-graph clustering baseline for ARI comparison
- name: SC3
  role: Clustering baseline for ARI comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:ef335e10758163cc4ef03e0101f6711dde50649a4cd8c67f0321b55bbb9dc289
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# single-cell-unsupervised-clustering

## Summary

Unsupervised clustering of scRNAseq cells via q-diffusion: build a q-diffused adaptive kernel over the full gene-by-cell matrix (no feature selection or dimensionality reduction), symmetrize it into a weighted graph, and run Leiden community detection, benchmarking cluster quality against atlas-supplied embeddings with adjusted Rand indices.

## When to use

Use when clustering unlabeled single-cell transcriptomes (e.g. small organ tissue samples from the Tabula Sapiens atlas with manually annotated phenotypes) and you want to capture high-order gene coexpression interactions rather than rely on PCA/scVI reductions.

## When NOT to use

- Datasets requiring batch-effect correction or alignment of disparate datasets — the paper explicitly did not consider batch-effect correction
- Very large samples where the O(n²) kernel evaluation is prohibitive — q-diffused kernel runtime is 20 min to 3 h for one affinity matrix
- Analyses needing directly interpretable nonlinearities, since q-diffusion's nonlinearities are less directly interpretable in general

## Inputs

- (genes x cells) count matrix from single-cell RNAseq (e.g. Tabula Sapiens organ tissue samples with fewer than 10,000 sampled cells, PBMCs screened out)
- Manually annotated cell phenotypes (e.g. Tabula Sapiens expert annotations) for ARI evaluation
- Optional: dimensionality reductions supplied with the dataset (e.g. scVI) for benchmark comparison

## Outputs

- Leiden cluster assignments per cell on the symmetrized q-diffused affinity graph
- Per-dataset ARI values comparing q-diffusion clusters to baseline methods
- ARI comparison table (e.g. tabula_sapiens_ari_comparison.csv)

## How to apply

Log-normalize the (genes x cells) matrix with preprocess_cell_counts in QDiffusion.jl (no feature selection or dimensionality reduction). Estimate the inner bandwidth via estimate_inner_bandwidth(data), build an AdaptiveKernel (scaled to the 16th nearest neighbor) wrapping a DeformedKernel with deformation parameter q = 1.2 and in_log=true for numerical stability. Evaluate the q-diffused kernel between all pairs of cells and symmetrize the anisotropic affinity matrix arithmetically, A = (Ã + Ãᵀ)/2, into a weighted undirected graph. Run Leiden community detection optimizing modularity, with clustering parameters optimized per dataset by a grid search over silhouette scores, then score clusters against expert annotations using adjusted Rand indices (ARI) alongside baselines such as scVI, Seurat, Monocle, Scanpy, and SC3. Rationale: q-diffusion retains the full dimensionality of gene coexpression, so kernel geometry — not projected latent space — drives cluster structure.

## Related tools

- **QDiffusion.jl** (Julia package providing preprocess_cell_counts, estimate_inner_bandwidth, AdaptiveKernel/DeformedKernel construction, and GPU-accelerated (cuda_devices) kernel evaluation) — github.com/marmarelis/QDiffusion.jl
- **Leiden** (Community detection on the symmetrized q-diffused weighted undirected graph to produce cell clusters)
- **scVI** (State-of-the-art dimensionality reduction supplied with Tabula Sapiens, used as a benchmark embedding for ARI comparison)
- **Seurat** (Neighborhood-graph clustering baseline for ARI comparison)
- **Scanpy** (Neighborhood-graph clustering baseline for ARI comparison)
- **Monocle** (Neighborhood-graph clustering baseline for ARI comparison)
- **SC3** (Clustering baseline for ARI comparison)

## Examples

```
data = preprocess_cell_counts(counts).normalized; inner_bandwidth = estimate_inner_bandwidth(data); kernel = AdaptiveKernel(16, DeformedKernel(1.2f0, data, inner_bandwidth=inner_bandwidth), data, in_log=true)
```

## Evaluation signals

- ARI of Leiden clusters against manually annotated cell phenotypes exceeds that of atlas-supplied reductions (scVI) and neighborhood-graph baselines (Seurat, Monocle, Scanpy, SC3) on the same tissue
- Inner bandwidth is estimated from the median (50% quantile) of per-gene average pairwise squared distances via estimate_inner_bandwidth, not hand-tuned arbitrarily
- Affinity matrix is symmetric after arithmetic symmetrization A = (Ã + Ãᵀ)/2
- Clustering parameters are selected by grid search over silhouette scores per dataset
- Kernel built with in_log=true for numerical stability; per-organ ARI pattern (wins in skin, trachea, uterus; degradation in liver) reproducible

## Limitations

- No algorithm for selecting the q-diffusion parameters q and φ is provided; careful selection of non-Euclidean deformation q and inner bandwidth φ remains open
- Runtime of the q-diffused kernel evaluation is suboptimal (20 min to 3 h for one affinity matrix)
- q-diffusion makes parametric assumptions on the data that might be restrictive in some use cases
- Nonlinearities of q-diffusion are less directly interpretable, generally
- No batch-effect correction or alignment of disparate datasets was considered

## Evidence

- [results] workflow: kernel construction and clustering: "the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells"
- [results] grid search optimization: "Clustering parameters were optimized by a grid search over the silhouette scores44 for each case."
- [results] Tabula Sapiens filter: "we selected those organ tissues for which fewer than 10,000 cells were sampled. We also screened out the PBMCs"
- [readme] README kernel example: "kernel = AdaptiveKernel(16, # scale to the 16th nearest neighbor
  DeformedKernel(1.5f0, # set deformation parameter q=1.5"
- [readme] README inner bandwidth heuristic: "inner_bandwidth = estimate_inner_bandwidth(data)"
- [readme] README preprocessing: "There is also a convenience mathod called `preprocess_cell_counts(data)` that returns a named tuple containing different versions of the matrix"
- [discussion] discussion limitation: parameter selection: "Such an exploration would invite a careful selection algorithm for non-Euclidean deformation q and inner bandwidth φ, the two vital parameters to q-diffusion."
- [discussion] discussion limitation: runtime: "there is room for improving the runtime efficiency of the algorithm evaluating the q-diffused kernel across all pairs of cells."
