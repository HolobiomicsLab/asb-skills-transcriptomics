---
name: cell-cycle-effect-removal
description: Decision support for removing cell-cycle effects from single-cell RNA-seq
  expression matrices when an annotated set of cell-cycle control genes is available
  and downstream analyses require corrected expression.
when_to_use_negative:
- When the goal is to infer the biological factors u_i themselves — scPLS can only
  infer biological effects, not the biological factors, due to model identifiability
- When downstream evaluation depends on analyses other than those evaluated in the
  paper (e.g. differential expression, lineage reconstruction), since performance
  of scPLS in other analysis settings was not evaluated
- When data are raw counts that require a count-based model — extending the scPLS
  framework to count data is left as future work
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Citrus
  role: R/C++ toolkit providing the CellCycleData dataset and the scPLS implementation
    used to remove cell-cycle effects
  repo: http://chenmengjie.github.io/Citrus/
- name: scPLS
  role: 'Core method: jointly models control (cell-cycle) and target genes via partial
    least squares regression to infer and remove confounding effects'
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: Required dependency for installing and running the C++-backed Citrus/scPLS
    package
- name: RcppArmadillo
  role: Required dependency for the linear-algebra backend of Citrus/scPLS
- name: gplots
  role: Used to render the composite PCA comparison figure of corrected vs uncorrected
    data
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:d025a2a4c55a91aeaf8438a2c108ebba73fb2393dc5fc1ac98b37b15ea0b7818
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - SC3
  - genlasso
  - gplots
  - scPLS
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# cell-cycle-effect-removal

## Summary

Removes cell-cycle confounding effects from scRNAseq data by jointly modeling a target gene matrix and an annotated cell-cycle control gene set with scPLS (partial least squares regression), and compares against scLVM, LMM, and PCA using per-gene PVE correlation.

## When to use

Decision support for removing cell-cycle effects from single-cell RNA-seq expression matrices when an annotated set of cell-cycle control genes is available and downstream analyses require corrected expression.

## When NOT to use

- When the goal is to infer the biological factors u_i themselves — scPLS can only infer biological effects, not the biological factors, due to model identifiability
- When downstream evaluation depends on analyses other than those evaluated in the paper (e.g. differential expression, lineage reconstruction), since performance of scPLS in other analysis settings was not evaluated
- When data are raw counts that require a count-based model — extending the scPLS framework to count data is left as future work

## Inputs

- Target gene expression matrix (e.g. 182-ESC matrix from CellCycleData in the Citrus package)
- Control gene set of 629 annotated cell-cycle genes from CellCycleData (Citrus)

## Outputs

- scPLS-corrected expression matrix with inferred cell-cycle confounding effects removed
- Inferred hidden factors from scPLS, scLVM, LMM, and PCA
- Per-method r^2 values between inferred factor and true per-gene cell-cycle PVE (e.g. cellcycle_r2_results.csv)
- Composite PCA comparison figure of uncorrected vs corrected data (e.g. pca_comparison.png)

## How to apply

Load the CellCycleData dataset from the Citrus package, extracting the 182-ESC expression matrix as the Target and the 629 annotated cell-cycle genes as the Control set. Run scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center) to jointly infer cell-cycle confounding effects and obtain the corrected expression matrix. Fit the comparison methods scLVM and linear mixed model (LMM) — control-gene-based — and PCA (all-gene-based) on the same configuration to obtain their inferred hidden factors. For each method, compute the coefficient of determination (r^2) between the inferred factor and the true per-gene cell-cycle PVE; in the paper scPLS achieved r2=0.94 vs r2=0.92 for scLVM, LMM, and PCA. Produce PCA plots of uncorrected and corrected data (using gplots) to visually confirm that culture/cell-cycle structure is captured and removed.

## Related tools

- **Citrus** (R/C++ toolkit providing the CellCycleData dataset and the scPLS implementation used to remove cell-cycle effects) — http://chenmengjie.github.io/Citrus/
- **scPLS** (Core method: jointly models control (cell-cycle) and target genes via partial least squares regression to infer and remove confounding effects) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (Required dependency for installing and running the C++-backed Citrus/scPLS package)
- **RcppArmadillo** (Required dependency for the linear-algebra backend of Citrus/scPLS)
- **gplots** (Used to render the composite PCA comparison figure of corrected vs uncorrected data)

## Examples

```
install.packages(c("Rcpp","RcppArmadillo","genlasso","gplots")); devtools::install_github("ChenMengjie/Citrus"); data(CellCycleData, package="Citrus"); fit <- scPLS(Target, Control)
```

## Evaluation signals

- Coefficient of determination r^2 between inferred factor and true per-gene cell-cycle PVE is >=0.92 for all methods, with scPLS reaching 0.94 and being slightly higher and statistically significant than comparators
- cellcycle_r2_results.csv exists and contains an r^2 value for all four methods (scPLS, scLVM, LMM, PCA)
- pca_comparison.png renders and shows PCA plots of uncorrected data and data corrected by each of the four methods
- PCA of the scPLS-corrected matrix is qualitatively comparable to or better than scLVM/LMM/PCA at removing cell-cycle structure

## Limitations

- scPLS can only infer biological effects (Lambda_u u_i) but not the biological factors (u_i) because of model identifiability
- The 'best' confounding-effect-removal method may vary depending on the downstream analysis; scPLS performance in other analysis settings has not been evaluated
- No count-based model is provided; extending the framework to modeling count data is left as future work

## Evidence

- [intro] By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding factors.: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding factors."
- [methods] scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center): "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [methods] CellCycleData (Citrus): "- `CellCycleData (Citrus)`"
- [discussion] because of model identifiability, scPLS can only be used to infer the biological effects (i.e.: "because of model identifiability, scPLS can only be used to infer the biological effects (i.e."
- [readme] Citrus relies on the following R packages: Rcpp, RcppArmadillo, genlasso, gplots.: "**Citrus** relies on the following R packages: **Rcpp**, **RcppArmadillo**, **genlasso**, **gplots**."
