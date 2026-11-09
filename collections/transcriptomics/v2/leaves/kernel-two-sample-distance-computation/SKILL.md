---
name: kernel-two-sample-distance-computation
description: Use when the analysis must compare spatial regions (rexels) or cell groups
  as transcriptome distributions rather than individual cells — e.g. segmenting MERFISH
  cortical tissue via LDS.
when_to_use_negative:
- The data is a plain single-cell expression matrix needing standard dimensionality
  reduction or clustering without a spatial/distributional comparison — use qPHATE
  or ordinary Leiden clustering instead.
- You need batch-effect correction or alignment of disparate datasets first — the
  paper explicitly did not consider batch-effect correction upstream of this analysis.
- The sample size per region is too small for stable within-set kernel averaging —
  q-diffusion also makes parametric assumptions that may be restrictive for some use
  cases.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: q-diffusion
  role: Provides the q-diffused kernel framework used to evaluate MMD between rexel
    transcriptome distributions for LDS
  repo: https://github.com/marmarelis/QDiffusion.jl
- name: QDiffusion.jl
  role: Julia package supplying DeformedKernel, AdaptiveKernel, estimate_inner_bandwidth,
    and par_spatially_cluster_mmd used to compute the pairwise distance matrices
  repo: https://github.com/marmarelis/QDiffusion.jl
- name: MERFISH
  role: Spatial scRNAseq modality whose human cortical tissue samples (cell-by-gene
    matrix plus coordinates) are segmented via the LDS MMD distances
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:34d676ea16758642a650f9311ced81e5d9b10784c50c20d7058b7ccc979f9f9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marmarelis/QDiffusion.jl
  tools:
  - MERFISH
  - QDiffusion.jl
  - q-diffusion
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# kernel-two-sample-distance-computation

## Summary

Computes pairwise distances between sets of cells (rexels or spatial regions) as distributional two-sample comparisons via maximum mean discrepancy (MMD) with a q-diffused kernel (or plain Gaussian baseline), enabling local distributional segmentation of spatial scRNAseq data.

## When to use

Use when the analysis must compare spatial regions (rexels) or cell groups as transcriptome distributions rather than individual cells — e.g. segmenting MERFISH cortical tissue via LDS.

## When NOT to use

- The data is a plain single-cell expression matrix needing standard dimensionality reduction or clustering without a spatial/distributional comparison — use qPHATE or ordinary Leiden clustering instead.
- You need batch-effect correction or alignment of disparate datasets first — the paper explicitly did not consider batch-effect correction upstream of this analysis.
- The sample size per region is too small for stable within-set kernel averaging — q-diffusion also makes parametric assumptions that may be restrictive for some use cases.

## Inputs

- log-normalized cell-by-gene expression matrix (genes x cells), e.g. from human cortical tissue MERFISH samples
- spatial coordinates matrix (dims x cells) for the cells
- rexel neighborhood assignments from approximate Voronoi parcellization via repeated k-means

## Outputs

- symmetric rexel-by-rexel pairwise MMD distance matrix for the q-diffused kernel (e.g. rexel_mmd_distances.csv)
- symmetric rexel-by-rexel pairwise MMD distance matrix for the Gaussian kernel baseline

## How to apply

Load the log-normalized cell-by-gene matrix (genes x cells) plus spatial coordinates, then build the q-diffused kernel by wrapping DeformedKernel (deformation parameter q=1.5 in the paper's setting) in an AdaptiveKernel scaled to the 16th nearest neighbor, with the inner bandwidth estimated by estimate_inner_bandwidth and log-space computation enabled for numerical stability. Run par_spatially_cluster_mmd with n_pixels=64 and method=VoronoiGrid(1000), make_psd=false, so MMD distances are averaged from within-set and between-set pairwise kernel evaluations across all rexel pairs (k=16 neighborhood size for LDS). Repeat with a plain GaussianKernel as the paper's comparison baseline, and assemble the symmetric rexel-by-rexel distance matrices from both kernel variants. Downstream, use these matrices for Leiden community detection or Ward-linkage hierarchical clustering with parameters optimized by grid search over silhouette scores.

## Related tools

- **q-diffusion** (Provides the q-diffused kernel framework used to evaluate MMD between rexel transcriptome distributions for LDS) — https://github.com/marmarelis/QDiffusion.jl
- **QDiffusion.jl** (Julia package supplying DeformedKernel, AdaptiveKernel, estimate_inner_bandwidth, and par_spatially_cluster_mmd used to compute the pairwise distance matrices) — https://github.com/marmarelis/QDiffusion.jl
- **MERFISH** (Spatial scRNAseq modality whose human cortical tissue samples (cell-by-gene matrix plus coordinates) are segmented via the LDS MMD distances)

## Examples

```
kernel = AdaptiveKernel(16, DeformedKernel(1.5f0, data, inner_bandwidth=estimate_inner_bandwidth(data)), data, in_log=true); D = par_spatially_cluster_mmd(bandwidth, coordinates, data; n_pixels=64, method=VoronoiGrid(1000), make_psd=false) do points kernel(points) end
```

## Evaluation signals

- Each output distance matrix must be square and symmetric.
- All pairwise distances must be nonnegative with a zero diagonal.
- The q-diffused variant must be usable downstream for silhouette-score-optimized segmentation, as reported in the paper.
- make_psd is left false, so verify the kernel-induced matrix does not need to be forced positive semidefinite for the intended downstream use.
- Compare the q-diffused matrix against the plain Gaussian kernel baseline matrix; both should be assembled from the same rexel pairs.

## Limitations

- Runtime of q-diffused kernel evaluation is suboptimal (20 min to 3 h for one affinity matrix), so GPU acceleration via cuda_devices is advisable.
- No algorithm is provided for selecting the deformation parameter q and inner bandwidth φ; the paper notes a careful selection algorithm is still needed.
- q-diffusion makes parametric assumptions on the data that might be restrictive in some use cases, and its nonlinearities are less directly interpretable in general.
- MMD matrix is computed with make_psd=false, so the resulting matrix is not guaranteed positive semidefinite.

## Evidence

- [results] we computed pairwise distance matrices between the small tissue regions termed rexels. These distances either came from the proposed local distributional segmentation (LDS) method: "we computed pairwise distance matrices between the small tissue regions termed rexels. These distances either came from the proposed local distributional segmentation (LDS) method"
- [results] the smaller samples in each rexel-rexel pairwise comparison justified a k = 16 neighborhood size for LDS. Finally the segmentation was performed by two popular algorithms: "the smaller samples in each rexel-rexel pairwise comparison justified a k = 16 neighborhood size for LDS. Finally the segmentation was performed by two popular algorithms"
- [readme] spatial = par_spatially_cluster_mmd(bandwidth, coordinates, data; n_pixels=64, method=VoronoiGrid(1000), make_psd=false, cuda_devices=[0]): "spatial = par_spatially_cluster_mmd(bandwidth, coordinates, data;
    n_pixels=64, # partition to 64 "pixels" and then classify
    method=VoronoiGrid(1000), # 1k attempts to find an even"
- [methods] inner_bandwidth = estimate_inner_bandwidth(data): "# pass in the same data matrix as you would to the kernel
inner_bandwidth = estimate_inner_bandwidth(data)"
- [abstract] a local distributional segmentation approach for spatial scRNAseq, driven by q-diffusion, yields interpretable structures of human cortical tissue: "a local distributional segmentation approach for spatial scRNAseq, driven by q-diffusion, yields interpretable structures of human cortical tissue"
