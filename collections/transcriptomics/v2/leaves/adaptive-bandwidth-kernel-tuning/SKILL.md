---
name: adaptive-bandwidth-kernel-tuning
description: Use when building cell-to-cell affinity matrices from raw or log-normalized
  scRNAseq count matrices in full dimensionality, and kernel bandwidths must adapt
  to local density via k-NN scaling rather than fixed global scales.
when_to_use_negative:
- Data has already undergone aggressive feature selection, dimensionality reduction,
  or marker-gene identification — q-diffusion's purpose is to capture high-order gene
  interactions in full dimensionality, so those stages should be avoided upstream.
- Analysis requires batch-effect correction or alignment of disparate datasets — the
  paper explicitly states this was not considered.
- Datasets are so large that a 20 min to 3 h kernel-evaluation runtime is prohibitive,
  since the full all-pairs q-diffused kernel evaluation is acknowledged as runtime-suboptimal.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: q-Diffusion.jl
  role: Julia package providing AdaptiveKernel, DeformedKernel, estimate_inner_bandwidth,
    and preprocess_cell_counts used to build the adaptive-bandwidth q-diffused kernel
  repo: github.com/marmarelis/QDiffusion.jl
- name: PHATE
  role: Source of the adaptive-resolution scheme in which the outer bandwidth is set
    to the k-th nearest-neighbor distance
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:2a4a2c38d6cacf568e4ae93d1401b430017c1b35f320506aa8c6720fc97dc9b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PHATE
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# adaptive-bandwidth-kernel-tuning

## Summary

Tunes the bandwidths of a q-diffused kernel on scRNAseq count data by estimating the inner bandwidth from gene-wise scatter and wrapping the base kernel in an AdaptiveKernel that rescales bandwidth to k-nearest-neighbor distances (PHATE-style adaptive resolution), enabling full-dimensionality cell-to-cell affinity computation.

## When to use

Use when building cell-to-cell affinity matrices from raw or log-normalized scRNAseq count matrices in full dimensionality, and kernel bandwidths must adapt to local density via k-NN scaling rather than fixed global scales.

## When NOT to use

- Data has already undergone aggressive feature selection, dimensionality reduction, or marker-gene identification — q-diffusion's purpose is to capture high-order gene interactions in full dimensionality, so those stages should be avoided upstream.
- Analysis requires batch-effect correction or alignment of disparate datasets — the paper explicitly states this was not considered.
- Datasets are so large that a 20 min to 3 h kernel-evaluation runtime is prohibitive, since the full all-pairs q-diffused kernel evaluation is acknowledged as runtime-suboptimal.

## Inputs

- (genes x cells) scRNAseq count matrix, log-normalized (e.g. PBMC dataset with no feature selection or dimensionality reduction)
- deformation parameter q (e.g. q=1.2, alpha = q-1)
- k-NN neighborhood size for adaptive scaling (k=16 following the PHATE adaptive-resolution scheme)

## Outputs

- n x n symmetrized affinity matrix (weighted undirected graph over cells, e.g. saved as affinity_matrix.npy)
- estimated inner bandwidth scalar (from estimate_inner_bandwidth)

## How to apply

Load the (genes x cells) count matrix and log-normalize it without feature selection or dimensionality reduction, e.g. via preprocess_cell_counts using the .normalized variant. Estimate the inner bandwidth with estimate_inner_bandwidth(data), which computes the average pairwise squared distance per gene and takes the 50% (median) quantile of gene scatters; choose the deformation parameter q (e.g. 1.2 or 1.5, with alpha = q - 1 in (0,1)). Wrap a DeformedKernel in an AdaptiveKernel with k=16 nearest neighbors, setting the outer bandwidth rho(k) to the 16th-NN q-diffused norm; set in_log=true so the adaptive scaling is computed in log space for numerical stability, and set bandwidth=1.0 when using the adaptive kernel downstream. Optionally pass cuda_devices for GPU acceleration of the cell-by-cell, gene-to-gene computations. Evaluate the kernel between all cell pairs and arithmetically symmetrize the matrix as A = (A + A^T)/2 into a weighted undirected graph.

## Related tools

- **q-Diffusion.jl** (Julia package providing AdaptiveKernel, DeformedKernel, estimate_inner_bandwidth, and preprocess_cell_counts used to build the adaptive-bandwidth q-diffused kernel) — github.com/marmarelis/QDiffusion.jl
- **PHATE** (Source of the adaptive-resolution scheme in which the outer bandwidth is set to the k-th nearest-neighbor distance)

## Examples

```
kernel = AdaptiveKernel(16, DeformedKernel(1.5f0, data, inner_bandwidth=1.125f0), data, cuda_devices=[0,1], in_log=true)
```

## Evaluation signals

- affinity_matrix.npy exists and has shape n x n matching the cell count
- The kernel matrix is symmetric after arithmetic symmetrization A = (A + A^T)/2
- Kernel values lie in (0, 1] as expected for a q-exponential kernel
- The adaptive scaling was computed with in_log=true for numerical stability (no NaN/overflow values in the matrix)
- Inner bandwidth equals the median (50% quantile) of per-gene average pairwise squared distances returned by estimate_inner_bandwidth

## Limitations

- No algorithm is provided for selecting the two vital parameters, the non-Euclidean deformation q and the inner bandwidth phi.
- Runtime of evaluating the q-diffused kernel across all pairs of cells is suboptimal, taking 20 minutes to 3 hours for a single affinity matrix.
- The GPU path requires CUDA device IDs passed via cuda_devices whose array length matches the Julia thread count; without it, computation falls back to CPU-bound evaluation.

## Evidence

- [readme] It is recommended to wrap one of these in an `AdaptiveKernel` type that adjusts the base kernel's bandwidth relative to $k$-nn distances.: "It is recommended to wrap one of these in an `AdaptiveKernel` type that adjusts the base kernel's bandwidth relative to $k$-nn distances."
- [readme] kernel = AdaptiveKernel(16, # scale to the 16th nearest neighbor: "kernel = AdaptiveKernel(16, # scale to the 16th nearest neighbor"
- [methods] inner_bandwidth = estimate_inner_bandwidth(data): "inner_bandwidth = estimate_inner_bandwidth(data)"
- [results] the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells: "the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells"
