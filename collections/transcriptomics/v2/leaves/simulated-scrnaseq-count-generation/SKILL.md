---
name: simulated-scrnaseq-count-generation
description: Use when you need a controlled scRNA-seq simulation fixture (known clusters,
  target/control gene partition, true labels) to evaluate confounding-effect removal
  or control-gene-set robustness for scPLS-type methods.
when_to_use_negative:
- Input is real experimental scRNA-seq data rather than a simulation fixture — this
  skill only produces synthetic data with known ground-truth labels
- You need a count-based generative model — the article explicitly leaves 'extending
  our framework to modeling count data' as future work
- You need to evaluate downstream analyses other than clustering (e.g. differential
  expression, lineage reconstruction), which the paper scopes out
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Citrus
  role: Hosts the scPLS() function whose Target/Control interface the simulated fixture
    must match; provides the EM-backed confounding-factor inference and clustering
    used to score ARI on the simulated data
  repo: http://chenmengjie.github.io/Citrus/
- name: scPLS
  role: Partial least squares model fitted on the simulated Target and Control matrices
    to estimate confounding factors (kz) and biological factors (ku), selected by
    BIC
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: Required dependency enabling Citrus's C++-backed computation on the simulated
    matrices
- name: RcppArmadillo
  role: Required dependency powering the block-wise EM inference over the simulated
    expression data
- name: genlasso
  role: Provides trend filtering for residual de-noising of the confounding-corrected
    simulated expression
- name: gplots
  role: Citrus dependency used for plotting/visualization of results from the simulated
    benchmarks
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:6dda7b64ec0927e87ef311bfd376014c6809730849680b530553c17951801838
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# simulated-scrnaseq-count-generation

## Summary

Generate scenario I simulated single-cell RNA-seq expression matrices — with known cell clusters, target genes, and a 50-gene control set plus true labels — to benchmark scPLS confounding-factor inference and downstream clustering (ARI), including sensitivity to reduced control-set sizes (q = 10, 20, 30, 40).

## When to use

Use when you need a controlled scRNA-seq simulation fixture (known clusters, target/control gene partition, true labels) to evaluate confounding-effect removal or control-gene-set robustness for scPLS-type methods.

## When NOT to use

- Input is real experimental scRNA-seq data rather than a simulation fixture — this skill only produces synthetic data with known ground-truth labels
- You need a count-based generative model — the article explicitly leaves 'extending our framework to modeling count data' as future work
- You need to evaluate downstream analyses other than clustering (e.g. differential expression, lineage reconstruction), which the paper scopes out

## Inputs

- Scenario I simulation specification: number of cell clusters, target genes, and 50 control genes, plus true cell-type labels
- Full 50-gene simulated control set for random subsampling to q = 10, 20, 30, 40
- R environment with Citrus, Rcpp, RcppArmadillo, genlasso, gplots installed

## Outputs

- Simulated Target and Control gene expression matrices with true labels (fixture for scPLS(Target, Control, ...) calls)
- Fitted scPLS model object with estimated confounding factors, BIC-selected kz/ku, EM convergence status (e.g. scpls_model_fit.rds)
- ari_vs_control_set_size.csv with an ARI row for each q value
- ari_curve_control_set_reduction.png (ARI curve across q = 10, 20, 30, 40 plus the q = 50 reference)

## How to apply

Construct the scenario I simulated expression matrix with the specified cell clusters, target genes, and a full 50-gene control set, keeping the true cell-type labels for later ARI scoring. Subsample reduced control sets of q ∈ {10, 20, 30, 40} genes from the full 50-gene control set to probe robustness. Split each matrix into Target and Control gene blocks matching the scPLS(Target, Control, ...) interface, and fit via the Citrus package's Rcpp/RcppArmadillo-backed block-wise EM, initializing latent factors with partial least squares regression and selecting kz and ku by BIC over a grid of candidate dimensions. Cluster the confounding-corrected residuals (optionally denoised with genlasso trend filtering) and score against true labels with adjusted Rand index (ARI), tabulating ARI versus q. Validate by confirming ARI at reduced q stays close to the q = 50 baseline and that outputs (ari_vs_control_set_size.csv, ari_curve_control_set_reduction.png) exist.

## Related tools

- **Citrus** (Hosts the scPLS() function whose Target/Control interface the simulated fixture must match; provides the EM-backed confounding-factor inference and clustering used to score ARI on the simulated data) — http://chenmengjie.github.io/Citrus/
- **scPLS** (Partial least squares model fitted on the simulated Target and Control matrices to estimate confounding factors (kz) and biological factors (ku), selected by BIC) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (Required dependency enabling Citrus's C++-backed computation on the simulated matrices)
- **RcppArmadillo** (Required dependency powering the block-wise EM inference over the simulated expression data)
- **genlasso** (Provides trend filtering for residual de-noising of the confounding-corrected simulated expression)
- **gplots** (Citrus dependency used for plotting/visualization of results from the simulated benchmarks)

## Examples

```
install.packages(c("Rcpp","RcppArmadillo","genlasso","gplots")); install_github("ChenMengjie/Citrus")
```

## Evaluation signals

- ari_vs_control_set_size.csv contains an ARI row for each q value (10, 20, 30, 40) plus the q = 50 reference point
- ARI at reduced control-set sizes remains close to the q = 50 baseline and matches the paper's reported scenario I trend of high ARI down to small q
- EM iteration terminates within the specified convergence limit/tolerance on the simulated fixture
- Fitted model object exposes the estimated confounding factors and the BIC-selected dimensions kz and ku
- ari_curve_control_set_reduction.png figure file is produced and exists on disk

## Limitations

- scPLS can only infer biological effects (Lambda_u u_i), not the biological factors u_i, because of model identifiability
- No count-based model is provided; extending the framework to modeling count data is left as future work
- Performance of scPLS in downstream analyses other than clustering has not been evaluated — the 'best' confounding-effect removal method may vary with the question of interest

## Evidence

- [intro] Joint PLS modeling rationale: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding factors."
- [intro] Scalable EM algorithm: "with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes"
- [methods] scPLS function interface for Target/Control matrices: "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [abstract] Confounding control as prerequisite for clustering evaluation: "Controlling for confounding effects in scRNAseq data is a crucial step for accurate downstream analysis."
- [readme] Citrus toolkit and dependencies for the simulation-benchmark stack: "**Citrus** relies on the following R packages: **Rcpp**, **RcppArmadillo**, **genlasso**, **gplots**. All packagess are hosted on CRAN."
