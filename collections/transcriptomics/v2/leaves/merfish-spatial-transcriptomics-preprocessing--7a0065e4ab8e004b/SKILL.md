---
name: merfish-spatial-transcriptomics-preprocessing
description: Use when the input is a spatial scRNAseq/MERFISH sample with a cell-by-gene
  count matrix plus (dims x cells) spatial coordinates, and the goal is to segment
  tissue into region-level pixels (rexels) before distributional comparison.
when_to_use_negative:
- Input is already reduced (feature-selected, dimensionality-reduced, or marker-gene-filtered)
  — q-diffusion/LDS operates on the full log-normalized transcriptome
- Data has no spatial coordinates, so rexel parcellization and LDS are impossible
  (plain scRNAseq should go through standard q-diffusion clustering/qPHATE instead)
- Batch-effect correction or alignment of disparate datasets is required — the paper
  explicitly did not consider batch-effect correction or alignment
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: q-Diffusion.jl
  role: Provides preprocess_cell_counts for log-normalization, estimate_inner_bandwidth
    for the inner-bandwidth heuristic, and par_spatially_cluster_mmd with VoronoiGrid(1000)
    for parcellizing samples into ~64 rexels using the (dims x cells) coordinates
  repo: github.com/marmarelis/QDiffusion.jl
- name: MERFISH
  role: Source modality providing the ten human cortical tissue samples, each as a
    cell-by-gene matrix with spatial coordinates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:56b646af8ebc21b53ab5338c836299ec98191373c23e969635b72a3f5a8419f3
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
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# merfish-spatial-transcriptomics-preprocessing

## Summary

Preprocesses spatial transcriptomics data (e.g., MERFISH human cortical tissue samples) for local distributional segmentation (LDS): log-normalizing the cell-by-gene matrix, pairing it with its (dims x cells) spatial coordinate matrix, and parcellizing each tissue sample into ~64 rexels via an approximate Voronoi partition built by repeated k-means.

## When to use

Use when the input is a spatial scRNAseq/MERFISH sample with a cell-by-gene count matrix plus (dims x cells) spatial coordinates, and the goal is to segment tissue into region-level pixels (rexels) before distributional comparison.

## When NOT to use

- Input is already reduced (feature-selected, dimensionality-reduced, or marker-gene-filtered) — q-diffusion/LDS operates on the full log-normalized transcriptome
- Data has no spatial coordinates, so rexel parcellization and LDS are impossible (plain scRNAseq should go through standard q-diffusion clustering/qPHATE instead)
- Batch-effect correction or alignment of disparate datasets is required — the paper explicitly did not consider batch-effect correction or alignment

## Inputs

- MERFISH cell-by-gene count matrix (genes x cells) per tissue sample
- Spatial coordinates matrix (dims x cells) aligned with the count matrix
- Number of target rexels per sample (e.g., 64) and VoronoiGrid attempt count (e.g., 1000)

## Outputs

- Log-normalized gene-by-cell expression matrix (no feature selection or dimensionality reduction applied)
- Rexel partition of each tissue sample (~64 spatial regions per sample)
- Rexel-to-cell assignment mapping for downstream pairwise MMD distance computation

## How to apply

Load the (genes x cells) count matrix and the parallel (dims x cells) spatial coordinate matrix for each tissue sample, then log-normalize expression (the QDiffusion.jl convenience method preprocess_cell_counts(data) returns a named tuple whose .normalized element is log-normalized, sans feature selection or dimensionality reduction). Parcellize each sample into approximately 64 rexels using an approximate Voronoi partitioning built by repeated k-means — in QDiffusion.jl this is the VoronoiGrid(1000) configuration, i.e., 1000 attempts to find an even partitioning, passed via n_pixels=64 and method=VoronoiGrid(1000) to par_spatially_cluster_mmd. Keep the spatial coordinates matrix separate from the transcriptomic matrix, since downstream LDS computes pairwise MMD distances between rexels purely from transcriptomic distributions, using coordinates only to define the partition. For later kernel steps, estimate the inner bandwidth from the data via estimate_inner_bandwidth (the paper's heuristic: 50% quantile of per-gene average pairwise squared distances across cells).

## Related tools

- **q-Diffusion.jl** (Provides preprocess_cell_counts for log-normalization, estimate_inner_bandwidth for the inner-bandwidth heuristic, and par_spatially_cluster_mmd with VoronoiGrid(1000) for parcellizing samples into ~64 rexels using the (dims x cells) coordinates) — github.com/marmarelis/QDiffusion.jl
- **MERFISH** (Source modality providing the ten human cortical tissue samples, each as a cell-by-gene matrix with spatial coordinates)

## Examples

```
data_tuple = preprocess_cell_counts(raw_counts); spatial = par_spatially_cluster_mmd(bandwidth, coordinates, data_tuple.normalized; n_pixels=64, method=VoronoiGrid(1000), cuda_devices=[0]) do points kernel(points) end
```

## Evaluation signals

- preprocess_cell_counts(data).normalized returns the log-normalized matrix with no feature selection or dimensionality reduction applied
- Each sample is partitioned into ~64 rexels, with VoronoiGrid running 1000 attempts to find an even partitioning
- The spatial coordinates matrix has shape (dims x cells) and matches the cell order of the count matrix
- Downstream rexel-rexel pairwise MMD distances can be computed with a k=16 neighborhood size, which the paper justified by the small sample sizes within each rexel pair
- Segmentation maps of q-diffused vs. Gaussian MMD distances reproduce cortical-layer-like parallel stripe structure in more than half of the ten human cortical tissue samples

## Limitations

- Parcellization is approximate: VoronoiGrid uses repeated k-means (1000 attempts) rather than an exact even partitioning of cells into rexels
- Runtime of downstream q-diffused kernel evaluation is suboptimal (20 min to 3 h for one affinity matrix), so rexel/cell scale matters
- No algorithm is provided for selecting the q-diffusion parameters q and inner bandwidth φ, and no batch-effect correction or dataset alignment is performed

## Evidence

- [readme] Convenience preprocessing method returning log-normalized matrix: "There is also a convenience mathod called `preprocess_cell_counts(data)` that returns a named tuple containing different versions of the matrix"
- [readme] LDS requires a spatial coordinates matrix of shape (dims x cells): "Here, you need to supply another spatial `coordinates` matrix that is `(dims x cells)`."
- [readme] Rexel parcellization parameters for spatial clustering: "n_pixels=64, # partition to 64 "pixels" and then classify, method=VoronoiGrid(1000), # 1k attempts to find an even partitioning"
- [results] Pairwise distance matrices computed between rexels: "we computed pairwise distance matrices between the small tissue regions termed rexels"
- [results] Justification for k=16 neighborhood size in LDS: "the smaller samples in each rexel-rexel pairwise comparison justified a k = 16 neighborhood size for LDS"
