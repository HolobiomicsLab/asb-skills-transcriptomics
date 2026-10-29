---
name: spatial-transcriptomics-rexel-parcellation
description: Use when you have spatial scRNAseq data (e.g., MERFISH cell coordinates
  + log-normalized cell-by-gene matrix) and need to segment tissue into spatial regions
  by comparing neighborhoods as transcriptome distributions rather than single cells.
when_to_use_negative:
- The data are non-spatial scRNAseq with no cell coordinates — par_spatially_cluster_mmd
  requires a (dims x cells) spatial coordinates matrix
- The goal is single-cell-level clustering rather than distributional comparison of
  tissue regions — the MMD/LDS approach compares rexels as transcriptome distributions,
  not individual cells
- Datasets are much larger than ~10,000 cells per tissue region; the paper's LDS setting
  (k=16, n_pixels=64) was tuned for small tissue samples
edam_operation: http://edamontology.org/operation_3432
tools:
- name: q-Diffusion.jl
  role: Julia package providing par_spatially_cluster_mmd, VoronoiGrid, DeformedKernel/AdaptiveKernel/GaussianKernel,
    and estimate_inner_bandwidth for computing rexel MMD distances
  repo: github.com/marmarelis/QDiffusion.jl
- name: q-diffusion
  role: Framework whose q-diffused kernel drives the local distributional segmentation
    of spatial transcriptomics
  repo: github.com/marmarelis/QDiffusion.jl
- name: MERFISH
  role: Spatial transcriptomics modality supplying the human cortical tissue samples
    (coordinates + expression) segmented by LDS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:a2d4d5338af540bcad945903d29e92325758a89153f7fd77d04a1dcc4b8c350e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MERFISH
  - QDiffusion.jl
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# spatial-transcriptomics-rexel-parcellation

## Summary

Segments spatial scRNAseq tissue (e.g., human cortical MERFISH) into local neighborhoods ('rexels') via approximate Voronoi parcellization and compares them as distributions of transcriptomes using MMD distances under the q-diffused kernel, enabling local distributional segmentation (LDS).

## When to use

Use when you have spatial scRNAseq data (e.g., MERFISH cell coordinates + log-normalized cell-by-gene matrix) and need to segment tissue into spatial regions by comparing neighborhoods as transcriptome distributions rather than single cells.

## When NOT to use

- The data are non-spatial scRNAseq with no cell coordinates — par_spatially_cluster_mmd requires a (dims x cells) spatial coordinates matrix
- The goal is single-cell-level clustering rather than distributional comparison of tissue regions — the MMD/LDS approach compares rexels as transcriptome distributions, not individual cells
- Datasets are much larger than ~10,000 cells per tissue region; the paper's LDS setting (k=16, n_pixels=64) was tuned for small tissue samples

## Inputs

