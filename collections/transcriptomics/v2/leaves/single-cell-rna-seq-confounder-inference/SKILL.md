---
name: single-cell-rna-seq-confounder-inference
description: Use when scRNA-seq expression matrices contain unmeasured confounders
  (e.g. cell cycle, batch) and a set of annotated control genes is available; benchmark
  against PCA, LMM, RUV, scLVM.
when_to_use_negative:
- The downstream goal requires direct inference of biological factors u_i — the paper
  states that 'because of model identifiability, scPLS can only be used to infer the
  biological effects ... but not the biological factors'
- The analysis goal is clustering-independent downstream evaluation (differential
  expression, lineage reconstruction) that the paper did not evaluate
- Data are raw counts requiring a count-based noise model — the authors note that
  'extending our framework to modeling count data is another promising avenue for
  future research'
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: Citrus
  role: Toolkit hosting scPLS and the CellCycleData/Pollen datasets; provides the
    scPLS(Target, Control, ...) entry point
  repo: http://chenmengjie.github.io/Citrus/
- name: scPLS
  role: Joint PLS-regression inference of confounding effects from control and target
    genes via block-wise EM
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: C++ backend required to install Citrus and run the scPLS EM implementation
- name: RcppArmadillo
  role: Linear-algebra backend for the scPLS block-wise EM algorithm
- name: genlasso
  role: CRAN dependency of Citrus used for penalized fitting in scPLS
- name: gplots
  role: R package used to render the composite PCA comparison figure of corrected
    vs uncorrected data
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:0a8ecd283a3af9c33f456bc2808c323d3ac1d613fbccf20fe95810d08ec1bf2f
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# single-cell-rna-seq-confounder-inference

## Summary

Infer and remove hidden confounding effects (technical or cell-cycle) in single-cell RNA-seq data using scPLS, which jointly models a control gene set and a target gene set via partial least squares regression with a block-wise EM algorithm.

## When to use

Use when scRNA-seq expression matrices contain unmeasured confounders (e.g. cell cycle, batch) and a set of annotated control genes is available; benchmark against PCA, LMM, RUV, scLVM.

## When NOT to use

- The downstream goal requires direct inference of biological factors u_i — the paper states that 'because of model identifiability, scPLS can only be used to infer the biological effects ... but not the biological factors'
- The analysis goal is clustering-independent downstream evaluation (differential expression, lineage reconstruction) that the paper did not evaluate
- Data are raw counts requiring a count-based noise model — the authors note that 'extending our framework to modeling count data is another promising avenue for future research'

## Inputs

- Target gene expression matrix from single-cell RNA-seq (e.g. 182-ESC matrix)
- Annotated control gene set (e.g. 629 cell-cycle genes from CellCycleData in Citrus)
- Simulated scRNA-seq data with known latent confounders and true cell labels (for benchmarking)

## Outputs

- Confounder-corrected expression matrix from scPLS
- Per-method r^2 values between inferred factors and true per-gene PVE (e.g. cellcycle_r2_results.csv)
- ARI table of k-means clustering on corrected data across methods (e.g. simulation_evaluation_results.csv)
- PCA comparison plots of uncorrected vs corrected data (e.g. pca_comparison.png)

## How to apply

Load a target expression matrix (e.g. 182-ESC genes from the CellCycleData dataset in Citrus) together with an annotated control set (e.g. 629 cell-cycle genes) and call scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center) from the Citrus package to jointly infer confounding effects and obtain corrected expression. For benchmarks, run the comparators scLVM, linear mixed model (LMM), PCA, and RUV on the same target/control configuration. Evaluate by computing r^2 between each method's inferred factor and the true per-gene confounder PVE (scPLS achieved r2=0.94 vs r2=0.92 for scLVM/LMM/PCA on ESC cell-cycle removal), or by Adjusted Rand Index of k-means clustering on corrected data across a PDE 20-100% sweep with and without dropout. Install dependencies Rcpp, RcppArmadillo, genlasso, and gplots; scPLS's block-wise EM makes it scalable to thousands of cells and tens of thousands of genes.

## Related tools

- **Citrus** (Toolkit hosting scPLS and the CellCycleData/Pollen datasets; provides the scPLS(Target, Control, ...) entry point) — http://chenmengjie.github.io/Citrus/
- **scPLS** (Joint PLS-regression inference of confounding effects from control and target genes via block-wise EM) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (C++ backend required to install Citrus and run the scPLS EM implementation)
- **RcppArmadillo** (Linear-algebra backend for the scPLS block-wise EM algorithm)
- **genlasso** (CRAN dependency of Citrus used for penalized fitting in scPLS)
- **gplots** (R package used to render the composite PCA comparison figure of corrected vs uncorrected data)

## Examples

```
library(devtools); install_github("ChenMengjie/Citrus"); fit <- scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)
```

## Evaluation signals

- cellcycle_r2_results.csv exists with an r^2 column for all four methods (scPLS, scLVM, LMM, PCA); scPLS should reach r^2 comparable to or above 0.92 (reported scPLS r2=0.94)
- pca_comparison.png renders and shows corrected data separating cell types better than uncorrected data
- simulation_evaluation_results.csv shows scPLS attaining comparable or higher Adjusted Rand Index than PCA, LMM, RUV, and scLVM across the PDE 20-100% sweep, with and without dropout
- PVE of estimated confounding factors matches the true per-gene PVE for the scPLS estimates
- Citrus installs cleanly only after Rcpp, RcppArmadillo, genlasso, and gplots are present — a missing dependency signals incorrect setup

## Limitations

- Because of model identifiability, scPLS can only infer biological effects (Lambda_u u_i), not the biological factors (u_i); direct inference of u_i is not feasible
- The 'best' confounding-removal method may vary by downstream analysis; scPLS was only evaluated for clustering-style benchmarks and PVE recovery
- No count-based model is provided; count-data extension is left as future work
- No software version numbers are stated for the tools

## Evidence

- [methods] scPLS signature: "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [abstract] joint modeling rationale: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects."
- [intro] scalability: "with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes"
- [readme] installation dependencies: "**Citrus** relies on the following R packages: **Rcpp**, **RcppArmadillo**, **genlasso**, **gplots**. All packagess are hosted on CRAN."
- [discussion] identifiability limitation: [evidence span withheld — anchor status: not_found]
- [abstract] empirical effectiveness: "With extensive simulations and comparisons with other methods, we demonstrate the effectiveness of scPLS."
