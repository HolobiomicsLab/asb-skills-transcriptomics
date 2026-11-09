---
name: differential-gene-expression-testing
description: Use when simulated or real single-cell data contain a binary covariate
  (e.g. stimulant/mutational status) and you need to quantify how well DE-based tests
  recover genes with covariate-pseudotime interactions versus a dedicated interaction
  model.
when_to_use_negative:
- The analysis goal is trajectory/pseudotime inference itself rather than gene-level
  covariate-interaction testing or its benchmarking.
- Data lack a per-cell covariate, so differential testing of covariate-positive versus
  covariate-negative cells and interaction detection are not defined.
- Features do not correspond to genes with simulated ground-truth interaction labels,
  so AUC against known truth cannot be computed.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PhenoPath
  role: Fits the Bayesian trajectory-covariate model via CAVI and provides interaction_effects()
    and significant_interactions() for posterior interaction ranking used as the reference
    method in the AUC benchmark.
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: R
  role: Implementation environment for all benchmark steps (simulation loading, model
    fitting, DE tests, ROC/AUC computation).
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: limma-voom
  role: Differential-expression-based interaction test comparing covariate-positive
    versus covariate-negative cells, used as a competing method.
- name: DESeq2
  role: Differential-expression-based interaction test used as a competing method
    in the benchmark.
- name: MAST
  role: Single-cell differential expression test used as a competing method in the
    benchmark.
- name: Monocle 2
  role: Differential-expression-based test used as a competing method in the benchmark.
- name: dplyr
  role: Tidy-table collation of per-method per-fraction AUC results into auc_benchmark_results.csv.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:f590631c01628a8db04c21abee7b0f67b1704fe86fff386e5b0f9b2f83a9df11
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# differential-gene-expression-testing

## Summary

Benchmark gene-level interaction detection by running standard differential-expression-based covariate tests (limma-voom, DESeq2, MAST, Monocle 2) against PhenoPath's posterior interaction ranking, evaluating each method by AUC on simulated data with known covariate-trajectory interaction labels (Fig 1f).

## When to use

Use when simulated or real single-cell data contain a binary covariate (e.g. stimulant/mutational status) and you need to quantify how well DE-based tests recover genes with covariate-pseudotime interactions versus a dedicated interaction model.

## When NOT to use

- The analysis goal is trajectory/pseudotime inference itself rather than gene-level covariate-interaction testing or its benchmarking.
- Data lack a per-cell covariate, so differential testing of covariate-positive versus covariate-negative cells and interaction detection are not defined.
- Features do not correspond to genes with simulated ground-truth interaction labels, so AUC against known truth cannot be computed.

## Inputs

- Simulated expression matrix from simulate_phenopath() with binary per-cell covariate and ground-truth interaction labels in the parameters data frame
- Gene-level count or normalized expression matrix per simulated interaction fraction (e.g. 5-10% through higher fractions of interacting genes)

## Outputs

- auc_benchmark_results.csv with one row per method (PhenoPath, limma-voom, DESeq2, MAST, Monocle 2) per interaction fraction and an AUC value in [0, 1]
- ROC curves per method per interaction fraction reproducing the structure of Fig 1f

## How to apply

Load the simulated datasets produced via simulate_phenopath(), spanning covariate-interaction gene fractions (default regime: first 20 genes with no interaction, latter 20 with interaction effects). Refit PhenoPath per dataset with phenopath() using coordinate ascent mean-field variational inference (elbo_tol = 1e-6, thin = 40), extracting effect sizes with interaction_effects() and significance calls with significant_interactions(), which classify an interaction as of interest by a posterior mean over posterior standard deviation threshold. For each competing method, run differential-expression tests comparing covariate-positive versus covariate-negative cells: limma-voom, DESeq2, MAST, and Monocle 2. Rank genes by each method's interaction/differential statistic, compute ROC curves against the simulated ground-truth interaction labels from the parameters data frame, and compute the AUC per method per interaction fraction, collating results into a tidy table (auc_benchmark_results.csv) reproducing Fig 1f.

## Related tools

- **PhenoPath** (Fits the Bayesian trajectory-covariate model via CAVI and provides interaction_effects() and significant_interactions() for posterior interaction ranking used as the reference method in the AUC benchmark.) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **R** (Implementation environment for all benchmark steps (simulation loading, model fitting, DE tests, ROC/AUC computation).) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **limma-voom** (Differential-expression-based interaction test comparing covariate-positive versus covariate-negative cells, used as a competing method.)
- **DESeq2** (Differential-expression-based interaction test used as a competing method in the benchmark.)
- **MAST** (Single-cell differential expression test used as a competing method in the benchmark.)
- **Monocle 2** (Differential-expression-based test used as a competing method in the benchmark.)
- **dplyr** (Tidy-table collation of per-method per-fraction AUC results into auc_benchmark_results.csv.)

## Examples

```
fit <- phenopath(Y, x, elbo_tol = 1e-6, thin = 40); sig <- significant_interactions(fit)
```

## Evaluation signals

- auc_benchmark_results.csv exists and contains one row per method per interaction fraction with an AUC value in [0, 1].
- PhenoPath's significance threshold |m_beta|/s_beta > k is applied consistently with significant_interactions() across all fractions.
- At low interaction fractions (5-10%) all methods show high AUC, while only PhenoPath maintains AUC as the interacting-gene fraction increases, matching the published finding.
- ROC curves are computed against the simulated ground-truth interaction labels taken from the simulate_phenopath() parameters data frame, not against inferred labels.
- Each competing DE test compares covariate-positive versus covariate-negative cells using the same cell partition as the PhenoPath fit.

## Limitations

- Differential-expression workflows lose interaction-detection ability (rapidly decreasing AUC) as the fraction of interacting genes grows, unlike PhenoPath.
- PhenoPath assumes features evolve linearly with pseudotime; nonlinear effects may occur and model fits should be checked for consistency with the data.
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings.

## Evidence

- [methods] Posterior interaction significance via mean/SD threshold: "We classify or label an interaction as of interest if"
- [methods] significant_interactions API: "significant_interactions(phenopath_fit, n)"
- [methods] CAVI inference engine: "We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form"
- [readme] PhenoPath purpose and covariate interactions: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates"
- [discussion] Linear-with-pseudotime assumption: "An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime."
