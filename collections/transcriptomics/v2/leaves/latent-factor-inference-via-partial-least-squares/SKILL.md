---
name: latent-factor-inference-via-partial-least-squares
description: Use when scRNA-seq expression data contain suspected unmeasured technical
  or cell-cycle confounders and a set of control genes is available; goal is confounder
  estimation and corrected expression for downstream clustering.
when_to_use_negative:
- You need to infer the biological factors u_i themselves — scPLS can only infer biological
  effects (Lambda_u u_i), not the factors, due to model identifiability
- Your data are raw counts and you require a count-based model — extending the framework
  to modeling count data is left as future work
- The downstream analysis is not clustering (e.g. differential expression, lineage
  reconstruction) — performance of scPLS in other analysis settings has not been evaluated
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: Citrus
  role: R package providing the scPLS(Target, Control, ...) function with block-wise
    EM inference; install via install_github('ChenMengjie/Citrus')
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: Dependency enabling the C++ core of the scPLS block-wise EM algorithm
- name: RcppArmadillo
  role: Dependency providing linear-algebra backend for the scPLS implementation
- name: genlasso
  role: Required R package dependency of Citrus installed from CRAN
- name: CellCycleData
  role: Control-gene resource dataset within Citrus used to supply control genes for
    confounder inference
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:4cceb0ade1cfdf54a5406a18f0b741a73de5a32fd097d7790a59ab7191266201
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellCycleData
  - Citrus
  - Rcpp
  - RcppArmadillo
  - genlasso
  - gplots
  - scPLS
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# latent-factor-inference-via-partial-least-squares

## Summary

Infer and remove hidden confounding factors in single-cell RNA-seq data with scPLS, which jointly models a control gene set and a target gene set via partial least squares regression, bridging PCA/LMM-type and RUV/scLVM-type methods. Implemented in the Citrus R package with an Rcpp/RcppArmadillo block-wise EM algorithm.

## When to use

Use when scRNA-seq expression data contain suspected unmeasured technical or cell-cycle confounders and a set of control genes is available; goal is confounder estimation and corrected expression for downstream clustering.

## When NOT to use

- You need to infer the biological factors u_i themselves — scPLS can only infer biological effects (Lambda_u u_i), not the factors, due to model identifiability
- Your data are raw counts and you require a count-based model — extending the framework to modeling count data is left as future work
- The downstream analysis is not clustering (e.g. differential expression, lineage reconstruction) — performance of scPLS in other analysis settings has not been evaluated

## Inputs

- Target gene expression matrix from scRNA-seq (cells x genes)
- Control gene expression matrix (e.g. cell-cycle or housekeeping control genes; CellCycleData provides a 50-gene control resource)
- Number of inferred factors k1 (target) and k2 (control), plus optional scPLS tuning parameters (iter, alpha, kappa, diagH, penalty, tol, Chunk/chunk.size, center)

## Outputs

- Estimated latent confounding factors / corrected target expression matrix
- Confounder-corrected expression suitable for k-means clustering and ARI evaluation
- PVE (proportion of variance explained) estimates of the inferred factors

## How to apply

Install Citrus (requires Rcpp, RcppArmadillo, genlasso, gplots) and call the exported scPLS(Target, Control, k1, k2, iter, alpha, kappa, ...) function with the target gene expression matrix and the control gene matrix as inputs. scPLS jointly models the two gene sets using partial least squares regression and infers latent confounding factors with a block-wise expectation maximization (EM) algorithm implemented in Rcpp/RcppArmadillo, making it scalable to thousands of cells and tens of thousands of genes. The method bridges PCA/LMM-type approaches (which use all genes) and RUV/scLVM-type approaches (which use control genes alone), with each recoverable as a special case, so tune the relative weighting of control vs. target information via the function parameters. After inferring factors, remove their effects from the target expression matrix and feed the corrected matrix to downstream analyses such as k-means clustering, scoring accuracy with adjusted Rand index (ARI) against true cell labels, or quantifying proportion of variance explained (PVE) of the estimated factors. The method is robust to reduced control-gene sets: ARI remains close to the full-set baseline when q is subsampled from 50 down to 10 control genes.

## Related tools

- **Citrus** (R package providing the scPLS(Target, Control, ...) function with block-wise EM inference; install via install_github('ChenMengjie/Citrus')) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (Dependency enabling the C++ core of the scPLS block-wise EM algorithm)
- **RcppArmadillo** (Dependency providing linear-algebra backend for the scPLS implementation)
- **genlasso** (Required R package dependency of Citrus installed from CRAN)
- **CellCycleData** (Control-gene resource dataset within Citrus used to supply control genes for confounder inference)

## Examples

```
install.packages(c("Rcpp","RcppArmadillo","genlasso","gplots")); library(devtools); install_github("ChenMengjie/Citrus"); fit <- scPLS(Target, Control, k1, k2)
```

## Evaluation signals

- K-means clustering on scPLS-corrected data should yield an Adjusted Rand Index (ARI) comparable to or higher than PCA, LMM, RUV, and scLVM across the PDE 20–100% sweep in both simulation scenarios, with or without dropout
- PVE of the estimated confounding factors should accurately reflect the true simulated latent factors
- ARI should remain close to the q=50 baseline when the control-gene set is subsampled to q = 10, 20, 30, or 40 genes
- The function call scPLS(Target, Control, k1, k2, ...) executes without error and returns inferred factors with dimensionality k1/k2
- Corrected data should show removal of technical/cell-cycle confounding effects in the two real-data applications (Pollen and CellCycleData)

## Limitations

- Because of model identifiability, scPLS can only be used to infer the biological effects (Lambda_u u_i), not the biological factors u_i
- The 'best' confounding-effect-removal method may vary depending on the downstream question; scPLS performance beyond clustering (e.g. differential expression, lineage reconstruction) was not evaluated
- No count-based model is provided; extending the framework to modeling count data is left as future work
- A clustering method in Citrus that accounts for confounding factors is not yet distributed — 'a tutorial of software on clustering will be available shortly'

## Evidence

- [methods] scPLS signature and implementation context: "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [abstract] Rationale: joint modeling via PLS improves confounder inference: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects."
- [intro] Scalability of the block-wise EM algorithm: "with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes"
- [intro] Method positioning between PCA/LMM and RUV/scLVM: "scPLS is closely related to and bridges between two existing subcategories of methods for transcriptome analysis"
- [readme] Installation dependencies of Citrus: "**Citrus** relies on the following R packages: **Rcpp**, **RcppArmadillo**, **genlasso**, **gplots**."
- [discussion] Identifiability limitation: [evidence span withheld — anchor status: not_found]