- Log-normalized cell-by-gene expression matrix for spatial scRNAseq samples (e.g., human cortical tissue MERFISH, 10 samples)
- Spatial coordinates matrix of shape (dims x cells) aligned with the expression matrix
- Deformation parameter q (paper's setting, e.g., q=1.5) and inner bandwidth (or the data to estimate it)
- Optional: GPU CUDA device IDs for acceleration

## Outputs

- Rexel assignments per cell (approximate Voronoi parcellization, e.g., 64 pixels)
- Symmetric square rexel-by-rexel pairwise MMD distance matrix (q-diffused variant, zero diagonal, nonnegative), e.g., rexel_mmd_distances.csv
- Baseline Gaussian-kernel rexel MMD distance matrix for comparison
- Segmented tissue map / region labels after silhouette-optimized Leiden or Ward clustering

## How to apply

Parcellize the spatial domain into rexels by approximate Voronoi partitioning (repeated k-means, e.g., n_pixels=64 with method=VoronoiGrid(1000)) using the cell spatial coordinates matrix. For each rexel-rexel pair, compute the maximum mean discrepancy (MMD) by averaging within-set and between-set pairwise kernel evaluations, using a q-diffused kernel (DeformedKernel, q≈1.5) wrapped in an AdaptiveKernel(k=16) whose inner bandwidth is estimated as the median quantile of per-gene average pairwise squared distances via estimate_inner_bandwidth, with in_log=true for numerical stability. Repeat with a plain GaussianKernel as a baseline comparison since the paper experimented with both kernels for MMD. Pass make_psd=false so the kernel-induced distance matrix is not forced positive semi-definite, and set the adaptive bandwidth scale to k=16 because 'the smaller samples in each rexel-rexel pairwise comparison justified a k = 16 neighborhood size for LDS.' Downstream, segment the rexels from the symmetric distance matrix via Leiden community detection or hierarchical clustering with Ward's linkage, choosing the number of segments by grid search over silhouette scores.

## Related tools

- **q-Diffusion.jl** (Julia package providing par_spatially_cluster_mmd, VoronoiGrid, DeformedKernel/AdaptiveKernel/GaussianKernel, and estimate_inner_bandwidth for computing rexel MMD distances) — github.com/marmarelis/QDiffusion.jl
- **q-diffusion** (Framework whose q-diffused kernel drives the local distributional segmentation of spatial transcriptomics) — github.com/marmarelis/QDiffusion.jl
- **MERFISH** (Spatial transcriptomics modality supplying the human cortical tissue samples (coordinates + expression) segmented by LDS)

## Examples

```
spatial = par_spatially_cluster_mmd(bandwidth, coordinates, data; n_pixels=64, method=VoronoiGrid(1000), make_psd=false, cuda_devices=[0]) do points; AdaptiveKernel(16, DeformedKernel(1.5f0, points, inner_bandwidth=1.125f0), points); end
```

## Evaluation signals

- Each output distance matrix is square, symmetric, and nonnegative with zero diagonal
- Parcellization produced the expected number of rexels (e.g., n_pixels=64) via VoronoiGrid attempts at an even partition
- MMD distances computed with the q-diffused kernel support downstream segmentation whose clustering parameters are validated by grid search over silhouette scores
- Gaussian-kernel baseline distance matrix is comparable in shape/scale to the q-diffused variant, since the paper experimented with both kernels for MMD
- Resulting segmentation of human cortical tissue yields interpretable structural regions (e.g., cortical layers) as reported in the paper

## Limitations

- make_psd=false means the kernel-induced rexel distance matrix is not guaranteed to be positive semi-definite
- The q-diffused kernel evaluation is slow — the paper reports 20 min to 3 h for one full cell-by-cell affinity matrix
- No algorithm is provided for selecting the deformation parameter q and inner bandwidth φ; the paper notes this as an open limitation
- No batch-effect correction or alignment of disparate datasets was considered in the LDS analysis

## Evidence

- [readme] LDS pairs rexels via MMD: "A novel method for segmentation of spatial transcriptomics from modalities like MERFISH. Here, you need to supply another spatial `coordinates` matrix that is `(dims x cells)`."
- [readme] par_spatially_cluster_mmd parameters: "spatial = par_spatially_cluster_mmd(bandwidth, coordinates, data;
    n_pixels=64, # partition to 64 "pixels" and then classify
    method=VoronoiGrid(1000)"
- [results] Rexel pairwise distance matrices from LDS: "we computed pairwise distance matrices between the small tissue regions termed rexels. These distances either came from the proposed local distributional segmentation (LDS) method"
- [results] k=16 neighborhood size rationale: "the smaller samples in each rexel-rexel pairwise comparison justified a k = 16 neighborhood size for LDS. Finally the segmentation was performed by two popular algorithms42: Leiden community"
- [methods] Inner bandwidth estimation: "inner_bandwidth = estimate_inner_bandwidth(data)"
- [results] Silhouette-based parameter optimization: "Clustering parameters were optimized by a grid search over the silhouette scores44 for each case."
