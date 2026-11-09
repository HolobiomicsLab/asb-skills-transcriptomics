---
name: control-gene-set-subsampling
description: Use when assessing how robust an scPLS confounding-correction workflow
  is to the number of available control genes, e.g. when only a subset of a full control-gene
  panel can be measured.
when_to_use_negative:
- The analysis goal is inferring the underlying biological factors (u_i) rather than
  removing confounding effects — scPLS can only infer biological effects, not the
  factors themselves, due to model identifiability.
- Downstream evaluation is a clustering-external task (e.g. differential expression
  or lineage reconstruction) — the paper notes scPLS performance in other analysis
  settings has not been evaluated.
- The input data are real tissues with no known true cell-type labels, so ARI against
  ground truth cannot be computed.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Citrus
  role: R/C++ toolkit providing the exported scPLS() function used to fit scPLS on
    target and reduced control gene sets
  repo: http://chenmengjie.github.io/Citrus/
- name: scPLS
  role: Partial least squares method that jointly models control and target gene sets
    to estimate confounding factors at each subsampled control-set size
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: Required R dependency backing Citrus's C++ implementation of scPLS inference
- name: RcppArmadillo
  role: Required R dependency providing the linear algebra backend for the block-wise
    EM algorithm
- name: genlasso
  role: R dependency of Citrus installed as part of the environment for scPLS
- name: gplots
  role: R dependency of Citrus used for plotting/output in the environment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:57855ac494be8a232e1faddeed7fd408ff7de2323ebf0b4c7e85704d7c6a744f
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# control-gene-set-subsampling

## Summary

A sensitivity-analysis skill for scPLS-based confounding removal in scRNAseq simulations: it subsamples reduced control gene sets (q = 10, 20, 30, 40 from a full 50-gene set) and verifies that adjusted Rand index (ARI) clustering performance is maintained with fewer control genes.

## When to use

Use when assessing how robust an scPLS confounding-correction workflow is to the number of available control genes, e.g. when only a subset of a full control-gene panel can be measured.

## When NOT to use

- The analysis goal is inferring the underlying biological factors (u_i) rather than removing confounding effects — scPLS can only infer biological effects, not the factors themselves, due to model identifiability.
- Downstream evaluation is a clustering-external task (e.g. differential expression or lineage reconstruction) — the paper notes scPLS performance in other analysis settings has not been evaluated.
- The input data are real tissues with no known true cell-type labels, so ARI against ground truth cannot be computed.

## Inputs

- Scenario I simulated scRNAseq expression matrix with target genes, cell clusters, and a full 50-gene control set
- True cell-type labels for the simulated cells
- Reduced control-set sizes q in {10, 20, 30, 40} to subsample from the 50-gene control set

## Outputs

- ari_vs_control_set_size.csv — table of ARI versus control-set size q
- ari_curve_control_set_reduction.png — ARI curve across q = 10, 20, 30, 40 plus the q = 50 baseline
- Per-q scPLS-estimated confounding factors on the reduced control sets

## How to apply

Generate the scenario I simulated single-cell RNA-seq expression matrix with its cell clusters, target genes, and the full 50-gene control set, along with true cell-type labels. For each reduced size q in {10, 20, 30, 40}, randomly subsample q control genes from the full 50-gene control set. Fit scPLS via the Citrus package's exported scPLS() function — which uses Rcpp/RcppArmadillo-backed block-wise EM inference — on the target genes and each reduced control set to estimate the confounding factors. Cluster the confounding-corrected expression data at each q and score clustering accuracy against the true labels with adjusted Rand index (ARI). Tabulate ARI versus q (including the q = 50 reference point) and plot the ARI curve; the rationale is to demonstrate that scPLS needs only a modest control-gene set to retain high clustering performance.

## Related tools

- **Citrus** (R/C++ toolkit providing the exported scPLS() function used to fit scPLS on target and reduced control gene sets) — http://chenmengjie.github.io/Citrus/
- **scPLS** (Partial least squares method that jointly models control and target gene sets to estimate confounding factors at each subsampled control-set size) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (Required R dependency backing Citrus's C++ implementation of scPLS inference)
- **RcppArmadillo** (Required R dependency providing the linear algebra backend for the block-wise EM algorithm)
- **genlasso** (R dependency of Citrus installed as part of the environment for scPLS)
- **gplots** (R dependency of Citrus used for plotting/output in the environment)

## Examples

```
install_github("ChenMengjie/Citrus"); fit <- scPLS(Target = target_expr[, ], Control = control_expr[, sampled_q_genes], k1, k2)
```

## Evaluation signals

- Reproduced ARI values at reduced q remain close to the q = 50 baseline, matching the paper's reported scenario I trend of high ARI maintained down to small control-set sizes
- ari_vs_control_set_size.csv contains an ARI row for each q value (10, 20, 30, 40, and the 50 reference)
- ari_curve_control_set_reduction.png figure file exists
- Each q in {10, 20, 30, 40} is randomly subsampled from the same full 50-gene control set, ensuring consistency across q

## Limitations

- Direct inference of biological factors u_i is not possible in the current scPLS implementation due to model identifiability.
- The 'best' confounding effects removing method may vary depending on the downstream analysis; performance of scPLS in other analysis settings (e.g. differential expression) has not been evaluated.
- The framework models continuous expression and does not provide a count-based model; extension to count data is left as future work.

## Evidence

- [abstract] By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects.: "By modeling the two sets of genes jointly using the partial least squares regression, scPLS is capable of making full use of the data to improve the inference of confounding effects."
- [intro] with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes: "with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes"
- [methods] `scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`: "scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)"
- [readme] Citrus relies on the following R packages: Rcpp, RcppArmadillo, genlasso, gplots.: "Citrus relies on the following R packages: **Rcpp**, **RcppArmadillo**, **genlasso**, **gplots**. All packagess are hosted on CRAN."
