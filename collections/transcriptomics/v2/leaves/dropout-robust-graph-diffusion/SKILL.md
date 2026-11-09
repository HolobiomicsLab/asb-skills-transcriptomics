---
name: dropout-robust-graph-diffusion
description: Use when scRNA-seq data contain high noise and dropout events that weaken
  cell-to-cell similarity measures, and an unsupervised distance metric is needed
  for clustering heterogeneous populations (e.g., PBMC subpopulations).
when_to_use_negative:
- The analysis requires pre-selected gene subsets of known function — SIMLR is explicitly
  unsupervised, allowing de novo discovery from the data.
- The data are not single-cell expression measurements with noise/dropout-driven weak
  similarities; the multi-kernel and graph-diffusion machinery targets scRNA-seq statistical
  characteristics.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SIMLR
  role: standard implementation that learns a multi-kernel distance metric, applies
    graph diffusion to dropout-weakened similarities, and performs clustering and
    2-D/3-D visualization
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: SIMLR_Large_Scale
  role: large-scale variant used on high-throughput datasets such as the PBMC GemCode
    matrix with hundreds of thousands of cells
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: R
  role: implementation language; the R scripts (SIMLR.R, SIMLR_Large_Scale.R) are
    run after compiling projsplx_R.c with R CMD SHLIB
  repo: https://github.com/BatzoglouLabSU/SIMLR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:e0608f6a8fbcc482e35c06cde4b2b7d64e94474c10543de595e3ce5a001d008d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/BatzoglouLabSU/SIMLR
  tools:
  - R
  - SIMLR
  - SIMLR_Large_Scale
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# dropout-robust-graph-diffusion

## Summary

A similarity-learning approach from SIMLR that combines multiple kernels and applies graph diffusion to strengthen cell-to-cell similarities weakened by noise and dropout events in single-cell RNA-seq data, supporting unsupervised dimension reduction, clustering, and visualization.

## When to use

Use when scRNA-seq data contain high noise and dropout events that weaken cell-to-cell similarity measures, and an unsupervised distance metric is needed for clustering heterogeneous populations (e.g., PBMC subpopulations).

## When NOT to use

- The analysis requires pre-selected gene subsets of known function — SIMLR is explicitly unsupervised, allowing de novo discovery from the data.
- The data are not single-cell expression measurements with noise/dropout-driven weak similarities; the multi-kernel and graph-diffusion machinery targets scRNA-seq statistical characteristics.

## Inputs

- single-cell RNA-seq gene expression matrix (e.g., PBMC GemCode/10x Genomics data)
- compiled external C routine projsplx_R.c (projsplx_R.so)
- R packages: Matrix, parallel; plus Rcpp, pracma, RcppAnnoy, RSpectra for large-scale runs

## Outputs

- learned cell-to-cell similarity/distance matrix from multi-kernel learning and graph diffusion
- per-cell unsupervised cluster assignments (e.g., simlr_pbmc_clusters.csv)
- 2-D (or 3-D) embedded coordinates for visualization of subpopulations

## How to apply

Load the scRNA-seq expression matrix (e.g., the high-throughput PBMC matrix from the GemCode single-cell technology from 10x Genomics) into R. Compile the external C routine with 'R CMD SHLIB -c projsplx_R.c' before running, removing pre-compiled projsplx_R.o/projsplx_R.so files if compilation issues occur. Run SIMLR (or SIMLR_Large_Scale for datasets with hundreds of thousands of cells) so it learns a distance metric by combining multiple kernels, then employs graph diffusion to improve weak similarity measures arising from noise or dropout events. Cluster cells from the learned similarity matrix without pre-selecting gene subsets, and apply SIMLR's dimension reduction to embed cells in 2-D for visualization.

## Related tools

- **SIMLR** (standard implementation that learns a multi-kernel distance metric, applies graph diffusion to dropout-weakened similarities, and performs clustering and 2-D/3-D visualization) — https://github.com/BatzoglouLabSU/SIMLR
- **SIMLR_Large_Scale** (large-scale variant used on high-throughput datasets such as the PBMC GemCode matrix with hundreds of thousands of cells) — https://github.com/BatzoglouLabSU/SIMLR
- **R** (implementation language; the R scripts (SIMLR.R, SIMLR_Large_Scale.R) are run after compiling projsplx_R.c with R CMD SHLIB) — https://github.com/BatzoglouLabSU/SIMLR

## Examples

```
R CMD SHLIB -c projsplx_R.c; Rscript R_main_demo_SIMLR_Large_Scale.R
```

## Evaluation signals

- Clustering separates known PBMC subpopulations more accurately than existing dimension reduction methods such as PCA and t-SNE, consistent with the reported high sensitivity and accuracy.
- Output file simlr_pbmc_clusters.csv exists and contains a cluster label and 2-D coordinates for every input cell.
- The similarity matrix is learned unsupervised, with no pre-selected gene subsets of known function.
- Normalized mutual information can be computed with the igraph package as in the repository demos (R_main_demo_SIMLR.R, R_main_demo_SIMLR_Large_Scale.R).
- The external C program compiled successfully (projsplx_R.so present after 'R CMD SHLIB -c projsplx_R.c').

## Limitations

- Requires compiling the external C file projsplx_R.c; if compilation issues occur, remove the pre-compiled projsplx_R.o and projsplx_R.so files (an OS X pre-compiled file is provided).
- Large-scale runs require four additional packages (Rcpp, pracma, RcppAnnoy, RSpectra) beyond Matrix and parallel.
- The example datasets in the repository (e.g., Test_1_mECS, Test_5_Zeisel) are pre-processed examples and should not be used in place of the data from the respective publications.

## Evidence
