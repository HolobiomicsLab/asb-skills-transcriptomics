---
name: maximum-mean-discrepancy-estimation
description: Use when comparing groups of cells as whole transcriptomic distributions
  (e.g., rexels from MERFISH spatial scRNAseq parcellization) so that spatial regions
  can be clustered or segmented on distributional rather than per-cell distance.
when_to_use_negative:
- When comparing individual cells rather than distributions of cells — the kernel
  between all cell pairs, not set-wise MMD, is the appropriate operation
- When the spatial modality has low per-region cell counts such that within-set kernel
  averages are unreliable (the paper's rexels justify k=16 neighborhood size only
  for the smaller rexel-rexel comparisons)
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: q-Diffusion.jl
  role: Provides par_spatially_cluster_mmd, DeformedKernel, AdaptiveKernel, GaussianKernel,
    VoronoiGrid, and estimate_inner_bandwidth used to compute the MMD distance matrices
  repo: https://github.com/marmarelis/QDiffusion.jl
- name: MERFISH
  role: Spatial transcriptomics modality supplying the human cortical tissue cell-by-gene
    matrices and spatial coordinates that the rexel MMD distances are computed over
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:ffb2897e3afc4dc77a99c774c732d09db619561ec94d05bbb6469eea360d34de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marmarelis/QDiffusion.jl
  tools:
  - MERFISH
  - QDiffusion.jl
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# maximum-mean-discrepancy-estimation

## Summary

Compute pairwise maximum mean discrepancy (MMD) distances between sets of cells (rexels) using a q-diffused kernel, treating each spatial region as a transcriptomic distribution rather than individual cells, as used in the local distributional segmentation (LDS) approach for spatial scRNAseq.

## When to use

Use when comparing groups of cells as whole transcriptomic distributions (e.g., rexels from MERFISH spatial scRNAseq parcellization) so that spatial regions can be clustered or segmented on distributional rather than per-cell distance.

## When NOT to use

- When comparing individual cells rather than distributions of cells — the kernel between all cell pairs, not set-wise MMD, is the appropriate operation
- When the spatial modality has low per-region cell counts such that within-set kernel averages are unreliable (the paper's rexels justify k=16 neighborhood size only for the smaller rexel-rexel comparisons)

## Inputs

- Log-normalized cell-by-gene expression matrix (e.g., MERFISH human cortical tissue counts; README assumes a (genes x cells) matrix, usually log-normalized)
- Spatial coordinates matrix of shape (dims x cells) for the sample
- Kernel object (q-diffused DeformedKernel in AdaptiveKernel, or plain GaussianKernel as baseline)
- Estimated inner bandwidth from estimate_inner_bandwidth(data)

## Outputs

- Square, symmetric, nonnegative rexel-by-rexel pairwise MMD distance matrix with zero diagonal (one per kernel variant)
- Distance tables (e.g., rexel_mmd_distances.csv) usable for downstream segmentation by Leiden community detection or Ward-linkage hierarchical clustering

## How to apply

Parcellize the tissue into rexels (e.g., ~64 pixels per sample) using an approximate Voronoi partitioning via repeated k-means with VoronoiGrid(1000) attempts. Build a q-diffused kernel by wrapping DeformedKernel (deformation q, e.g., 1.2, with an inner bandwidth estimated as the 50% quantile of per-gene average pairwise squared distances via estimate_inner_bandwidth) in an AdaptiveKernel with k=16 neighbor scaling and in_log=true for numerical stability. Call par_spatially_cluster_mmd with n_pixels=64, method=VoronoiGrid(1000), and make_psd=false, so MMD distances between all rexel pairs are obtained by averaging within-set and between-set pairwise kernel evaluations. Also compute a plain GaussianKernel baseline since the paper experimented with both kernel variants for MMD. Assemble the symmetric rexel-by-rexel distance matrices and use them downstream (e.g., Leiden community detection or hierarchical clustering with Ward's linkage, parameters optimized by grid search over silhouette scores).

## Related tools

- **q-Diffusion.jl** (Provides par_spatially_cluster_mmd, DeformedKernel, AdaptiveKernel, GaussianKernel, VoronoiGrid, and estimate_inner_bandwidth used to compute the MMD distance matrices) — https://github.com/marmarelis/QDiffusion.jl
- **MERFISH** (Spatial transcriptomics modality supplying the human cortical tissue cell-by-gene matrices and spatial coordinates that the rexel MMD distances are computed over)

## Examples

```
spatial = par_spatially_cluster_mmd(bandwidth, coordinates, data; n_pixels=64, method=VoronoiGrid(1000), make_psd=false, cuda_devices=[0]) do points; AdaptiveKernel(16, DeformedKernel(1.2f0, points, inner_bandwidth=estimate_inner_bandwidth(points)), points, in_log=true); end
```

## Evaluation signals

- Each output distance matrix is square, symmetric, nonnegative, and has a zero diagonal
- The Gaussian-kernel baseline MMD matrix can be compared side-by-side with the q-diffused variant, as in the paper's dual-kernel experiment
- Downstream segmentation of the q-diffused MMD matrix should be usable with silhouette-score-optimized clustering (Leiden or Ward's linkage)
- On MERFISH human cortical tissue samples, q-diffused LDS segmentation should recover spatially contiguous, layer-like parallel stripe structure consistent with cortical functional layers

## Limitations

- Runtime of evaluating the q-diffused kernel is suboptimal (reported 20 min to 3 h for one affinity matrix), so MMD over many rexels benefits from GPU acceleration via cuda_devices
- No algorithm is provided for selecting the deformation parameter q and inner bandwidth φ; the paper notes such a selection algorithm is an open need
- make_psd is left false, so the kernel-induced distance matrix is not forced to be positive semidefinite
- No batch-effect correction or alignment of disparate datasets was considered in the study

## Evidence

- [results] q-diffused LDS pairwise MMD between rexels: "we computed pairwise distance matrices between the small tissue regions termed rexels. These distances either came from the proposed local distributional segmentation (LDS) method"
- [results] k=16 neighborhood size for LDS: "the smaller samples in each rexel-rexel pairwise comparison justified a k = 16 neighborhood size for LDS."
- [readme] LDS method entry point: "spatial = par_spatially_cluster_mmd(bandwidth, coordinates, data;
    n_pixels=64, # partition to 64 "pixels" and then classify
    method=VoronoiGrid(1000)"
- [readme] make_psd option: "make_psd=false, # don't force kernel-induced matrix to be p.s.d"
- [readme] kernel construction in log space: "in_log=true) # log-transform (necessary for numerical stability)"
- [results] silhouette grid search: "Clustering parameters were optimized by a grid search over the silhouette scores44 for each case."
