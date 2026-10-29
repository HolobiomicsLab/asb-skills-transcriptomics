---
name: cross-cohort-svm-disease-classification
description: Use when you have a trained GEDI model on a multi-sample, multi-condition
  dataset (e.g., two independent COVID-19 cohorts) and need to predict sample-level
  disease labels in an unseen cohort from learned parameters.
when_to_use_negative:
- You only have a single cohort with no independent cohort to validate against — the
  skill's evaluation depends on cross-cohort prediction.
- Your input is raw single-cell counts without a trained GEDI model; the skill requires
  the learned sample-specific parameter matrix Q, not per-cell expression data.
- The prediction target is a per-cell property (e.g., cell-type annotation) rather
  than a sample-level characteristic; the Q matrix has one column per sample.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GEDI
  role: Generative model whose learned sample-specific manifold parameters (Qi) serve
    as the SVM classification features
  repo: https://github.com/csglab/GEDI
- name: R
  role: Implementation language for assembling Q/H matrices, selecting top-20 parameters
    by SD, training/predicting with SVM, and computing AUROC
  repo: https://github.com/csglab/GEDI_manuscript
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:98abb387de044c8b818f5c3cd4f11e83809fa293c7aa12bdc2347f7ef9f7f267
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-cohort-svm-disease-classification

## Summary

Trains support vector machines on GEDI's sample-specific manifold parameters (model$params$Qi) to predict sample-level disease status (COVID-19 vs. healthy) across independent cohorts, using the top 20 most variable parameters as features and reporting AUROC.

## When to use

Use when you have a trained GEDI model on a multi-sample, multi-condition dataset (e.g., two independent COVID-19 cohorts) and need to predict sample-level disease labels in an unseen cohort from learned parameters.

## When NOT to use

- You only have a single cohort with no independent cohort to validate against — the skill's evaluation depends on cross-cohort prediction.
- Your input is raw single-cell counts without a trained GEDI model; the skill requires the learned sample-specific parameter matrix Q, not per-cell expression data.
- The prediction target is a per-cell property (e.g., cell-type annotation) rather than a sample-level characteristic; the Q matrix has one column per sample.

## Inputs

- Trained GEDI model (RDS file) containing per-sample manifold parameters model$params$Qi
- Sample-level metadata matrix H with cohort membership and disease status (COVID vs. healthy)
- Feature matrix Q (rows = gene x latent variable parameters, columns = samples)

## Outputs

- covid_svm_auroc.csv containing AUROC for both cross-cohort directions
- Trained SVM models (one per direction) with predictions for the held-out cohort

## How to apply

Load the trained GEDI model (RDS file, available via the Zenodo accessions 10.5281/zenodo.8222039/8222697) and assemble the sample-specific parameter matrix Q by concatenating model$params$Qi so that rows are gene-by-latent-variable parameters and columns are samples. Build the sample-level metadata matrix H mapping each sample to its cohort and COVID vs. healthy status. Select the top 20 most variable parameters via order(apply(Q,1,sd), decreasing=TRUE)[1:20] and use their values as SVM features. Train the SVM on cohort 2 and predict disease status for cohort 1, computing AUROC (expected 1.0); then reverse the split — train on cohort 1, evaluate on cohort 2 (expected 0.97). Write both AUROC values to covid_svm_auroc.csv. The rationale is that GEDI's sample-specific manifold parameters capture disease-relevant transcriptional shifts, enabling machine learning-based prediction of sample characteristics from single-cell data.

## Related tools

- **GEDI** (Generative model whose learned sample-specific manifold parameters (Qi) serve as the SVM classification features) — https://github.com/csglab/GEDI
- **R** (Implementation language for assembling Q/H matrices, selecting top-20 parameters by SD, training/predicting with SVM, and computing AUROC) — https://github.com/csglab/GEDI_manuscript

## Examples

```
top_parameters <- order( apply(Q,1,sd), decreasing=T )[1:20]; fit <- e1071::svm(status ~ ., data=train_df); auroc <- pROC::auc(pROC::roc(test_labels, as.numeric(predict(fit, test_df))))
```

## Evaluation signals

- AUROC = 1.0 when the SVM is trained on cohort 2 and evaluated on cohort 1 (perfect COVID vs. healthy classification)
- AUROC = 0.97 for the reversed split (train on cohort 1, evaluate on cohort 2)
- Exactly 20 parameter rows are selected (the 20 highest standard-deviation rows of Q); Q dimensions are genes x latent variables by samples
- Output file covid_svm_auroc.csv exists and contains two AUROC values, one per train/test direction

## Limitations

- The reported AUROC values (1.0 and 0.97) are specific to the COVID-19 two-cohort dataset; other datasets may not replicate perfect separation.
- The results depend on cohort 1's integrated space being obtained with Harmony before GEDI modeling, per the article's workflow.
- The article notes that more extensive analyses are needed to understand how factors (sample/cell counts, sequencing depth) influence GEDI's performance, which could affect classifier generalization.

## Evidence

- [methods] top-20 parameter selection: "top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20]"
- [discussion] Harmony space for cohort 1: "we used the integrated space identified by Harmony41 for the samples in cohort 1 of the COVID-19 dataset"
- [abstract] ML prediction of sample characteristics: "GEDI enables cross-sample cell state mapping on par with state-of-the-art integration methods, cluster-free differential gene expression analysis"
- [readme] Zenodo models: [evidence span withheld — anchor status: external_doc]
