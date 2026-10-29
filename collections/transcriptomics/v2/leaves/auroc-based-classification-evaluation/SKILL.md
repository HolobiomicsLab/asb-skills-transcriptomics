---
name: auroc-based-classification-evaluation
description: Use when GEDI-trained Qi parameters from a multi-sample, multi-condition
  dataset (e.g., two independent COVID-19 cohorts with COVID vs. healthy labels) must
  be validated via cross-cohort SVM classification with AUROC as the metric.
when_to_use_negative:
- Only one cohort is available — there is no independent cohort on which to compute
  a cross-cohort AUROC
- The task is cluster-free differential gene expression or pathway/GRN activity inference
  from GEDI output rather than sample-level classification
- No trained GEDI model exists (no Qi parameters); cells/genes have not yet been filtered
  (log10_total_counts > 2.5, log10_total_features_by_counts > 2, subsets_Mito_percent
  <= 20) and GEDI has not been run
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GEDI
  role: Generative model whose sample-specific manifold parameters (Qi) serve as classification
    features for disease-status prediction
  repo: https://github.com/csglab/GEDI/
- name: R
  role: Language in which the parameter matrix Q is assembled, the top-20 variable
    parameters are selected, the SVM is trained, and AUROC is computed
- name: Harmony
  role: Provided the integrated space used as input for samples in cohort 1 of the
    COVID-19 dataset in the article's analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:df7c7bac73a13e2e38149f574826accfc57d54cd5f9d2354c0ea22d47491ce05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - Harmony
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# auroc-based-classification-evaluation

## Summary

Evaluates whether GEDI-learned sample-specific manifold parameters (model$params$Qi) predict sample-level disease status by training support vector machines in one cohort and scoring area under the ROC curve (AUROC) in an independent cohort. This is the manuscript's cross-cohort validation of GEDI parameters as machine-learning features.

## When to use

Use when GEDI-trained Qi parameters from a multi-sample, multi-condition dataset (e.g., two independent COVID-19 cohorts with COVID vs. healthy labels) must be validated via cross-cohort SVM classification with AUROC as the metric.

## When NOT to use

- Only one cohort is available — there is no independent cohort on which to compute a cross-cohort AUROC
- The task is cluster-free differential gene expression or pathway/GRN activity inference from GEDI output rather than sample-level classification
- No trained GEDI model exists (no Qi parameters); cells/genes have not yet been filtered (log10_total_counts > 2.5, log10_total_features_by_counts > 2, subsets_Mito_percent <= 20) and GEDI has not been run

## Inputs

- Trained GEDI model (RDS file) with per-sample Qi parameters (model$params$Qi)
- Sample-level metadata matrix H mapping each sample to cohort and disease status (COVID vs. healthy)
- Multi-sample, multi-condition single-cell COVID-19 dataset with two independent cohorts

## Outputs

- covid_svm_auroc.csv containing the two cross-cohort AUROC values
- Trained SVM classifier per split direction with predicted labels/probabilities for held-out cohort samples

## How to apply

Assemble the sample-specific parameter matrix Q by concatenating model$params$Qi across samples, so rows are genes x latent variables and columns are samples; build the metadata matrix H giving each sample's cohort and disease status. Select the top 20 most variable parameters by ordering rows of Q on standard deviation (apply(Q,1,sd), decreasing=TRUE, first 20), following the manuscript's top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20] step. Train an SVM on cohort 2 using the top-20 parameter values as features and disease status as labels, then predict cohort 1 samples and compute AUROC; reverse the split (train on cohort 1, test on cohort 2). Write both AUROC values to covid_svm_auroc.csv. The rationale is that GEDI's deconvolved sample-to-sample variability should encode disease state well enough to transfer across independent cohorts, using Harmony's integrated space for cohort 1 of the COVID-19 dataset as in the article.

## Related tools

- **GEDI** (Generative model whose sample-specific manifold parameters (Qi) serve as classification features for disease-status prediction) — https://github.com/csglab/GEDI/
- **R** (Language in which the parameter matrix Q is assembled, the top-20 variable parameters are selected, the SVM is trained, and AUROC is computed)
- **Harmony** (Provided the integrated space used as input for samples in cohort 1 of the COVID-19 dataset in the article's analysis)

## Examples

```
top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20]; svm_model <- svm(x = t(Q[top_parameters, train_samples]), y = H$status[train_samples]); pred <- predict(svm_model, t(Q[top_parameters, test_samples]))
```

## Evaluation signals

- Train on cohort 2 / test on cohort 1 yields AUROC = 1.0 (perfect COVID vs. healthy classification)
- Reversed split (train cohort 1, test cohort 2) yields AUROC = 0.97
- Exactly the top 20 parameters by row-wise standard deviation of Q are used as SVM features
- covid_svm_auroc.csv exists and contains both AUROC values, one per split direction
- Q matrix dimensions are (genes x latent variables) rows by samples columns, matching the concatenated model$params$Qi

## Limitations

- GEDI currently supports linear manifold learning (with optional hyperellipsoid restriction); other parametric manifold approximations are future extensions, which bounds the parameter space available as classification features
- More extensive analyses are needed to understand how factors such as sample/cell counts, sequencing depth, and variances influence GEDI and downstream method performance
- Classification performance depends on having genuinely independent cohorts with disease-status labels; results (AUROC 1.0 / 0.97) are specific to this COVID-19 two-cohort benchmark

## Evidence

- [methods] top-20 parameter selection: "top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20]"
- [discussion] cohort-1 input space: "we used the integrated space identified by Harmony41 for the samples in cohort 1 of the COVID-19 dataset"
- [abstract] machine learning of sample characteristics: "and machine learning-based prediction of sample characteristics from single-cell data"
- [readme] GEDI installation environment: [evidence span withheld — anchor status: external_doc]
