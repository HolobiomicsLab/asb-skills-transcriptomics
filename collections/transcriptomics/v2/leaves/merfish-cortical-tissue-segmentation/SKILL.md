---
name: merfish-cortical-tissue-segmentation
description: Use when segmenting high-resolution spatial transcriptomics data (e.g.
  MERFISH human cortical tissue) into spatial regions by comparing neighborhoods as
  transcriptome distributions, not single cells.
when_to_use_negative:
- Input is non-spatial scRNAseq (e.g. PBMC or Tabula Sapiens atlas samples) where
  cells, not spatial neighborhoods, are the unit of comparison — use standard q-diffusion
  clustering or qPHATE instead.
- Modality has low spatial resolution (e.g. Visium), leaving too few cells per rexel
  for meaningful distributional MMD comparison.
- A per-cell embedding or cell-type annotation is the goal; LDS operates on rexel-level
  transcriptome distributions and does not classify individual cells.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: q-diffusion
  role: framework providing the q-diffused kernel whose MMD distances drive the LDS
    segmentation
  repo: github.com/marmarelis/QDiffusion.jl
- name: QDiffusion.jl
  role: Julia package implementing DeformedKernel/AdaptiveKernel and par_spatially_cluster_mmd
    for rexel-pair MMD computation
  repo: github.com/marmarelis/QDiffusion.jl
- name: MERFISH
  role: spatial transcriptomics modality supplying the human cortical tissue cell-by-gene
    matrix and spatial coordinates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:c42801616345fefae0e96f747801a025c7ed5cfd0b8a879350c065e771af32e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MERFISH
  - QDiffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# merfish-cortical-tissue-segmentation

## Summary

Segment spatial scRNAseq tissue (e.g. human cortical MERFISH) into interpretable structural regions via Local Distributional Segmentation (LDS), which compares rexels as distributions of transcriptomes using q-diffused-kernel MMD distances rather than per-cell comparisons.

## When to use

Use when segmenting high-resolution spatial transcriptomics data (e.g. MERFISH human cortical tissue) into spatial regions by comparing neighborhoods as transcriptome distributions, not single cells.

## When NOT to use

- Input is non-spatial scRNAseq (e.g. PBMC or Tabula Sapiens atlas samples) where cells, not spatial neighborhoods, are the unit of comparison — use standard q-diffusion clustering or qPHATE instead.
- Modality has low spatial resolution (e.g. Visium), leaving too few cells per rexel for meaningful distributional MMD comparison.
- A per-cell embedding or cell-type annotation is the goal; LDS operates on rexel-level transcriptome distributions and does not classify individual cells.

## Inputs

- log-normalized cell-by-gene expression matrix for MERFISH human cortical tissue samples
- (dims x cells) spatial coordinates matrix for each cell
- rexel neighborhood assignments from approximate Voronoi parcellization (repeated k-means)
- kernel specification: DeformedKernel (with q and inner bandwidth) or GaussianKernel, wrapped in AdaptiveKernel

## Outputs

