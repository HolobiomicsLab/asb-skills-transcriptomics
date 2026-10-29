---
name: partial-least-squares-latent-factor-modeling
description: Use when scRNA-seq expression must be corrected for hidden confounders
  (e.g. cell cycle, batch/technical effects) and both control genes and target genes
  are available to inform factor inference.
when_to_use_negative:
- When the goal is direct inference of biological factors (u_i) — by model identifiability
  scPLS can only infer biological effects, not the biological factors
- When a count-based model of the raw data is required — the article leaves extension
  to count models as future work
- When control genes are unavailable and no all-genes (PCA/LMM-style) configuration
  is acceptable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: scPLS
  role: 'Main implementation: scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH,
    g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size,
    center) fits the joint PLS model'
  repo: http://chenmengjie.github.io/Citrus/
- name: Citrus
  role: R toolkit (C++ with R wrappers) that hosts scPLS and provides the CellCycleData
    dataset
  repo: https://github.com/ChenMengjie/Citrus
- name: Rcpp
  role: Required dependency for Citrus's C++ implementation of scPLS
- name: RcppArmadillo
  role: Required dependency for Citrus's C++ implementation of scPLS
- name: genlasso
  role: Provides trend filtering used for residual de-noising of corrected expression
    residuals
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:95d197363eb43797053836b9f13fe3135622d6fb2d266784fab21dfc3f3a6048
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - genlasso
  - gplots
  - scPLS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# partial-least-squares-latent-factor-modeling

## Summary

Fit scPLS, a partial least squares latent factor model that jointly models a control gene set and a target gene set in single cell RNA-seq data to infer hidden confounding factors, using the paper's PLS-based initializer and a naive/block-wise EM algorithm.

## When to use

Use when scRNA-seq expression must be corrected for hidden confounders (e.g. cell cycle, batch/technical effects) and both control genes and target genes are available to inform factor inference.

## When NOT to use

- When the goal is direct inference of biological factors (u_i) — by model identifiability scPLS can only infer biological effects, not the biological factors
- When a count-based model of the raw data is required — the article leaves extension to count models as future work
- When control genes are unavailable and no all-genes (PCA/LMM-style) configuration is acceptable

## Inputs

- Target gene expression matrix (e.g. 182-ESC single-cell expression from CellCycleData in the Citrus package)
- Control gene expression matrix (e.g. 629 annotated cell-cycle genes)
- Candidate grids for the number of confounding factors (kz) and biological factors (ku)

## Outputs

- Fitted scPLS model object with estimated latent confounding factors, BIC-selected kz and ku, and EM convergence status (e.g. scpls_model_fit.rds)
- Corrected/denoised expression residuals after trend filtering
- Per-method r^2 values between inferred factors and true per-gene PVE (e.g. cellcycle_r2_results.csv)

## How to apply

Split the expression matrix into a Target gene matrix and a Control gene matrix (e.g. the 182-ESC matrix with 629 annotated cell-cycle genes from CellCycleData in Citrus). Initialize latent factors with the paper's Algorithm 1, which runs partial least squares regression on the two gene sets to obtain starting values for the kz confounding factors and ku biological factors. Fit the joint model (Equations 1-2, where confounders affect both gene sets and targets additionally carry biological factors) with the naive EM algorithm (Algorithm 2) until the convergence limit/tolerance criterion is met. Select kz and ku by BIC over a grid of candidate dimensions, then denoise residuals with trend filtering via genlasso. Rationale: joint modeling makes full use of the data to improve confounding inference and bridges PCA/LMM-type (all genes) and RUV/scLVM-type (control genes only) methods, with each as a special case.

## Related tools

- **scPLS** (Main implementation: scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center) fits the joint PLS model) — http://chenmengjie.github.io/Citrus/
- **Citrus** (R toolkit (C++ with R wrappers) that hosts scPLS and provides the CellCycleData dataset) — https://github.com/ChenMengjie/Citrus
- **Rcpp** (Required dependency for Citrus's C++ implementation of scPLS)
- **RcppArmadillo** (Required dependency for Citrus's C++ implementation of scPLS)
- **genlasso** (Provides trend filtering used for residual de-noising of corrected expression residuals)

## Examples

```
install_github("ChenMengjie/Citrus"); fit <- scPLS(Target, Control, k1=1, k2=1, iter=1000, limit=1e-5)
```

## Evaluation signals

- EM iteration terminates within the specified convergence limit/tolerance on the fixture without error
- The fitted object exposes the estimated confounding factors and the BIC-selected kz and ku dimensions
- r^2 between inferred factor and true per-gene cell-cycle PVE reaches ~0.94 for scPLS (paper reports r2=0.94, slightly higher and significant vs scLVM/LMM/PCA at r2=0.92)
- BIC selection across the candidate grid returns a finite optimum for both kz and ku
- PCA plots of corrected data show removal of the cell-cycle structure compared to uncorrected data

## Limitations

- scPLS can only infer biological effects (Lambda_u u_i) but not the biological factors (u_i) because of model identifiability
- The 'best' confounding-effect-removal method may vary depending on the downstream analysis; scPLS performance in other analysis settings has not been evaluated
- No count-based model is provided; extending the framework to modeling count data is left as future work

## Evidence

- [results] scPLS jointly models control and target genes via partial least squares regression: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects."
- [methods] scPLS function signature: "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [readme] Citrus dependencies: "**Citrus** relies on the following R packages: **Rcpp**, **RcppArmadillo**, **genlasso**, **gplots**."
- [discussion] identifiability limitation: [evidence span withheld — anchor status: not_found]
- [intro] scalability via EM: "with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes"
