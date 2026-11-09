---
name: ari-vs-parameter-curve-visualization
description: Use when you must show that a method's clustering performance (ARI against
  true labels) remains stable as a hyperparameter is reduced, e.g. subsampling scPLS
  control genes to q = 10, 20, 30, 40 versus the full 50-gene baseline.
when_to_use_negative:
- You need a single point estimate of performance at a fixed setting rather than a
  robustness/sensitivity curve across parameter values
- No ground-truth labels are available, so adjusted Rand index against true cell types
  cannot be computed
- The question concerns scPLS performance on downstream analyses other than clustering
  (e.g. differential expression or lineage reconstruction), which the paper explicitly
  leaves out of scope
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Citrus
  role: Provides the exported scPLS() function used to estimate confounding factors
    at each reduced control-set size before clustering and ARI scoring
  repo: http://chenmengjie.github.io/Citrus/
- name: scPLS
  role: Jointly models control and target genes via partial least squares regression
    to infer confounding effects for each q
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: Required dependency backing the C++ implementation of Citrus's block-wise
    EM inference
- name: RcppArmadillo
  role: Required dependency backing the C++ implementation of Citrus's block-wise
    EM inference
- name: genlasso
  role: R package dependency of Citrus installed alongside the pipeline
- name: gplots
  role: R package dependency of Citrus used for plotting support
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:4fac52cec6b68efdbce63a53e41ea6deef5f552b97aea6413b73282034cba971
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
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ari-vs-parameter-curve-visualization

## Summary

Sensitivity-analysis visualization skill: tabulate and plot clustering accuracy (adjusted Rand index, ARI) against a varying method parameter — here the number of control genes q used by scPLS — to demonstrate robustness of confounding removal with reduced inputs.

## When to use

Use when you must show that a method's clustering performance (ARI against true labels) remains stable as a hyperparameter is reduced, e.g. subsampling scPLS control genes to q = 10, 20, 30, 40 versus the full 50-gene baseline.

## When NOT to use

- You need a single point estimate of performance at a fixed setting rather than a robustness/sensitivity curve across parameter values
- No ground-truth labels are available, so adjusted Rand index against true cell types cannot be computed
- The question concerns scPLS performance on downstream analyses other than clustering (e.g. differential expression or lineage reconstruction), which the paper explicitly leaves out of scope

## Inputs

- Scenario I simulated single-cell RNA-seq expression matrix with specified cell clusters, target genes, and a 50-gene control set
- True cell-type label vector for the simulated cells
- Reduced control-set sizes q in {10, 20, 30, 40} plus the q = 50 baseline

## Outputs

- ari_vs_control_set_size.csv — table of ARI versus q with a row for each q value
- ari_curve_control_set_reduction.png — ARI curve across q = 10, 20, 30, 40 and the q = 50 reference point

## How to apply

Generate the scenario I simulated scRNA-seq matrix with the full 50-gene control set and true cell-type labels. For each reduced size q in {10, 20, 30, 40}, randomly subsample q control genes from the 50-gene set, fit scPLS (via Citrus's exported scPLS() function, Rcpp/RcppArmadillo-backed block-wise EM) to estimate confounding factors, cluster the confounding-corrected expression data, and score against true labels with ARI. Tabulate ARI versus q (including the q = 50 reference) into ari_vs_control_set_size.csv, then plot the ARI curve across q as ari_curve_control_set_reduction.png. The reproduction is judged correct when the reduced-q ARI values stay close to the q = 50 baseline, matching the paper's scenario I trend of high ARI maintained down to small control-set sizes.

## Related tools

- **Citrus** (Provides the exported scPLS() function used to estimate confounding factors at each reduced control-set size before clustering and ARI scoring) — http://chenmengjie.github.io/Citrus/
- **scPLS** (Jointly models control and target genes via partial least squares regression to infer confounding effects for each q) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (Required dependency backing the C++ implementation of Citrus's block-wise EM inference)
- **RcppArmadillo** (Required dependency backing the C++ implementation of Citrus's block-wise EM inference)
- **genlasso** (R package dependency of Citrus installed alongside the pipeline)
- **gplots** (R package dependency of Citrus used for plotting support)

## Examples

```
install.packages(c("Rcpp","RcppArmadillo","genlasso","gplots")); devtools::install_github("ChenMengjie/Citrus"); scPLS(Target, Control, ...)
```

## Evaluation signals

- ari_vs_control_set_size.csv contains an ARI row for every q value (10, 20, 30, 40, 50)
- Reproduced ARI values at reduced q remain close to the q = 50 baseline
- ARI trend matches the paper's scenario I finding of high ARI maintained down to small control-set sizes
- Figure file ari_curve_control_set_reduction.png exists and shows ARI as a monotone curve across q values
- Each q replicate uses a random subsample drawn from the same original 50-gene control set (subsets are nested in the full set)

## Limitations

- The paper evaluates scPLS for clustering only; downstream analyses such as differential expression, lineage reconstruction, allele-specific expression, and spatial reconstruction are beyond its scope
- scPLS can infer biological effects (Lambda_u u_i) but not the biological factors (u_i) due to model identifiability
- The 'best' confounding-removal method may vary depending on the downstream analysis or question of interest; scPLS performance in other settings was not evaluated

## Evidence

- [results] In a sensitivity analysis, scPLS maintains a high ARI when a smaller subset of control genes is used: [evidence span withheld — anchor status: not_found]
- [intro] with a new block-wise expectation maximization (EM) algorithm, it is scalable: "with a new block-wise expectation maximization (EM) algorithm, it is scalable to thousands of cells and tens of thousands of genes"
- [readme] Citrus relies on the following R packages: "**Citrus** relies on the following R packages: **Rcpp**, **RcppArmadillo**, **genlasso**, **gplots**."
- [discussion] the “best” confounding effects removing method may vary: "the “best” confounding effects removing method may vary depending on the question of interest."
