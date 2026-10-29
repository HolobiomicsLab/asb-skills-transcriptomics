---
name: voronoi-rexel-parcellization
description: Use when you have spatial single-cell data (e.g. MERFISH cell-by-gene
  matrix with (dims x cells) coordinates) and need cell-grouped spatial units for
  distribution-level segmentation such as LDS.
when_to_use_negative:
- Input is single-cell data without spatial coordinates — the Voronoi partition requires
  a (dims x cells) spatial matrix
- Goal is cell-level clustering or embedding (e.g. qPHATE, Leiden on cells) rather
  than region-level spatial segmentation
- Spatial modality has low resolution per spot (e.g. Visium), where individual rexels
  would not contain enough cells for meaningful distributions
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: q-Diffusion.jl
  role: Provides par_spatially_cluster_mmd with the VoronoiGrid partitioning method
    and n_pixels parameter used to build rexels from spatial coordinates
  repo: github.com/marmarelis/QDiffusion.jl
- name: MERFISH
  role: Spatial transcriptomics modality supplying the high-resolution (dims x cells)
    coordinates and cell-by-gene matrix that are parcellized into rexels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:9d237f438996da724faeb8e435c1d155130ff87031f99873ab4902866878583f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MERFISH
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# voronoi-rexel-parcellization

## Summary

Parcellizes spatial transcriptomics tissue samples into region-level pixels ('rexels') using an approximate Voronoi partitioning built by repeated k-means (VoronoiGrid with ~1000 attempts), so that downstream distributional methods (e.g. q-diffused LDS) can compare transcriptomic distributions across small tissue regions instead of individual cells.

## When to use

Use when you have spatial single-cell data (e.g. MERFISH cell-by-gene matrix with (dims x cells) coordinates) and need cell-grouped spatial units for distribution-level segmentation such as LDS.

## When NOT to use

- Input is single-cell data without spatial coordinates — the Voronoi partition requires a (dims x cells) spatial matrix
- Goal is cell-level clustering or embedding (e.g. qPHATE, Leiden on cells) rather than region-level spatial segmentation
- Spatial modality has low resolution per spot (e.g. Visium), where individual rexels would not contain enough cells for meaningful distributions

## Inputs

- Spatial coordinates matrix of shape (dims x cells) for each tissue sample
- Log-normalized cell-by-gene expression matrix (e.g. output of preprocess_cell_counts, *.normalized element)
- Kernel constructor (e.g. AdaptiveKernel wrapping DeformedKernel with q=1.2 and estimated inner bandwidth)

## Outputs

- Rexel partition of each tissue sample (~64 spatial regions per sample)
- Rexel-level transcriptomic distributions used for pairwise MMD distance computation and segmentation maps

## How to apply

Log-normalize the (genes x cells) count matrix first (e.g. via preprocess_cell_counts). Use par_spatially_cluster_mmd with n_pixels=64 to target ~64 rexels per sample and method=VoronoiGrid(1000), which runs ~1000 k-means attempts to find an even (approximate Voronoi) partition of the (dims x cells) spatial coordinates. Supply a kernel-construction callback over the points of each sample and the spatial coordinates matrix. The resulting rexels serve as the units for pairwise MMD distance matrices and subsequent Leiden or Ward hierarchical segmentation; keep k=16 neighborhood size for LDS on these small per-rexel cell groups.

## Related tools

- **q-Diffusion.jl** (Provides par_spatially_cluster_mmd with the VoronoiGrid partitioning method and n_pixels parameter used to build rexels from spatial coordinates) — github.com/marmarelis/QDiffusion.jl
- **MERFISH** (Spatial transcriptomics modality supplying the high-resolution (dims x cells) coordinates and cell-by-gene matrix that are parcellized into rexels)

## Examples

```
spatial = par_spatially_cluster_mmd(bandwidth, coordinates, data; n_pixels=64, method=VoronoiGrid(1000), make_psd=false, cuda_devices=[0]) do points; AdaptiveKernel(16, DeformedKernel(1.2f0, points, inner_bandwidth=estimate_inner_bandwidth(points)), points, in_log=true); end
```

## Evaluation signals

- Each sample yields approximately the target number of rexels (n_pixels=64) covering the tissue
- VoronoiGrid converges within its allotted attempts (1000) to an even partition of the spatial coordinates
- Downstream pairwise MMD distance matrix between rexels is computable for every rexel pair in the sample
- Subsequent Leiden/Ward segmentation on the rexel graph produces spatially contiguous, layer-like structures consistent with cortical anatomy (parallel stripes) in the MERFISH cortical samples

## Limitations

- Rexel-level comparisons rest on small per-rexel cell counts, which is why k=16 neighborhood size was used for LDS rather than larger k
- The partition is approximate (repeated k-means attempts), not an exact Voronoi tessellation; evenness depends on the number of attempts (1000)
- Runtime of the underlying kernel evaluations is suboptimal (20 min to 3 h for one affinity matrix), so parcellization scale should be kept modest

## Evidence

- [other] Parcellize each tissue sample into rexels (region-level pixels) using an approximate Voronoi partitioning built by repeated k-means: [evidence span withheld — anchor status: not_found]
- [readme] n_pixels=64, method=VoronoiGrid(1000): "n_pixels=64, # partition to 64 "pixels" and then classify
    method=VoronoiGrid(1000), # 1k attempts to find an even partitioning"
- [readme] spatial coordinates matrix (dims x cells): "Here, you need to supply another spatial `coordinates` matrix that is `(dims x cells)`."
- [results] k=16 neighborhood size for LDS: "the smaller samples in each rexel-rexel pairwise comparison justified a k = 16 neighborhood size for LDS."
