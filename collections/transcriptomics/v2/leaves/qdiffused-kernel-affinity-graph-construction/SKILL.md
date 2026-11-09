---
name: qdiffused-kernel-affinity-graph-construction
description: Use when you need full-dimensionality single-cell clustering or embedding
  without feature selection or dimensionality reduction — e.g., unsupervised clustering
  of small organ-tissue samples (< 10,000 cells) benchmarked against scVI/Seurat embeddings.
when_to_use_negative:
- The data require batch-effect correction or alignment of disparate datasets — the
  paper explicitly did not consider these, and suggests coopting pre-existing tools
  upstream instead
- A fast standard pipeline suffices — q-diffused kernel evaluation takes 20 min to
  3 h per affinity matrix, far slower than neighborhood-graph methods
- Cells need direct interpretability of the nonlinearities — the paper notes q-diffusion's
  nonlinearities are less directly interpretable than linear methods
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: q-Diffusion.jl
  role: Julia package providing DeformedKernel, GaussianKernel, AdaptiveKernel, estimate_inner_bandwidth,
    and preprocess_cell_counts for building the q-diffused affinity graph
  repo: github.com/marmarelis/QDiffusion.jl
- name: q-diffusion
  role: The framework whose q-diffused kernel geometry captures coexpression structure
    of an entire library of genes in full dimensionality
  repo: github.com/marmarelis/QDiffusion.jl
- name: PHATE
  role: 'Downstream consumer of the q-diffused kernel: PHATE with a q-diffused kernel
    yields the qPHATE embedding'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:75d902374435bbb6d0a73f8ac597e77b39b62305f3277cca553d12d7a549743c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle
  - PHATE
  - SC3
  - Scanpy
  - Seurat
  - q-Diffusion.jl
  - q-diffusion
  - scVI
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# qdiffused-kernel-affinity-graph-construction

## Summary

Builds a weighted undirected cell-cell affinity graph from log-normalized gene-by-cell scRNAseq data using the q-diffused kernel (deformation parameter q > 1) wrapped in an adaptive k-nn kernel, as implemented in QDiffusion.jl. This graph is the substrate for downstream Leiden clustering, PHATE embedding, and NMF analyses.

## When to use

Use when you need full-dimensionality single-cell clustering or embedding without feature selection or dimensionality reduction — e.g., unsupervised clustering of small organ-tissue samples (< 10,000 cells) benchmarked against scVI/Seurat embeddings.

## When NOT to use

- The data require batch-effect correction or alignment of disparate datasets — the paper explicitly did not consider these, and suggests coopting pre-existing tools upstream instead
- A fast standard pipeline suffices — q-diffused kernel evaluation takes 20 min to 3 h per affinity matrix, far slower than neighborhood-graph methods
- Cells need direct interpretability of the nonlinearities — the paper notes q-diffusion's nonlinearities are less directly interpretable than linear methods

## Inputs

- Log-normalized gene-by-cell count matrix (e.g., from preprocess_cell_counts(data).normalized)
- Deformation parameter q (e.g., 1.2 for Tabula Sapiens clustering, 1.5 in README example)
- Inner bandwidth φ, estimated by estimate_inner_bandwidth(data)
- Adaptive kernel k-nn scale (16 in the examples)
- CUDA device IDs (optional, for GPU acceleration)

## Outputs

- Symmetrized weighted undirected affinity graph A = (Ã + Ãᵀ)/2 over all cell pairs
- AdaptiveKernel/DeformedKernel object reusable for NMF, qPHATE, and local distributional segmentation

## How to apply

Start with a log-normalized (genes x cells) matrix — QDiffusion.jl's preprocess_cell_counts returns the .normalized version without feature selection or dimensionality reduction. Estimate the inner bandwidth via estimate_inner_bandwidth(data), a heuristic controlling the multiscale behavior of the deformed kernel. Construct the kernel by wrapping a DeformedKernel (choose deformation parameter q > 1, e.g., 1.2-1.5; q=1 recovers a Gaussian kernel) in an AdaptiveKernel scaled to the 16th nearest neighbor, passing cuda_devices for GPU acceleration and in_log=true for numerical stability. Evaluate the q-diffused kernel between all pairs of cells, then symmetrize the anisotropic affinity matrix arithmetically, A = (Ã + Ãᵀ)/2, to produce a weighted undirected graph. Note runtime can be 20 minutes to 3 hours per affinity matrix; the k-nn distance used by AdaptiveKernel should reflect sample density (k=16 for small tissue samples in the LDS use case).

## Related tools

- **q-Diffusion.jl** (Julia package providing DeformedKernel, GaussianKernel, AdaptiveKernel, estimate_inner_bandwidth, and preprocess_cell_counts for building the q-diffused affinity graph) — github.com/marmarelis/QDiffusion.jl
- **q-diffusion** (The framework whose q-diffused kernel geometry captures coexpression structure of an entire library of genes in full dimensionality) — github.com/marmarelis/QDiffusion.jl
- **PHATE** (Downstream consumer of the q-diffused kernel: PHATE with a q-diffused kernel yields the qPHATE embedding)

## Examples

```
kernel = AdaptiveKernel(16, DeformedKernel(1.2f0, data, inner_bandwidth=estimate_inner_bandwidth(data)), data, cuda_devices=[0], in_log=true)
```

## Evaluation signals

- The kernel graph must be symmetric after applying A = (Ã + Ãᵀ)/2, i.e., A equals Aᵀ
- Downstream Leiden community detection on the graph should yield clusters whose ARI against expert annotations exceeds atlas-supplied embeddings (e.g., scVI) in 3 of 4 Tabula Sapiens organs (skin, trachea, uterus), while liver may degrade
- Clustering parameters should be optimized by grid search over silhouette scores per case
- Setting q = 1 with the same pipeline should reproduce a plain Gaussian-kernel graph, providing a sanity baseline
- Log-transform of kernel computations (in_log=true) must be enabled for numerical stability; check for NaN/Inf in the affinity matrix if omitted

## Limitations

- Runtime of evaluating the q-diffused kernel across all pairs of cells is suboptimal: 20 minutes to 3 hours for one affinity matrix
- No algorithm is provided for selecting the two vital parameters, deformation parameter q and inner bandwidth φ
- q-diffusion arguably makes parametric assumptions on the data that might be restrictive in some use cases
- Nonlinearities of q-diffusion are less directly interpretable, generally
- No batch-effect correction or alignment of disparate datasets was considered; pre-existing tools must be coopted upstream

## Evidence

- [results] Kernel construction workflow: "the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells"
- [methods] Inner bandwidth estimation: "inner_bandwidth = estimate_inner_bandwidth(data)"
- [methods] Log-normalized input without feature selection: "There is also a convenience mathod called `preprocess_cell_counts(data)` that returns a named tuple containing different versions of the matrix"
- [readme] Deformation parameter choice: "DeformedKernel(1.5f0, # set deformation parameter q=1.5
    # (kernel is less Gaussian the farther q>1 is from 1)"
- [readme] Numerical stability requirement: "in_log=true) # log-transform (necessary for numerical stability)"
- [results] Small-sample organ selection filter: "we selected those organ tissues for which fewer than 10,000 cells were sampled. We also screened out the PBMCs"
- [results] Silhouette-based parameter optimization: "Clustering parameters were optimized by a grid search over the silhouette scores44 for each case."
- [discussion] Runtime limitation: "there is room for improving the runtime efficiency of the algorithm evaluating the q-diffused kernel across all pairs of cells."
