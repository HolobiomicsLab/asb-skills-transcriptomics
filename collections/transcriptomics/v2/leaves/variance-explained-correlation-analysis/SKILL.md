---
name: variance-explained-correlation-analysis
description: Use when you have scRNAseq data with control genes (e.g. 629 annotated
  cell-cycle genes on 182 ESCs) plus known per-gene PVE and need to benchmark hidden-factor
  inference methods by r^2 against true PVE.
when_to_use_negative:
- When no control gene set (e.g. annotated cell-cycle genes) is available for the
  dataset, since scPLS jointly models a control set and a target set
- When the goal is to infer the biological factors (u_i) themselves rather than the
  biological effects, because of model identifiability scPLS can only infer effects
  (Lambda_u u_i)
- When the downstream analysis or question of interest differs (e.g. differential
  expression, lineage reconstruction), since the best confounding-removal method may
  vary and performance of scPLS in other settings was not evaluated
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scPLS
  role: Jointly infers cell-cycle confounding effects from target and control genes
    via partial least squares regression and outputs the corrected expression matrix
  repo: http://chenmengjie.github.io/Citrus/
- name: Citrus
  role: Toolkit providing the scPLS implementation and the CellCycleData dataset used
    for the ESC benchmark
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: Required dependency for installing and running Citrus (C++ backend with R
    wrappers)
- name: RcppArmadillo
  role: Required dependency for installing and running Citrus (C++ linear algebra
    backend)
- name: genlasso
  role: R package dependency of Citrus used by the scPLS penalty machinery
- name: gplots
  role: R package used to render the composite PCA comparison figure (pca_comparison.png)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:47124d13adeef64e62c036febb2ed31f175396176c7420ea294b9bd056c33ff2
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# variance-explained-correlation-analysis

## Summary

Evaluate how well a confounding-factor removal method (scPLS, scLVM, LMM, PCA) recovers true per-gene variance explained (PVE) by correlating each method's inferred factor against the true per-gene cell-cycle PVE on the 182-ESC dataset, comparing r^2 values across methods.

## When to use

Use when you have scRNAseq data with control genes (e.g. 629 annotated cell-cycle genes on 182 ESCs) plus known per-gene PVE and need to benchmark hidden-factor inference methods by r^2 against true PVE.

## When NOT to use

- When no control gene set (e.g. annotated cell-cycle genes) is available for the dataset, since scPLS jointly models a control set and a target set
- When the goal is to infer the biological factors (u_i) themselves rather than the biological effects, because of model identifiability scPLS can only infer effects (Lambda_u u_i)
- When the downstream analysis or question of interest differs (e.g. differential expression, lineage reconstruction), since the best confounding-removal method may vary and performance of scPLS in other settings was not evaluated

## Inputs

- CellCycleData dataset from the Citrus package: 182-ESC expression matrix (target genes)
- 629 annotated cell-cycle genes (control gene set)
- True per-gene cell-cycle PVE vector for correlation against inferred factors

## Outputs

- cellcycle_r2_results.csv with an r^2 column for all four methods (scPLS, scLVM, LMM, PCA)
- pca_comparison.png: composite PCA plots of uncorrected data and data corrected by scPLS, scLVM, LMM, PCA
- Per-method r^2 values quantifying correlation between inferred and true PVE (e.g. scPLS r2=0.94 vs r2=0.92 for scLVM/LMM/PCA)

## How to apply

Load the CellCycleData dataset from the Citrus package, extracting the 182-ESC target gene expression matrix and the 629 annotated cell-cycle genes as the control set. Run scPLS(Target, Control, ...) with the cell-cycle genes as Control to jointly infer the confounding effects via partial least squares regression, and fit the comparator methods scLVM, linear mixed model (LMM), and PCA on the same target/control configuration to obtain their inferred hidden factors. For each method, compute the coefficient of determination (r^2) between the inferred factor and the true per-gene cell-cycle PVE and write per-method values to cellcycle_r2_results.csv. Produce PCA plots of uncorrected and corrected data for all four methods and save a composite figure as pca_comparison.png with gplots. Correct application is judged when scPLS's r^2 with true cell-cycle PVE (expected ~0.94) is comparable to or better than scLVM, LMM, and PCA (each ~0.92) and is statistically significant.

## Related tools

- **scPLS** (Jointly infers cell-cycle confounding effects from target and control genes via partial least squares regression and outputs the corrected expression matrix) — http://chenmengjie.github.io/Citrus/
- **Citrus** (Toolkit providing the scPLS implementation and the CellCycleData dataset used for the ESC benchmark) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (Required dependency for installing and running Citrus (C++ backend with R wrappers))
- **RcppArmadillo** (Required dependency for installing and running Citrus (C++ linear algebra backend))
- **genlasso** (R package dependency of Citrus used by the scPLS penalty machinery)
- **gplots** (R package used to render the composite PCA comparison figure (pca_comparison.png))

## Examples

```
install_github("ChenMengjie/Citrus"); data(CellCycleData); fit <- scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)
```

## Evaluation signals

- cellcycle_r2_results.csv exists and contains an r^2 column with values for all four methods (scPLS, scLVM, LMM, PCA)
- scPLS's r^2 with true cell-cycle PVE is ~0.94, qualitatively comparable to or better than scLVM (0.92), LMM (0.92), and PCA (0.92)
- The correlation between inferred and true PVE for scPLS is statistically significant, not merely numerically higher
- pca_comparison.png renders and shows PCA plots of both uncorrected data and data corrected by each of the four methods
- The scPLS run completes using the documented signature scPLS(Target, Control, k1, k2, iter, ...) with the 629 cell-cycle genes supplied as Control

## Limitations

- Because of model identifiability, scPLS can only infer the biological effects (Lambda_u u_i), not the biological factors (u_i); direct inference of biological factors is not feasible in the current implementation
- The 'best' confounding effects removing method may vary depending on the downstream analysis or question of interest; scPLS performance in other analysis settings (e.g. differential expression, lineage reconstruction) was not evaluated
- No count-based model is provided; extending the framework to modeling count data is left as future work
- scPLS results here are on log-normalized expression from the Citrus toolkit, which is described as under development with clustering methods not yet fully distributed

## Evidence

- [abstract] By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects.: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects."
- [methods] scPLS signature: "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [methods] CellCycleData dataset: "- `CellCycleData (Citrus)`"
- [readme] README installation dependencies: "**Citrus** relies on the following R packages: **Rcpp**, **RcppArmadillo**, **genlasso**, **gplots**. All packagess are hosted on CRAN."
- [discussion] identifiability limitation: [evidence span withheld — anchor status: not_found]
