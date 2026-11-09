---
name: roc-auc-benchmark-evaluation
description: Use when simulated single-cell datasets with ground-truth covariate-pseudotime
  interaction labels are available and you must compare interaction-detection accuracy
  (AUC) of PhenoPath vs DE-based methods across varying fractions of interacting genes.
when_to_use_negative:
- Input is real experimental data without simulated ground-truth interaction labels
  — no ROC/AUC can be computed
- The analysis goal is pseudotime/covariate inference itself rather than benchmarking
  detection accuracy of competing methods
- Features are assumed to evolve nonlinearly with pseudotime, since PhenoPath assumes
  features evolve linearly with respect to pseudotime
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PhenoPath
  role: Method under benchmark; refit per simulated dataset with CAVI to rank genes
    by |m_beta|/s_beta interaction significance
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: limma-voom
  role: Differential-expression-based interaction test comparing covariate-positive
    versus covariate-negative cells to produce a gene ranking statistic
- name: DESeq2
  role: Differential-expression-based interaction test producing gene rankings for
    ROC/AUC comparison
- name: MAST
  role: Differential-expression-based interaction test producing gene rankings for
    ROC/AUC comparison
- name: Monocle 2
  role: Differential-expression-based interaction test producing gene rankings for
    ROC/AUC comparison
  repo: https://bioconductor.org/packages/release/bioc/html/monocle.html
- name: dplyr
  role: R package for collating per-method per-fraction AUC results into a tidy table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:d8617fc19ebb23fefe28acbebd8498fdacb98e93e058c9f74028604f2d878283
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# roc-auc-benchmark-evaluation

## Summary

Benchmarks PhenoPath's covariate-trajectory interaction detection against differential-expression workflows (limma-voom, DESeq2, MAST, Monocle 2) by ranking genes per method and computing ROC curves/AUC on simulated data with known interaction labels, reproducing Fig 1f.

## When to use

Use when simulated single-cell datasets with ground-truth covariate-pseudotime interaction labels are available and you must compare interaction-detection accuracy (AUC) of PhenoPath vs DE-based methods across varying fractions of interacting genes.

## When NOT to use

- Input is real experimental data without simulated ground-truth interaction labels — no ROC/AUC can be computed
- The analysis goal is pseudotime/covariate inference itself rather than benchmarking detection accuracy of competing methods
- Features are assumed to evolve nonlinearly with pseudotime, since PhenoPath assumes features evolve linearly with respect to pseudotime

## Inputs

- Simulated single-cell expression datasets from simulate_phenopath() with varying covariate-interaction gene fractions
- Simulated ground-truth interaction labels from the parameters data frame
- Per-cell binary/continuous covariate values (e.g. mutational status or stimulus)

## Outputs

- auc_benchmark_results.csv with one row per method per interaction fraction and an AUC value in [0, 1]
- ROC curves per method per interaction fraction
- PhenoPath interaction effect sizes (interaction_effects()) and significance calls (significant_interactions())

## How to apply

Load the simulated datasets from simulate_phenopath() spanning covariate-interaction gene fractions (default regime: first 20 genes with no interaction, latter 20 with interaction effects). Refit PhenoPath with phenopath() using coordinate ascent mean-field variational inference (elbo_tol = 1e-6, thin = 40), extract posterior interaction effect sizes via interaction_effects() and significance calls via significant_interactions(). For each competing method, run differential-expression-based interaction tests comparing covariate-positive versus covariate-negative cells (limma-voom, DESeq2, MAST, Monocle 2). Rank genes by each method's interaction/differential statistic, compute ROC curves using the simulated ground-truth interaction labels from the parameters data frame, and compute AUC per method per interaction fraction into a tidy table (auc_benchmark_results.csv) reproducing the structure of Fig 1f. The rationale is that when only 5-10% of genes carry interactions all methods perform well, but PhenoPath uniquely maintains detection ability as the interaction fraction increases.

## Related tools

- **PhenoPath** (Method under benchmark; refit per simulated dataset with CAVI to rank genes by |m_beta|/s_beta interaction significance) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **limma-voom** (Differential-expression-based interaction test comparing covariate-positive versus covariate-negative cells to produce a gene ranking statistic)
- **DESeq2** (Differential-expression-based interaction test producing gene rankings for ROC/AUC comparison)
- **MAST** (Differential-expression-based interaction test producing gene rankings for ROC/AUC comparison)
- **Monocle 2** (Differential-expression-based interaction test producing gene rankings for ROC/AUC comparison) — https://bioconductor.org/packages/release/bioc/html/monocle.html
- **dplyr** (R package for collating per-method per-fraction AUC results into a tidy table)

## Examples

```
fit <- phenopath(Y, x, elbo_tol = 1e-6, thin = 40); ie <- interaction_effects(fit); sig <- significant_interactions(fit)
```

## Evaluation signals

- auc_benchmark_results.csv exists and contains exactly one row per method per interaction fraction
- All AUC values fall in [0, 1]
- PhenoPath's significance threshold |m_beta|/s_beta > k is applied consistently with significant_interactions()
- With 5-10% interacting genes, all methods achieve high AUC; as the fraction increases, AUC of all methods except PhenoPath decreases while PhenoPath maintains interaction detection ability
- Gene rankings are computed against the simulated ground-truth interaction labels from the parameters data frame

## Limitations

- An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime; nonlinear effects may occur and checks should be conducted to verify model fits
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings (deferred to future work)
- Benchmark conclusions are grounded in simulated data with a default regime of 20 non-interacting and 20 interacting genes; behavior on real data without ground truth cannot be AUC-validated

## Evidence

- [methods] all algorithms high AUC at low interaction fraction: [evidence span withheld — anchor status: not_found]
- [discussion] An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime.: "An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime."
- [methods] significant_interactions usage: "significant_interactions(phenopath_fit, n)"
- [readme] CAVI inference implemented in Rcpp: [evidence span withheld — anchor status: external_doc]
