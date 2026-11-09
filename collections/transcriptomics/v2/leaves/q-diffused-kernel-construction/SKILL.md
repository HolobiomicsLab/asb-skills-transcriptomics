---
name: q-diffused-kernel-construction
description: Use when you have a log-normalized (genes x cells) scRNAseq matrix and
  need an affinity/graph structure over cells that preserves high-order gene interactions
  for downstream clustering, NMF, PHATE, or MMD-based spatial segmentation.
when_to_use_negative:
- When your pipeline already relies on aggressive feature selection, dimensionality
  reduction, or marker-gene identification — q-diffusion's purpose is to work in full
  dimensionality instead
- When the input is not per-cell expression data (the kernel is defined over cells
  in transcriptomic observational space; it does not directly address batch-effect
  correction or alignment of disparate datasets)
- When a fast, well-understood Gaussian-kernel graph suffices and full-dimensionality
  coexpression structure is not needed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: q-Diffusion.jl
  role: Julia package providing DeformedKernel, GaussianKernel, AdaptiveKernel, estimate_inner_bandwidth,
    and preprocess_cell_counts used to construct and evaluate the q-diffused kernel
  repo: github.com/marmarelis/QDiffusion.jl
- name: q-diffusion
  role: the method/framework implemented; the deformed kernel induces the cell graph
    used by all downstream analyses
- name: Scanpy
  role: Gaussian-kernel baseline neighborhood graph method for comparison of downstream
    clustering accuracy
- name: Seurat
  role: competing neighborhood graph method benchmarked against the q-diffused graph
- name: PHATE
  role: embedding method that consumes the q-diffused kernel to yield qPHATE; also
    a baseline kernel variant
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_006
  - task_007
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:3566ab3efc9404121bb12c94285028efac553a86c1ee737d3f40b87bb0a8141b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle
  - PHATE
  - QDiffusion.jl
  - SC3
  - Scanpy
  - Seurat
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# q-diffused-kernel-construction

## Summary

Constructs the q-diffused kernel of cells in transcriptomic space using QDiffusion.jl — a DeformedKernel (non-Euclidean deformation parameter q) wrapped in an AdaptiveKernel with a heuristic inner bandwidth — to capture full-dimensionality gene coexpression without feature selection or dimensionality reduction.

## When to use

Use when you have a log-normalized (genes x cells) scRNAseq matrix and need an affinity/graph structure over cells that preserves high-order gene interactions for downstream clustering, NMF, PHATE, or MMD-based spatial segmentation.

## When NOT to use

- When your pipeline already relies on aggressive feature selection, dimensionality reduction, or marker-gene identification — q-diffusion's purpose is to work in full dimensionality instead
- When the input is not per-cell expression data (the kernel is defined over cells in transcriptomic observational space; it does not directly address batch-effect correction or alignment of disparate datasets)
- When a fast, well-understood Gaussian-kernel graph suffices and full-dimensionality coexpression structure is not needed

## Inputs

- log-normalized (genes x cells) scRNAseq count matrix (e.g., the *.normalized element returned by preprocess_cell_counts)
- inner bandwidth scalar from estimate_inner_bandwidth (median quantile of per-gene average pairwise squared distances)
- optional (dims x cells) spatial coordinates matrix for LDS/MERFISH use cases

## Outputs

- symmetrized anisotropic affinity matrix A = (A~ + A~T)/2 over cells (weighted undirected graph)
- n-by-n q-diffused kernel evaluation between all cell pairs (with plain GaussianKernel as the standard baseline comparison)
- kernel object consumable by par_estimate_nmf, leave_it_to_phate, par_spatially_cluster_mmd, and Leiden community detection

## How to apply

