---
name: multi-kernel-similarity-learning
description: Use when input is a single-cell RNA-seq expression matrix with high noise
  and dropout events, and the goal is de novo, unsupervised clustering or 2-D/3-D
  visualization of cell subpopulations.
when_to_use_negative:
- Input data are provided purely as examples in the repository and should not be used
  in place of the ones from the respective publications.
- The analysis requires supervised classification using pre-selected gene subsets
  of known function — SIMLR is deliberately unsupervised for de novo discovery.
- Data fit the statistical assumptions of standard dimension reduction algorithms
  and lack the noise/dropout structure SIMLR is designed to address.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SIMLR
  role: core multi-kernel similarity learning implementation (SIMLR.R / SIMLR.m) that
    combines kernels, applies graph diffusion, and produces clustering and visualization
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: SIMLR_Large_Scale
  role: scalable variant for large datasets (e.g., Test_5_Zeisel, PBMC GemCode data)
    using Rcpp, pracma, RcppAnnoy, and RSpectra
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: R
  role: runtime environment; requires Matrix and parallel packages for the standard
    run, igraph for normalized mutual information, grDevices for plot coloring
- name: MATLAB
  role: alternative implementation (SIMLR.m, Matlab_main_demo_SIMLR.m)
  repo: https://github.com/BatzoglouLabSU/SIMLR
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_007
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:d81e278e93741da03519d0453ce0cc6b503b43994e9d8012874c69bd19910596
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/BatzoglouLabSU/SIMLR
  tools:
  - MATLAB
  - Matlab
  - R
  - SIMLR
  - SIMLR_Large_Scale
  - grDevices
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# multi-kernel-similarity-learning

## Summary

Learn a cell-to-cell similarity measure from single-cell RNA-seq data by combining multiple kernels and applying graph diffusion, as implemented in SIMLR, to support dimension reduction, clustering, and visualization of heterogeneous cell populations.

## When to use

Use when input is a single-cell RNA-seq expression matrix with high noise and dropout events, and the goal is de novo, unsupervised clustering or 2-D/3-D visualization of cell subpopulations.

## When NOT to use

- Input data are provided purely as examples in the repository and should not be used in place of the ones from the respective publications.
- The analysis requires supervised classification using pre-selected gene subsets of known function — SIMLR is deliberately unsupervised for de novo discovery.
- Data fit the statistical assumptions of standard dimension reduction algorithms and lack the noise/dropout structure SIMLR is designed to address.

## Inputs

- single-cell RNA-seq expression matrix (sparse or dense, loaded from .RData example datasets such as Test_1_mECS.RData, Test_2_Kolod.RData, Test_3_Pollen.RData, Test_4_Usoskin.RData, Test_5_Zeisel.RData)
- high-throughput PBMC expression matrix from 10x Genomics GemCode single-cell technology (for large-scale runs)

## Outputs

- learned cell-to-cell similarity matrix
- cluster assignment per cell
- 2-D and 3-D low-dimensional embedding coordinates (e.g., t-SNE-style visualization)
- normalized mutual information clustering scores for benchmarking

## How to apply

Load the scRNA-seq expression matrix (e.g., one of the example .RData datasets such as Test_1_mECS.RData or Test_5_Zeisel.RData) into R and run SIMLR.R, which learns a distance metric that best fits the data structure by combining multiple kernels, providing a better fit than standard dimension reduction algorithms to the diverse statistical characteristics of noisy scRNA-seq data. Because dropout events can significantly weaken cell-to-cell similarities even under an appropriate distance metric, SIMLR employs graph diffusion to improve weak similarity measures resulting from noise or dropouts. The run is unsupervised (no pre-selected gene subsets), enabling de novo discovery. For datasets beyond standard scale (e.g., hundreds of thousands of PBMC cells from 10x Genomics GemCode), use SIMLR_Large_Scale instead, after compiling the external C code with 'R CMD SHLIB -c projsplx_R.c' and installing Rcpp, pracma, RcppAnnoy, and RSpectra. Downstream, cluster cells from the learned similarity matrix and embed them in 2-D/3-D via stochastic neighbor embedding; the igraph package computes normalized mutual information against known labels for benchmarking against PCA and t-SNE baselines.

## Related tools

- **SIMLR** (core multi-kernel similarity learning implementation (SIMLR.R / SIMLR.m) that combines kernels, applies graph diffusion, and produces clustering and visualization) — https://github.com/BatzoglouLabSU/SIMLR
- **SIMLR_Large_Scale** (scalable variant for large datasets (e.g., Test_5_Zeisel, PBMC GemCode data) using Rcpp, pracma, RcppAnnoy, and RSpectra) — https://github.com/BatzoglouLabSU/SIMLR
- **R** (runtime environment; requires Matrix and parallel packages for the standard run, igraph for normalized mutual information, grDevices for plot coloring)
- **MATLAB** (alternative implementation (SIMLR.m, Matlab_main_demo_SIMLR.m)) — https://github.com/BatzoglouLabSU/SIMLR

## Examples

```
R CMD SHLIB -c projsplx_R.c; Rscript R_main_demo_SIMLR_Large_Scale.R
```

## Evaluation signals

- SIMLR should show higher normalized mutual information clustering scores than PCA (linear) and t-SNE (nonlinear) baselines on the four test datasets, computed with the igraph package as in the demo workflow.
- The recovered cell clusters should separate the known subpopulations of each example dataset (e.g., Zeisel clusters on Test_5_Zeisel.RData).
- The large-scale run on Test_5_Zeisel should complete within minutes on a single-core laptop without loss of accuracy relative to standard SIMLR.
- The external C code compiles successfully via 'R CMD SHLIB -c projsplx_R.c' (stale projsplx_R.o and projsplx_R.so removed if compilation issues occur).
- Embedding output contains 2-D (and optionally 3-D) coordinates for every input cell.

## Limitations

- The 5 example datasets in the repository are provided purely as examples after pre-processing and should not be used in place of the ones from the respective publications.
- Compilation of projsplx_R.c may fail; the fix is to remove stale pre-compiled files (projsplx_R.o and projsplx_R.so).
- Single-cell data introduce challenges to conventional similarity measures because of high levels of noise, outliers, and dropouts — methods that do not address dropout via graph diffusion weaken cell-to-cell similarities.
- The large-scale implementation requires 4 additional packages (Rcpp, pracma, RcppAnnoy, RSpectra) beyond the base Matrix and parallel requirements.

## Evidence
