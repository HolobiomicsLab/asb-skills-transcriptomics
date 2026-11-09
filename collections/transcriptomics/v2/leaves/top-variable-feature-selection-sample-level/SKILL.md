---
name: top-variable-feature-selection-sample-level
description: Use when a per-sample parameter/feature matrix (rows = parameters, columns
  = samples) must be reduced to its most variable features before training or evaluating
  a cross-cohort classifier on sample-level labels.
when_to_use_negative:
- When classification is at the single-cell level rather than the sample level — this
  selection is designed for per-sample parameter matrices
- When the parameter matrix already contains few features (selection by SD adds no
  value and may drop discriminative low-variance features)
- When the task is differential expression or integration itself rather than ML-based
  prediction of sample characteristics
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GEDI
  role: Produces the sample-specific manifold parameter matrix Qi whose per-parameter
    variability across samples is ranked for feature selection
  repo: https://github.com/csglab/GEDI
- name: R
  role: Computes per-parameter standard deviations with apply(Q,1,sd) and orders/
    subsets the top parameters
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:801569256cd84fe923bdf349fe96c642e24a4847c0816f848cb84bdee464ded0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# Top-variable feature selection for sample-level parameters

## Summary

Selects the top-N most variable sample-specific parameters (by standard deviation across samples) from a parameter matrix such as GEDI's Q matrix, to reduce noise before training a sample-level classifier like the SVM used for COVID-19 vs. healthy prediction.

## When to use

Use when a per-sample parameter/feature matrix (rows = parameters, columns = samples) must be reduced to its most variable features before training or evaluating a cross-cohort classifier on sample-level labels.

## When NOT to use

- When classification is at the single-cell level rather than the sample level — this selection is designed for per-sample parameter matrices
- When the parameter matrix already contains few features (selection by SD adds no value and may drop discriminative low-variance features)
- When the task is differential expression or integration itself rather than ML-based prediction of sample characteristics

## Inputs

- Sample-specific parameter matrix Q (rows = gene x latent-variable parameters, columns = samples), e.g. concatenated GEDI model$params$Qi
- Sample-level metadata matrix H mapping each sample to cohort and disease status (COVID vs. healthy)
- Number of top features to retain (default in the article: 20)

## Outputs

- Ordered index of the top 20 most variable parameters (top_parameters)
- Reduced feature matrix (top-20 parameters x samples) for classifier training/testing
- Downstream AUROC values (e.g., covid_svm_auroc.csv with 1.0 for cohort1-as-test and 0.97 for cohort2-as-test)

## How to apply

Assemble the sample-specific parameter matrix Q (e.g., by concatenating GEDI's model$params$Qi across samples so that rows are gene x latent-variable parameters and columns are samples), and pair it with a sample-level metadata matrix H holding cohort and disease status. Order the rows of Q by their standard deviation across samples using apply(Q,1,sd) with decreasing=TRUE, and keep the first 20 rows (top_parameters <- order(apply(Q,1,sd), decreasing=T)[1:20]). Use the selected top-20 parameter values as features for the downstream classifier (e.g., an SVM trained on cohort 2 to predict cohort 1 disease status, or vice versa). The rationale is that highly variable parameters carry the cross-sample signal distinguishing conditions, while low-variance parameters contribute little discriminatory information. Sanity-check that the selection preserves both cohorts' samples and that 20 features is small relative to the number of samples to avoid overfitting.

## Related tools

- **GEDI** (Produces the sample-specific manifold parameter matrix Qi whose per-parameter variability across samples is ranked for feature selection) — https://github.com/csglab/GEDI
- **R** (Computes per-parameter standard deviations with apply(Q,1,sd) and orders/ subsets the top parameters)

## Examples

```
top_parameters <- order( apply(Q,1,sd), decreasing=TRUE )[1:20]; Q_top <- Q[top_parameters, ]
```

## Evaluation signals

- top_parameters contains exactly 20 unique row indices within the row count of Q
- Selected parameters have the 20 largest standard deviations; spot-check sd() ordering is decreasing
- Cross-cohort SVM AUROC on the selected features matches published values (1.0 when trained on cohort 2 / tested on cohort 1; 0.97 in the reverse split)
- Both AUROC values are written to covid_svm_auroc.csv with correct cohort split labels
- All samples from both cohorts are present in the reduced feature matrix with matching H metadata

## Limitations

- Selection is purely unsupervised (variance-based); it does not guarantee features are condition-discriminative for every dataset
- Expected AUROCs (1.0 / 0.97) are specific to the COVID-19 two-cohort dataset and need not replicate on other datasets or modalities
- The fixed cutoff of 20 parameters is a hyperparameter; other values are untested in the article

## Evidence

- [methods] top-variable selection code: "top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20]"
- [abstract] GEDI supports ML-based prediction of sample characteristics: "machine learning-based prediction of sample characteristics from single-cell data"
- [readme] GEDI installation from GitHub: [evidence span withheld — anchor status: external_doc]