Log-normalize the (genes x cells) count matrix (or use the *.normalized element of preprocess_cell_counts), then estimate the inner bandwidth with estimate_inner_bandwidth, which takes the 50% (median) quantile of per-gene average pairwise squared distances. Build a DeformedKernel with deformation parameter q=1.2 (the paper's setting; q=1.5 in the README example) wrapped in an AdaptiveKernel whose outer bandwidth is scaled to the k-th nearest q-diffused norm (k=16), passing cuda_devices for GPU acceleration and in_log=true for numerical stability. Evaluate the kernel between all cell pairs and symmetrize the anisotropic affinity matrix arithmetically as A=(A~+A~T)/2 to yield a weighted, undirected graph. The inner bandwidth 'tells us what gene expressions to take seriously' and controls the multiscale nature of the deformed kernel; the farther q>1 is from 1, the less Gaussian the kernel. Downstream steps (Leiden community detection, par_estimate_nmf, leave_it_to_phate, par_spatially_cluster_mmd) all consume this kernel or its affinity matrix.

## Related tools

- **q-Diffusion.jl** (Julia package providing DeformedKernel, GaussianKernel, AdaptiveKernel, estimate_inner_bandwidth, and preprocess_cell_counts used to construct and evaluate the q-diffused kernel) — github.com/marmarelis/QDiffusion.jl
- **q-diffusion** (the method/framework implemented; the deformed kernel induces the cell graph used by all downstream analyses)
- **Scanpy** (Gaussian-kernel baseline neighborhood graph method for comparison of downstream clustering accuracy)
- **Seurat** (competing neighborhood graph method benchmarked against the q-diffused graph)
- **PHATE** (embedding method that consumes the q-diffused kernel to yield qPHATE; also a baseline kernel variant)

## Examples

```
kernel = AdaptiveKernel(16, DeformedKernel(1.5f0, data, inner_bandwidth=1.125f0), data, cuda_devices=[0], in_log=true)
```

## Evaluation signals

- The affinity matrix is square, symmetric, and nonnegative with zero diagonal after arithmetic symmetrization A=(A~+A~T)/2
- The q-diffused variant is compared against a plain GaussianKernel on the same data; downstream tasks (e.g., Leiden ARI on Tabula Sapiens annotations or PBMC IFN-γ stimulation ARIs) should favor the q-diffused graph
- Inner bandwidth reproduces the heuristic: the 50% (median) quantile of per-gene average pairwise squared distances (estimate_inner_bandwidth output)
- Kernel computation runs with in_log=true (numerical stability) and consistent q (e.g., 1.2) and k=16 adaptive outer bandwidth across experiments
- Downstream qNMF at rank=16 with a fixed seed yields top-weighted GEP genes almost equivalent to plain NMF, as the paper reports

## Limitations

- Full all-pairs kernel evaluation is slow: 20 minutes to 3 hours per affinity matrix; sparsification via approximate nearest-neighbor (tree_leaf_size/n_kernel_neighbors) trades some accuracy for runtime
- No algorithm is provided for selecting the two vital parameters q and inner bandwidth φ — no automated parameter-selection procedure exists
- q-diffusion makes parametric assumptions on the data that might be restrictive in some use cases, and its nonlinearities are less directly interpretable in general
- No batch-effect correction or alignment of disparate datasets is considered — pre-existing tools would need to be coopted upstream

## Evidence

- [readme] It is recommended to wrap one of these in an `AdaptiveKernel` type that adjusts the base kernel's bandwidth relative to $k$-nn distances.: "It is recommended to wrap one of these in an `AdaptiveKernel` type that adjusts the base kernel's bandwidth relative to $k$-nn distances."
- [readme] The inner bandwidth controls the multiscale nature of the deformed kernel.: "The inner bandwidth controls the multiscale nature of the deformed kernel. Very approximately, it tells us what gene expressions to take seriously."
- [methods] There is also a convenience mathod called `preprocess_cell_counts(data)`: "There is also a convenience mathod called `preprocess_cell_counts(data)` that returns a named tuple containing different versions of the matrix"
- [methods] inner_bandwidth = estimate_inner_bandwidth(data): "inner_bandwidth = estimate_inner_bandwidth(data)"
- [results] the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified: "the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells"
- [discussion] Such an exploration would invite a careful selection algorithm for non-Euclidean deformation q: "Such an exploration would invite a careful selection algorithm for non-Euclidean deformation q and inner bandwidth φ, the two vital parameters to q-diffusion."
- [discussion] there is room for improving the runtime efficiency of the algorithm evaluating the q-diffused kernel: "there is room for improving the runtime efficiency of the algorithm evaluating the q-diffused kernel across all pairs of cells."
