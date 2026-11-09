---
name: rank-constrained-matrix-optimization
description: Use when you have a raw single-cell expression matrix and need to learn
  cell-to-cell similarities robust to noise, outliers and dropouts, rather than using
  a fixed distance metric.
when_to_use_negative:
- The dataset is already a cell-to-cell similarity or feature table rather than an
  expression matrix
- A supervised analysis with pre-selected gene subsets of known function is required
  — SIMLR is unsupervised and supports de novo discovery only
- The goal is simply applying a standard dimension reduction method (e.g. PCA, t-SNE)
  to data whose statistical characteristics already fit standard assumptions
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: SIMLR
  role: Implements the multi-kernel similarity-learning framework; the SIMLR.R script
    runs the optimization loop combining kernels into the learned metric
  repo: github.com/BatzoglouLabSU/SIMLR
- name: R
  role: 'Runtime environment: Matrix package handles sparse expression matrices, parallel
    package parallelizes kernel estimation'
  repo: https://www.bioconductor.org/packages/release/bioc/html/SIMLR.html
- name: projsplx_R.c
  role: External C routine compiled with 'R CMD SHLIB -c projsplx_R.c' that performs
    the projection step inside the optimization loop
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:3fef33f5a421fa0c552703c0b59169cfc65a53258c3e1b33dc93d1b4b8e4d233
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - SIMLR
  - projsplx_R.c
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# rank-constrained-matrix-optimization

## Summary

Reconstructs SIMLR's optimization loop, which combines multiple Gaussian kernels into a learned sample-to-sample similarity matrix for single-cell RNA-seq data under a rank constraint, solved by a simple iterative procedure. It is the core similarity-learning step that enables downstream dimension reduction, clustering, and visualization of heterogeneous cell populations.

## When to use

Use when you have a raw single-cell expression matrix and need to learn cell-to-cell similarities robust to noise, outliers and dropouts, rather than using a fixed distance metric.

## When NOT to use

- The dataset is already a cell-to-cell similarity or feature table rather than an expression matrix
- A supervised analysis with pre-selected gene subsets of known function is required — SIMLR is unsupervised and supports de novo discovery only
- The goal is simply applying a standard dimension reduction method (e.g. PCA, t-SNE) to data whose statistical characteristics already fit standard assumptions

## Inputs

- Single-cell gene expression matrix loaded from an RData file (e.g. Test_1_mECS.RData), possibly sparse (handled via the Matrix package)
- Target number of cell populations C

## Outputs

- Learned sample-to-sample similarity matrix (e.g. written to simlr_similarity_matrix.csv) displaying an approximate block-diagonal structure with C blocks
- Learned distance metric usable for dimension reduction, clustering, and visualization

## How to apply

Load the expression matrix (e.g. Test_1_mECS.RData) in R using the Matrix package to handle sparsity, and set the target number of cell populations C. Compute multiple Gaussian kernel representations of cell-to-cell distances, parallelizing kernel estimation with the parallel package, since multiple kernels better fit the true statistical distribution of noisy scRNA-seq data. Execute the SIMLR optimization loop (SIMLR.R), which combines the kernels into a learned distance metric under the rank constraint, calling the external C routine projsplx_R.c — compiled via 'R CMD SHLIB -c projsplx_R.c' — for the projection step. The optimization is efficiently solved by a simple iterative procedure. Finally, apply graph diffusion to the learned similarity to strengthen weak cell-to-cell similarities caused by noise and dropout events, and write the similarity matrix to simlr_similarity_matrix.csv.

## Related tools

- **SIMLR** (Implements the multi-kernel similarity-learning framework; the SIMLR.R script runs the optimization loop combining kernels into the learned metric) — github.com/BatzoglouLabSU/SIMLR
- **R** (Runtime environment: Matrix package handles sparse expression matrices, parallel package parallelizes kernel estimation) — https://www.bioconductor.org/packages/release/bioc/html/SIMLR.html
- **projsplx_R.c** (External C routine compiled with 'R CMD SHLIB -c projsplx_R.c' that performs the projection step inside the optimization loop)

## Examples

```
R CMD SHLIB -c projsplx_R.c; Rscript R_main_demo_SIMLR.R
```

## Evaluation signals

- The resulting sample-to-sample similarity matrix displays an approximate block-diagonal structure with C blocks corresponding to the cell populations
- The optimization loop terminates via the simple iterative procedure without errors from the projsplx_R.c projection step
- SIMLR separates known subpopulations more accurately in single-cell data sets than existing dimension reduction methods (e.g. PCA, t-SNE)
- Kernel estimation completes across all parallel workers (parallel package) without sparse-matrix handling errors
- Downstream clustering on the learned similarities recovers the expected number of cell populations

## Limitations

- Single-cell data introduce high levels of noise, outliers and dropouts that weaken cell-to-cell similarities even under an appropriate distance metric; graph diffusion is required to mitigate this
- The example datasets (e.g. Test_1_mECS.RData) are provided purely as examples and after some pre-processing; they should not be used in place of the ones provided in the respective publications
- If there are issues compiling the .c file, remove the pre-compiled files (projsplx_R.o and projsplx_R.so) before recompiling
- The external C routine projsplx_R.c must be compiled before the R optimization loop can run

## Evidence

- [full_text] SIMLR combines multiple Gaussian kernels in an optimization framework, which can be efficiently solved by a simple iterative procedure.: [evidence span withheld — anchor status: not_found]
