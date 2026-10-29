---
name: single-cell-simulation-benchmarking
description: Use when comparing interaction-detection performance of a new trajectory
  method against DE-based alternatives on simulated single-cell datasets with known
  ground-truth interaction labels, as in Fig 1f.
when_to_use_negative:
- Data are real (non-simulated) expression with no ground-truth interaction labels
  — ROC/AUC benchmarking against truth is not possible
- The goal is trajectory (pseudotime) ordering accuracy rather than covariate-trajectory
  interaction detection
- Input is already a feature table of ranked interaction statistics rather than raw/normalized
  cell-by-gene expression
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: PhenoPath
  role: Method under benchmark; refit per simulated dataset via CAVI and posterior
    interaction calls with interaction_effects() and significant_interactions()
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: limma-voom
  role: DE-based competitor interaction test on covariate-positive vs covariate-negative
    cells
- name: DESeq2
  role: DE-based competitor interaction test on covariate-positive vs covariate-negative
    cells
- name: MAST
  role: DE-based competitor interaction test on covariate-positive vs covariate-negative
    cells
- name: Monocle 2
  role: DE-based competitor interaction test on covariate-positive vs covariate-negative
    cells
  repo: https://bioconductor.org/packages/release/bioc/html/monocle.html
- name: dplyr
  role: Tidy-table collation of per-method per-fraction AUC results
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:96b9654ed30b1c956e30973ea15b0d482193d134df226bb2ff7c74d7579a9d83
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - MAST
  - Monocle 2
  - PhenoPath
  - R
  - dplyr
  - limma-voom
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# single-cell-simulation-benchmarking

## Summary

A benchmarking skill for evaluating how well a pseudotime/covariate-interaction method (PhenoPath) recovers covariate-trajectory interactions relative to differential-expression workflows (limma-voom, DESeq2, MAST, Monocle 2) on simulated single-cell data, quantified by ROC AUC across increasing fractions of interacting genes.

## When to use

Use when comparing interaction-detection performance of a new trajectory method against DE-based alternatives on simulated single-cell datasets with known ground-truth interaction labels, as in Fig 1f.

## When NOT to use

- Data are real (non-simulated) expression with no ground-truth interaction labels — ROC/AUC benchmarking against truth is not possible
- The goal is trajectory (pseudotime) ordering accuracy rather than covariate-trajectory interaction detection
- Input is already a feature table of ranked interaction statistics rather than raw/normalized cell-by-gene expression

## Inputs

- Simulated single-cell expression matrices from simulate_phenopath() spanning varying covariate-interaction gene fractions
- Simulated ground-truth interaction labels from the parameters data frame
- Per-cell covariate vector (e.g. covariate-positive vs covariate-negative status)

## Outputs

- auc_benchmark_results.csv with one row per method per interaction fraction and an AUC value in [0, 1]
- Per-method gene rankings by interaction/differential statistic
- ROC curves and collated AUC table reproducing the structure of Fig 1f

## How to apply

Load simulated datasets spanning covariate-interaction gene fractions (default regime: first 20 genes with no interaction, latter 20 with interaction effects). Refit PhenoPath on each dataset using coordinate ascent mean-field variational inference (phenopath() with elbo_tol = 1e-6, thin = 40), then extract posterior interaction effects with interaction_effects() and call hits with significant_interactions(), which labels an interaction of interest by the posterior mean over posterior standard deviation threshold (|m_beta|/s_beta > k). For each competitor, run DE-based interaction tests comparing covariate-positive versus covariate-negative cells via limma-voom, DESeq2, MAST, and Monocle 2, rank genes by each method's statistic, and compute ROC curves against the simulated ground-truth labels. Collate per-method, per-fraction AUC values in [0,1] into a tidy table (auc_benchmark_results.csv) mirroring Fig 1f.

## Related tools

- **PhenoPath** (Method under benchmark; refit per simulated dataset via CAVI and posterior interaction calls with interaction_effects() and significant_interactions()) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **limma-voom** (DE-based competitor interaction test on covariate-positive vs covariate-negative cells)
- **DESeq2** (DE-based competitor interaction test on covariate-positive vs covariate-negative cells)
- **MAST** (DE-based competitor interaction test on covariate-positive vs covariate-negative cells)
- **Monocle 2** (DE-based competitor interaction test on covariate-positive vs covariate-negative cells) — https://bioconductor.org/packages/release/bioc/html/monocle.html
- **dplyr** (Tidy-table collation of per-method per-fraction AUC results)

## Examples

```
fit <- phenopath(Y, x, elbo_tol = 1e-6, thin = 40); sig <- significant_interactions(fit, 5)
```

## Evaluation signals

- auc_benchmark_results.csv exists with exactly one row per method per interaction fraction
- All AUC values lie in [0, 1]
- PhenoPath's hit calls use |m_beta|/s_beta > k consistently with significant_interactions()
- For low interacting-gene fractions (5-10%) all methods show high AUC, while only PhenoPath maintains AUC as the fraction increases
- ROC curves are computed against the simulated ground-truth interaction labels from the parameters data frame

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects may occur and model fits should be verified against the data
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings
- Benchmark conclusion is specific to the simulated regime: at small interaction fractions (5-10%) all methods achieve high AUC, so differences only emerge as the fraction grows

## Evidence

- [methods] CAVI and convergence settings: "We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form"
- [methods] interaction significance rule: "We classify or label an interaction as of interest if"
- [readme] Rcpp implementation: [evidence span withheld — anchor status: external_doc]
- [discussion] linearity assumption: "An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime."
- [abstract] framework purpose: "We describe a novel statistical framework that learns how pseudotime trajectories can be modulated through covariates that encode such factors."
