---
name: multi-kernel-distance-metric-optimization
description: Use when a single-cell expression matrix (e.g., .RData workspace like
  Test_1_mECS.RData) must be converted into a cell-to-cell similarity matrix for clustering,
  dimension reduction, or visualization, especially under high noise and dropout.
when_to_use_negative:
- The samples are not single-cell expression data and a standard distance metric or
  linear method such as PCA already fits the data's statistical assumptions
- A gene subset with known function has been pre-selected and a supervised or feature-restricted
  analysis is required — SIMLR is unsupervised and allows only de novo discovery
- The goal is ranking features rather than learning sample-to-sample similarities
  — use SIMLR_Feature_Ranking instead
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SIMLR
  role: core implementation of the multi-kernel similarity-learning optimization (SIMLR.R
    / SIMLR.m) with graph diffusion post-processing
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: R
  role: runtime for SIMLR.R, the Matrix package for sparse inputs, the parallel package
    for parallelized kernel estimation, and R CMD SHLIB compilation of the C projection
    routine
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: SIMLR_Large_Scale
  role: variant implementation (SIMLR_Large_Scale.R / SIMLR_Large_Scale.m) of the
    same similarity-learning framework for large-scale datasets, e.g., Test_5_Zeisel.RData
  repo: https://github.com/BatzoglouLabSU/SIMLR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:1d3be5b47260eb40ef2d9f30c4db401689339c8fee28c792f8d957f68e06252b
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# multi-kernel-distance-metric-optimization

## Summary

SIMLR's similarity-learning procedure combines multiple Gaussian kernels of cell-to-cell distances into a single learned sample-to-sample distance metric via an iterative optimization with a rank constraint, then applies graph diffusion to repair weak similarities caused by noise and dropout events in single-cell RNA-seq data.

## When to use

Use when a single-cell expression matrix (e.g., .RData workspace like Test_1_mECS.RData) must be converted into a cell-to-cell similarity matrix for clustering, dimension reduction, or visualization, especially under high noise and dropout.

## When NOT to use

- The samples are not single-cell expression data and a standard distance metric or linear method such as PCA already fits the data's statistical assumptions
- A gene subset with known function has been pre-selected and a supervised or feature-restricted analysis is required — SIMLR is unsupervised and allows only de novo discovery
- The goal is ranking features rather than learning sample-to-sample similarities — use SIMLR_Feature_Ranking instead

## Inputs

- single-cell gene expression matrix (e.g., loaded from Test_1_mECS.RData, potentially sparse and handled via the R Matrix package)
- target number of cell populations C
- compiled projsplx_R.c shared object (or the pre-compiled OS X projsplx_R.so)

## Outputs

- learned sample-to-sample similarity matrix (e.g., written to simlr_similarity_matrix.csv) exhibiting approximate block-diagonal structure with C blocks
- learned distance metric usable for dimension reduction, clustering, and 2-D/3-D visualization

## How to apply

Load the sparse expression matrix in R using the Matrix package and set the target number of cell populations C. Compute multiple Gaussian kernel representations of cell-to-cell distances, parallelizing kernel estimation with the parallel package. Run the SIMLR optimization loop (SIMLR.R), which combines the kernels into a learned distance metric under a rank constraint and calls the external C routine projsplx_R.c (compiled with 'R CMD SHLIB -c projsplx_R.c') for the projection step. Finally, apply graph diffusion to the learned similarity, since weak similarity measures 'are likely to result from noise or dropout events', and optionally feed the result to dimension reduction, clustering, or SIMLR_Feature_Ranking. The rationale is that multiple kernels better fit the true statistical distribution of a specific scRNA-seq dataset than any single fixed metric.

## Related tools

- **SIMLR** (core implementation of the multi-kernel similarity-learning optimization (SIMLR.R / SIMLR.m) with graph diffusion post-processing) — https://github.com/BatzoglouLabSU/SIMLR
- **R** (runtime for SIMLR.R, the Matrix package for sparse inputs, the parallel package for parallelized kernel estimation, and R CMD SHLIB compilation of the C projection routine) — https://github.com/BatzoglouLabSU/SIMLR
- **SIMLR_Large_Scale** (variant implementation (SIMLR_Large_Scale.R / SIMLR_Large_Scale.m) of the same similarity-learning framework for large-scale datasets, e.g., Test_5_Zeisel.RData) — https://github.com/BatzoglouLabSU/SIMLR

## Examples

```
R CMD SHLIB -c projsplx_R.c && Rscript R_main_demo_SIMLR.R
```

## Evaluation signals

- The learned sample-to-sample similarity matrix displays an approximate block-diagonal structure with C blocks corresponding to the expected cell populations
- The optimization loop terminates without errors from the projsplx_R.c projection routine, and projsplx_R.o/projsplx_R.so compile cleanly (remove stale pre-compiled files if issues arise)
- Cluster assignments derived from the learned metric separate known subpopulations more accurately than existing dimension reduction methods (as reported for SIMLR on single-cell benchmark datasets such as Test_1_mECS through Test_4_Usoskin)
- Graph diffusion measurably strengthens weak cell-to-cell similarities that would otherwise be degraded by noise or dropout events
- Kernel estimation runs across parallel workers without failure, confirming the parallel package setup

## Limitations

- Single-cell data introduce high levels of noise, outliers and dropouts, which challenge conventional similarity measures and motivated the method but can still weaken individual cell-to-cell similarities
- Standard statistical assumptions made by dimension reduction algorithms may not fit the diverse statistical characteristics of scRNA-seq data, so a single fixed metric is unreliable — hence the multi-kernel design
- Compilation of projsplx_R.c can fail; the README advises removing pre-compiled files (projsplx_R.o and projsplx_R.so) if issues occur
- The bundled example datasets (Test_1_mECS.RData, Test_2_Kolod.RData, Test_3_Pollen.RData, Test_4_Usoskin.RData, Test_5_Zeisel.RData) are pre-processed examples and 'should not be used in place of the ones provided in the respective publications'

## Evidence

- [readme] required R libraries for the pipeline: [evidence span withheld — anchor status: not_found]
