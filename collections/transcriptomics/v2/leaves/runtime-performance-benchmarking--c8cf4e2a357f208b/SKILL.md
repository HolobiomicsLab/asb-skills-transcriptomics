---
name: runtime-performance-benchmarking
description: Use when an exact computation with known excessive runtime (e.g. 20 min–3
  h per affinity matrix) must be replaced by an approximation and you must quantify
  the speedup without degrading a defined accuracy metric.
when_to_use_negative:
- The downstream task is exploratory visualization where no reference labels exist
  to compute an accuracy metric — the accept/reject criterion depends on a ground-truth
  ARI comparison
- The exact computation is already fast enough (small cell counts where the n-by-n
  kernel evaluation takes seconds), so an approximation introduces unnecessary error
  risk
- Batch-effect correction or dataset alignment is the bottleneck rather than kernel-evaluation
  runtime — the article notes no batch correction was considered in this framework
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: q-Diffusion.jl
  role: Provides the exact q-diffused kernel construction (AdaptiveKernel, DeformedKernel,
    estimate_inner_bandwidth, cuda_devices GPU acceleration, in_log) and the spatial-tree
    approximate options tree_leaf_size/n_kernel_neighbors used to build the sparse
    kernel variant
  repo: github.com/marmarelis/QDiffusion.jl
- name: Scanpy
  role: Neighborhood-graph toolkit against which q-diffusion clustering is benchmarked;
    representative of the tools whose ARI the exact baseline must preserve
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:d32a96a8fa7bd1ba4c27a3bdcf6504025fe064221993c5783903d30e8f5a12d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Scanpy
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# runtime-performance-benchmarking

## Summary

Benchmarks wall-clock runtime and downstream-task accuracy of an accelerated computational variant against a slow exact baseline, using the q-diffused kernel (20 min–3 h per n-by-n affinity matrix) versus an approximate nearest-neighbor-sparse kernel as the motivating case, judged by Leiden clustering ARI on Tabula Sapiens annotations.

## When to use

Use when an exact computation with known excessive runtime (e.g. 20 min–3 h per affinity matrix) must be replaced by an approximation and you must quantify the speedup without degrading a defined accuracy metric.

## When NOT to use

- The downstream task is exploratory visualization where no reference labels exist to compute an accuracy metric — the accept/reject criterion depends on a ground-truth ARI comparison
- The exact computation is already fast enough (small cell counts where the n-by-n kernel evaluation takes seconds), so an approximation introduces unnecessary error risk
- Batch-effect correction or dataset alignment is the bottleneck rather than kernel-evaluation runtime — the article notes no batch correction was considered in this framework

## Inputs

- Log-normalized gene-by-cell count matrix (fewer than 10,000 cells; Tabula Sapiens organ-tissue sample with PBMCs screened out)
- Exact q-diffused kernel affinity matrix (q=1.2, inner bandwidth = 50% quantile of gene-wise average pairwise squared distances, adaptive outer bandwidth at the 16th nearest neighbor)
- Approximate kernel affinity matrix restricted to k nearest neighbors via approximate nearest-neighbor search
- Manually annotated cell-phenotype labels (Tabula Sapiens atlas) as reference ground truth

## Outputs

- Wall-clock runtime measurements for the exact and approximate affinity-matrix evaluations
- Adjusted Rand indices of Leiden clusterings from both kernels against manual annotations
- A comparison table such as ann_benchmark.csv with runtime and ARI columns for exact vs. approximate kernels

## How to apply

First time the exact baseline end-to-end: build the exact q-diffused kernel with AdaptiveKernel(16, DeformedKernel(q, data, inner_bandwidth), cuda_devices, in_log=true), evaluate the full n-by-n affinity matrix on GPU, and record wall-clock time. Then implement the approximate variant by restricting kernel evaluation to each cell's k nearest neighbors via a spatial-tree structure (the tree_leaf_size and n_kernel_neighbors options in QDiffusion.jl), recording its runtime identically. Symmetrize both affinity matrices arithmetically as A=(A~+A~ᵀ)/2, run Leiden community detection on each weighted undirected graph, and compute adjusted Rand indices against expert manual annotations (here the manually annotated Tabula Sapiens cell phenotypes). Tabulate runtime and ARI for exact versus approximate variants (e.g. ann_benchmark.csv) and accept the approximation only if runtime is reduced while ARI stays close to the exact kernel's ARI.

## Related tools

- **q-Diffusion.jl** (Provides the exact q-diffused kernel construction (AdaptiveKernel, DeformedKernel, estimate_inner_bandwidth, cuda_devices GPU acceleration, in_log) and the spatial-tree approximate options tree_leaf_size/n_kernel_neighbors used to build the sparse kernel variant) — github.com/marmarelis/QDiffusion.jl
- **Scanpy** (Neighborhood-graph toolkit against which q-diffusion clustering is benchmarked; representative of the tools whose ARI the exact baseline must preserve)

## Examples

```
inner_bandwidth = estimate_inner_bandwidth(data); kernel = AdaptiveKernel(16, DeformedKernel(1.2f0, data, inner_bandwidth=inner_bandwidth), data, cuda_devices=[0], in_log=true)
```

## Evaluation signals

- Approximate-kernel runtime is measurably lower than the exact evaluation (exact baseline reported at 20 min–3 h per affinity matrix)
- Leiden ARI of the approximate kernel is close to the exact kernel's ARI against the same manual Tabula Sapiens annotations
- Both kernels' affinity matrices are symmetrized identically, A=(A~+A~ᵀ)/2, before clustering so the only variable is kernel sparsification
- Inner bandwidth for both variants is estimated identically (50% quantile of gene-wise average pairwise squared distances) and the adaptive outer bandwidth targets the 16th nearest neighbor
- Comparison table (ann_benchmark.csv) contains runtime and ARI rows for both exact and approximate kernels

## Limitations

- The exact q-diffused kernel runtime is suboptimal: the article notes 'there is room for improving the runtime efficiency of the algorithm evaluating the q-diffused kernel across all pairs of cells' with 20 min to 3 h per affinity matrix
- No algorithm is provided for selecting the q-diffusion parameters q and inner bandwidth φ, so benchmark fairness depends on holding these fixed across variants
- Spatial-tree kNN options in QDiffusion.jl are documented for CPU mode (tree_leaf_size, n_kernel_neighbors), which may constrain GPU-accelerated approximate evaluation

## Evidence

- [discussion] Runtime of the q-diffused kernel evaluation is suboptimal: "there is room for improving the runtime efficiency of the algorithm evaluating the q-diffused kernel across all pairs of cells"
- [readme] ANN sparsification options in README: "tree_leaf_size=0, # this and the below are for CPU mode, n_kernel_neighbors=0, # if you want to use a spatial tree structure"
- [results] Exact kernel benchmark ground truth: "the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells"
- [results] ARI evaluation metric: "Adjusted Rand indices (ARIs)31 of Leiden community detection42 in the PBMC dataset"
- [readme] Kernel construction parameters from README: "kernel = AdaptiveKernel(16, # scale to the 16th nearest neighbor
  DeformedKernel(1.5f0, # set deformation parameter q=1.5"
