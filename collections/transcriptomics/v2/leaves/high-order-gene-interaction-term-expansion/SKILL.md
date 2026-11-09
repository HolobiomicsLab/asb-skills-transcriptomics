---
name: high-order-gene-interaction-term-expansion
description: Use when building a cell-to-cell affinity matrix from raw scRNAseq counts
  where the analysis goal is full-dimensionality similarity — capturing high-order
  gene interactions without feature selection, PCA, or marker-gene filtering.
when_to_use_negative:
- Data have already undergone feature selection, dimensionality reduction, or marker-gene
  identification — q-diffusion requires the full-dimensional log-normalized matrix
- Disparate datasets needing batch-effect correction or alignment are involved — the
  paper explicitly did not consider batch-effect correction upstream of q-diffusion
- Interactive analyses on very large datasets where a 20 min to 3 h affinity-matrix
  runtime is infeasible
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: q-Diffusion.jl
  role: Julia package providing preprocess_cell_counts, estimate_inner_bandwidth,
    DeformedKernel, and AdaptiveKernel to build the q-diffused affinity kernel
  repo: https://github.com/marmarelis/QDiffusion.jl
- name: q-diffusion
  role: The underlying framework whose kernel-like similarity function automatically
    incorporates all orders of gene-interaction terms weighted by decay parameter
    α
  repo: https://github.com/marmarelis/QDiffusion.jl
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:8351149949d23db74bb1819851db5e20405ecbbda9552f34154e5253437b1d2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marmarelis/QDiffusion.jl
  tools:
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# high-order-gene-interaction-term-expansion

## Summary

Constructs a q-diffused kernel over log-normalized scRNAseq cells that expands all orders of gene-interaction terms (pairwise, trilinear, quadrilinear, …) weighted by an α-decay, yielding a symmetric cell-by-cell affinity graph. It captures combinatorial coexpression structure that feature selection, dimensionality reduction, or marker-gene pipelines discard.

## When to use

Use when building a cell-to-cell affinity matrix from raw scRNAseq counts where the analysis goal is full-dimensionality similarity — capturing high-order gene interactions without feature selection, PCA, or marker-gene filtering.

## When NOT to use

- Data have already undergone feature selection, dimensionality reduction, or marker-gene identification — q-diffusion requires the full-dimensional log-normalized matrix
- Disparate datasets needing batch-effect correction or alignment are involved — the paper explicitly did not consider batch-effect correction upstream of q-diffusion
- Interactive analyses on very large datasets where a 20 min to 3 h affinity-matrix runtime is infeasible

## Inputs

- Raw scRNAseq count matrix, (genes × cells), e.g. the in vitro PBMC dataset (optionally IFN-γ-stimulated)
- Deformation parameter q (with α = q − 1, 0 < α < 1), e.g. q = 1.2
- Inner bandwidth φ (estimated via estimate_inner_bandwidth as the median quantile of gene scatters)
- Outer adaptive bandwidth: 16th-nearest-neighbor distance (PHATE adaptive-resolution scheme)

## Outputs

- Symmetric n×n affinity matrix (affinity_matrix.npy) with kernel values in (0,1]
- Weighted undirected cell-cell graph (adjacency matrix) usable downstream by Leiden community detection or qPHATE

## How to apply

Load the PBMC scRNAseq count matrix as a (genes × cells) matrix and log-normalize it via QDiffusion.jl's preprocess_cell_counts, with no feature selection or dimensionality reduction. Estimate the inner bandwidth φ with estimate_inner_bandwidth (average pairwise squared distance per gene, median quantile of gene scatters), which 'tells us what gene expressions to take seriously'. Build a DeformedKernel with deformation parameter q (α = q − 1, with 0 < α < 1; e.g. q = 1.2), then wrap it in an AdaptiveKernel that scales the outer bandwidth ρ(k) to the 16th-nearest-neighbor q-diffused norm, computed in log space (in_log=true) for numerical stability. Evaluate the q-diffused kernel between all cell pairs — the recursive q-sum expands α-discounted pair, trilinear, and higher-order gene products — then symmetrize the anisotropic kernel arithmetically as A = (Ã + Ãᵀ)/2 to produce a weighted undirected graph.

## Related tools

- **q-Diffusion.jl** (Julia package providing preprocess_cell_counts, estimate_inner_bandwidth, DeformedKernel, and AdaptiveKernel to build the q-diffused affinity kernel) — https://github.com/marmarelis/QDiffusion.jl
- **q-diffusion** (The underlying framework whose kernel-like similarity function automatically incorporates all orders of gene-interaction terms weighted by decay parameter α) — https://github.com/marmarelis/QDiffusion.jl

## Examples

```
inner_bandwidth = estimate_inner_bandwidth(data); kernel = AdaptiveKernel(16, DeformedKernel(1.2f0, data, inner_bandwidth=inner_bandwidth), data, in_log=true)
```

## Evaluation signals

- affinity_matrix.npy exists and has shape n×n matching the cell count
- Affinity matrix is exactly symmetric after the arithmetic symmetrization A = (Ã + Ãᵀ)/2
- All kernel values lie in (0,1], as expected for a q-exponential kernel
- Deformation parameter satisfies α = q − 1 with 0 < α < 1 (e.g. q = 1.2 → α = 0.2)
- Kernel evaluation ran with in_log=true — log-space computation is stated as necessary for numerical stability

## Limitations

- No algorithm is provided for selecting q and φ; 'Such an exploration would invite a careful selection algorithm for non-Euclidean deformation q and inner bandwidth φ'
- Runtime of evaluating the q-diffused kernel across all cell pairs is suboptimal — 20 min to 3 h for one affinity matrix (GPU acceleration via cuda_devices helps)
- q-diffusion makes parametric assumptions on the data that might be restrictive in some use cases, and its nonlinearities are less directly interpretable in general
- No batch-effect correction or alignment of disparate datasets was considered in the study

## Evidence

- [readme] The inner bandwidth controls the multiscale nature of the deformed kernel. Very approximately, it tells us what gene expressions to take seriously.: "The inner bandwidth controls the multiscale nature of the deformed kernel. Very approximately, it tells us what gene expressions to take seriously."
- [readme] Convenience preprocessing: "There is also a convenience mathod called `preprocess_cell_counts(data)` that returns a named tuple containing different versions of the matrix due to common preprocessing pipelines."
- [results] Symmetrization into graph: "the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells"
- [methods] Inner bandwidth estimation call: "inner_bandwidth = estimate_inner_bandwidth(data)"
- [discussion] Runtime limitation: "there is room for improving the runtime efficiency of the algorithm evaluating the q-diffused kernel across all pairs of cells."