- symmetric rexel-by-rexel pairwise MMD distance matrix (q-diffused kernel variant)
- symmetric rexel-by-rexel pairwise MMD distance matrix (Gaussian kernel baseline)
- spatial segmentation labels of tissue regions (via Leiden community detection or hierarchical clustering with Ward's linkage)
- silhouette-score grid-search results used to optimize clustering parameters

## How to apply

Log-normalize the cell-by-gene expression matrix and supply the (dims x cells) spatial coordinates from approximate Voronoi parcellization via repeated k-means. Build a DeformedKernel (paper's deformation parameter q, e.g. q=1.5) wrapped in an AdaptiveKernel (16-nn bandwidth scaling), estimating the inner bandwidth with estimate_inner_bandwidth (median quantile of per-gene average pairwise squared distances) and enabling in_log=true for numerical stability. Call par_spatially_cluster_mmd with n_pixels=64, method=VoronoiGrid(1000), make_psd=false to compute pairwise MMD distances between rexels by averaging within-set and between-set kernel evaluations; repeat with a plain GaussianKernel as the comparison baseline. Assemble the symmetric rexel-by-rexel distance matrices and feed them downstream to Leiden community detection or Ward-linkage hierarchical clustering with k=16 neighborhood size, optimizing clustering parameters by grid search over silhouette scores.

## Related tools

- **q-diffusion** (framework providing the q-diffused kernel whose MMD distances drive the LDS segmentation) — github.com/marmarelis/QDiffusion.jl
- **QDiffusion.jl** (Julia package implementing DeformedKernel/AdaptiveKernel and par_spatially_cluster_mmd for rexel-pair MMD computation) — github.com/marmarelis/QDiffusion.jl
- **MERFISH** (spatial transcriptomics modality supplying the human cortical tissue cell-by-gene matrix and spatial coordinates)

## Examples

```
kernel = AdaptiveKernel(16, DeformedKernel(1.5f0, data, inner_bandwidth=estimate_inner_bandwidth(data)), data, in_log=true); spatial = par_spatially_cluster_mmd(bandwidth, coordinates, data; n_pixels=64, method=VoronoiGrid(1000), make_psd=false) do pts; DeformedKernel(1.5f0, pts); end
```

## Evaluation signals

- Each output rexel distance matrix is square, symmetric, and nonnegative with zero diagonal.
- Segmentation downstream succeeds with silhouette-score-optimized clustering, as reported for the q-diffused variant in the paper.
- The q-diffused-kernel MMD matrix is compared against the Gaussian-kernel baseline matrix; both are computed over the same rexel pairs.
- Voronoi parcellization attempts to find an even partition (VoronoiGrid(1000)) so rexels contain comparable numbers of cells (n_pixels=64).
- Kernel computation runs in log space (in_log=true) without numerical instability artifacts.

## Limitations

- No algorithm is provided for selecting the deformation parameter q and inner bandwidth phi; parameter choice requires manual exploration.
- Runtime of the q-diffused kernel evaluation is suboptimal (20 min to 3 h for one affinity matrix); GPU acceleration via cuda_devices is recommended.
- make_psd=false means the kernel-induced distance matrix is not forced to be positive semi-definite.
- No batch-effect correction or alignment of disparate spatial datasets was considered in the study.

## Evidence

- [results] we computed pairwise distance matrices between the small tissue regions termed rexels. These distances either came from the proposed local distributional segmentation (LDS) method: "we computed pairwise distance matrices between the small tissue regions termed rexels. These distances either came from the proposed local distributional segmentation (LDS) method"
- [results] the smaller samples in each rexel-rexel pairwise comparison justified a k = 16 neighborhood size for LDS. Finally the segmentation was performed by two popular algorithms: "the smaller samples in each rexel-rexel pairwise comparison justified a k = 16 neighborhood size for LDS. Finally the segmentation was performed by two popular algorithms"
- [readme] spatial = par_spatially_cluster_mmd(bandwidth, coordinates, data; n_pixels=64, method=VoronoiGrid(1000), make_psd=false: "spatial = par_spatially_cluster_mmd(bandwidth, coordinates, data;
    n_pixels=64, # partition to 64 "pixels" and then classify
    method=VoronoiGrid(1000), # 1k attempts to find an even"
- [readme] A novel method for segmentation of spatial transcriptomics from modalities like MERFISH. Here, you need to supply another spatial coordinates matrix: "A novel method for segmentation of spatial transcriptomics from modalities like MERFISH.
Here, you need to supply another spatial `coordinates` matrix that is `(dims x cells)`."
- [abstract] a local distributional segmentation approach for spatial scRNAseq, driven by q-diffusion, yields interpretable structures of human cortical tissue: "a local distributional segmentation approach for spatial scRNAseq, driven by q-diffusion, yields interpretable structures of human cortical tissue"
- [results] Clustering parameters were optimized by a grid search over the silhouette scores: "Clustering parameters were optimized by a grid search over the silhouette scores44 for each case."
