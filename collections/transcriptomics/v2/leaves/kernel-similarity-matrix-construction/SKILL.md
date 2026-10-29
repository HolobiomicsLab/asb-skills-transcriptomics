---
name: kernel-similarity-matrix-construction
description: Use when input is a raw single-cell RNA-seq expression matrix from heterogeneous
  samples and a cell-to-cell similarity measure is needed before clustering, t-SNE-style
  visualization, or estimating the number of clusters C.
when_to_use_negative:
- Input is already a cell-to-cell similarity/adjacency matrix rather than an expression
  matrix.
- The data are not heterogeneous single-cell samples and standard metrics (e.g., Euclidean/PCA)
  are known to suffice; SIMLR targets scRNA-seq noise and dropout structure.
- Only gene importance is needed without a similarity model — use SIMLR_Feature_Ranking
  on an already learned similarity instead of re-learning kernels.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: SIMLR
  role: learns the sample-to-sample similarity measure via multiple kernel learning
    from the expression matrix
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: R
  role: implementation language for SIMLR.R kernel/similarity learning
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: Matrix
  role: handles sparse matrices during kernel/similarity computation
  repo: https://cran.r-project.org/web/packages/Matrix/index.html
- name: parallel
  role: parallel implementation of the kernel estimation
  repo: https://stat.ethz.ch/R-manual/R-devel/library/parallel/doc/parallel.pdf
- name: SIMLR_Large_Scale
  role: large-scale variant for computing similarities on high-throughput datasets
    (PBMC 10x GemCode)
  repo: https://github.com/BatzoglouLabSU/SIMLR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:663449ea2bc9140a28aa90f3cb0d2d970c5c0a1a53569485dd0a59caeb89e252
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/BatzoglouLabSU/SIMLR
  tools:
  - Matrix
  - R
  - SIMLR
  - SIMLR_Large_Scale
  - igraph
  - parallel
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# kernel-similarity-matrix-construction

## Summary

Constructs a cell-to-cell (sample-to-sample) similarity matrix from single-cell RNA-seq expression data by learning a convex combination of multiple kernels, so the metric fits the data's statistical structure despite large noise and dropout. This similarity structure underpins SIMLR's dimension reduction, clustering, visualization, and cluster-number estimation.

## When to use

Use when input is a raw single-cell RNA-seq expression matrix from heterogeneous samples and a cell-to-cell similarity measure is needed before clustering, t-SNE-style visualization, or estimating the number of clusters C.

## When NOT to use

- Input is already a cell-to-cell similarity/adjacency matrix rather than an expression matrix.
- The data are not heterogeneous single-cell samples and standard metrics (e.g., Euclidean/PCA) are known to suffice; SIMLR targets scRNA-seq noise and dropout structure.
- Only gene importance is needed without a similarity model — use SIMLR_Feature_Ranking on an already learned similarity instead of re-learning kernels.

## Inputs

- single-cell RNA-seq gene expression matrix (e.g., .RData test datasets such as Test_1_mECS, Test_2_Kolod, Test_3_Pollen, Test_4_Usoskin, Test_5_Zeisel, or PBMC 10x Genomics GemCode data)

## Outputs

- learned sample-to-sample similarity matrix (multiple kernel combination over cells)
- kernel-embedded distance metric used for dimension reduction, clustering, and visualization

## How to apply

Load the scRNA-seq expression matrix (e.g., Test_3_Pollen.RData from the repository's data directory) into R. Compute the multiple kernel similarity representations with SIMLR's kernel estimation, using the parallel package for a parallel implementation of the kernel estimation and the Matrix package to handle sparse matrices. Combine the kernels so the learned distance metric best fits the structure of the data, since the diverse statistical characteristics due to large noise and dropout of single-cell data do not fit the assumptions of standard dimension reduction algorithms. Apply graph diffusion to refine weak similarity entries caused by noise or dropout events before downstream use. The resulting kernel-weighted similarity matrix feeds clustering, 2-D/3-D visualization, feature ranking via SIMLR_Feature_Ranking, or eigenspectrum-based estimation of the number of clusters; for large datasets switch to SIMLR_Large_Scale (requires Rcpp, pracma, RcppAnnoy, RSpectra).

## Related tools

- **SIMLR** (learns the sample-to-sample similarity measure via multiple kernel learning from the expression matrix) — https://github.com/BatzoglouLabSU/SIMLR
- **R** (implementation language for SIMLR.R kernel/similarity learning) — https://github.com/BatzoglouLabSU/SIMLR
- **Matrix** (handles sparse matrices during kernel/similarity computation) — https://cran.r-project.org/web/packages/Matrix/index.html
- **parallel** (parallel implementation of the kernel estimation) — https://stat.ethz.ch/R-manual/R-devel/library/parallel/doc/parallel.pdf
- **SIMLR_Large_Scale** (large-scale variant for computing similarities on high-throughput datasets (PBMC 10x GemCode)) — https://github.com/BatzoglouLabSU/SIMLR

## Examples

```
R CMD SHLIB -c projsplx_R.c; then in R: library("SIMLR"); load("data/Test_3_Pollen.RData"); res = SIMLR(X = Test_3_Pollen$in_x, c = 11, k = 10)
```

## Evaluation signals

- Downstream clustering separates known subpopulations more accurately than existing dimension reduction methods (e.g., PCA, t-SNE) on benchmark datasets like Test_3_Pollen.
- The similarity matrix is sparse (built via Matrix) and graph-diffused so weak similarities from dropout events are improved.
- The eigenspectrum of the learned kernel yields an eigenvalue-gap estimate of C that matches the reference subpopulation count of the validation dataset.
- The module runs to completion after compiling projsplx_R.c (R CMD SHLIB -c projsplx_R.c) with only Matrix and parallel required as core dependencies.
- For large-scale inputs, the SIMLR_Large_Scale variant completes on PBMC 10x Genomics GemCode data within the 4 additional packages (Rcpp, pracma, RcppAnnoy, RSpectra).

## Limitations

- High levels of noise, outliers and dropouts in single-cell data weaken cell-to-cell similarities even under an appropriate distance metric, requiring graph diffusion to recover them.
- Standard dimension reduction assumptions do not fit the diverse statistical characteristics of noisy scRNA-seq data, so a single fixed metric or kernel is unreliable.
- The bundled example datasets (Test_1_mECS ... Test_5_Zeisel) are pre-processed examples and should not be used in place of the ones provided in the respective publications.
- The external C code projsplx_R.c must be compiled (R CMD SHLIB -c projsplx_R.c); if compilation fails, remove pre-compiled files projsplx_R.o and projsplx_R.so (OS X pre-compiled file is provided).

## Evidence
