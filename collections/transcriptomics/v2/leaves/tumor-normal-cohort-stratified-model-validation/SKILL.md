---
name: tumor-normal-cohort-stratified-model-validation
description: Use when evaluating a trained RNA-seq-to-miRNA prediction model on large
  paired bulk mRNA/miRNA cohorts and you need cohort-wise accuracy (e.g., 5-fold CV
  Spearman per TCGA cancer type) rather than a single pooled score.
when_to_use_negative:
- The goal is scoring cell type-specific miRNA activity in scRNA-seq clusters (predicted
  vs observed fold-difference correlation ~0.81), not bulk cohort cross-validation
  accuracy
- Only target-based miRNA activity inference without paired training data is needed
  (e.g., miReact-style approaches that require no training cohort)
- The application sample's cellular/tissue context does not match the training cohorts
  — expect reduced accuracy; retraining on detected genes may be required instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: miRSCAPE
  role: Trains per-miRNA XGBoost models with default settings and returns out-of-fold
    predicted miRNA expression in the pred slot during cohort-stratified 5-fold cross-validation
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: bulkTransform
  role: Preprocesses bulk mRNA (default normalization) and bulk miRNA (justNorm =
    TRUE, log-transform only, no scaling) matrices before model training/validation
  repo: https://github.com/hannenhalli-lab/miRSCAPE
- name: xgboost
  role: Underlying gradient-boosting learner used by miRSCAPE with default booster
    (bstr), objective (objt), depth (mdpth), eta (ett), rounds (nrnds) and early stopping
    (esr)
- name: R
  role: Implementation environment; R 4.1.0 is required to run miRSCAPE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.104962
    title: miRSCAPE - inferring miRNA expression from scRNA-seq data
  source_content_hash: sha256:2cc8a81842c5d73b5d0f8b4f079160c36f3c19984a4e0470ba623ac3c9e99e14
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hannenhalli-lab/miRSCAPE
  repo_url: https://github.com/hannenhalli-lab/miRSCAPE
  tools:
  - R
  - bulkTransform
  - miRSCAPE
  - xgboost
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Tumor-normal cohort-stratified model validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratified cross-validation of a paired-expression prediction model (e.g., miRSCAPE trained on paired mRNA-miRNA bulk profiles) within each of ~10 TCGA tumor and normal cohorts, scoring out-of-fold predictions by cross-sample Spearman correlation per miRNA.

## When to use

Use when evaluating a trained RNA-seq-to-miRNA prediction model on large paired bulk mRNA/miRNA cohorts and you need cohort-wise accuracy (e.g., 5-fold CV Spearman per TCGA cancer type) rather than a single pooled score.

## When NOT to use

- The goal is scoring cell type-specific miRNA activity in scRNA-seq clusters (predicted vs observed fold-difference correlation ~0.81), not bulk cohort cross-validation accuracy
- Only target-based miRNA activity inference without paired training data is needed (e.g., miReact-style approaches that require no training cohort)
- The application sample's cellular/tissue context does not match the training cohorts — expect reduced accuracy; retraining on detected genes may be required instead

## Inputs

- Paired bulk mRNA expression matrix for 10 TCGA tumor and normal cohorts (~10,000 samples), rows = genes, gene names as row names
- Paired bulk miRNA expression matrix for the same cohorts, rows = miRNAs, gene names as row names
- miRSCAPE R package (source ~/miRSCAPE-main/code/miRSCAPE.R) with xgboost and Seurat dependencies (R 4.1.0)

## Outputs

- Out-of-fold predicted miRNA expression per sample and miRNA (returned in the miRSCAPE pred slot)
- miRNA-wise cross-sample Spearman accuracy table per cohort and overall (mean ~0.45, range 0.39-0.51), e.g., mirna_cv_spearman_accuracy.csv

## How to apply

Load paired bulk mRNA and bulk miRNA expression matrices for the 10 TCGA tumor and normal tissue cohorts (~10,000 samples), with genes/miRNAs as rows and gene names as row names. Preprocess both matrices with the miRSCAPE bulkTransform function, setting justNorm = TRUE for the miRNA data (log-transform only, no scaling) and default normalization for mRNA. Within each cohort, train one XGBoost model per miRNA with the miRSCAPE function using default XGBoost settings (booster bstr, objective objt, max depth mdpth, eta ett, rounds nrnds, early stopping esr), and run 5-fold cross-validation to obtain out-of-fold predictions in the pred slot. Compute cross-sample Spearman correlation between predicted and observed expression for each miRNA, then aggregate per cohort and overall; the reference run yields mean Spearman ~0.45 (range 0.39-0.51), with 58% of miRNAs at Spearman >= 0.4.

## Related tools

- **miRSCAPE** (Trains per-miRNA XGBoost models with default settings and returns out-of-fold predicted miRNA expression in the pred slot during cohort-stratified 5-fold cross-validation) — https://github.com/hannenhalli-lab/miRSCAPE
- **bulkTransform** (Preprocesses bulk mRNA (default normalization) and bulk miRNA (justNorm = TRUE, log-transform only, no scaling) matrices before model training/validation) — https://github.com/hannenhalli-lab/miRSCAPE
- **xgboost** (Underlying gradient-boosting learner used by miRSCAPE with default booster (bstr), objective (objt), depth (mdpth), eta (ett), rounds (nrnds) and early stopping (esr))
- **R** (Implementation environment; R 4.1.0 is required to run miRSCAPE)

## Examples

```
bulkk_mirna = bulkTransform(pdac_mirna, justNorm = TRUE); bulkk_mrna = bulkTransform(pdac_mrna); pred = miRSCAPE(bulkmRNA = bulkk_mrna, bulkmiRNA = bulkk_mirna, scmRNA = denem)
```

## Evaluation signals

- Per-cohort cross-sample Spearman between predicted and observed miRNA expression falls in the reported range 0.39-0.51 with overall mean ~0.45
- Approximately 58% of miRNAs achieve Spearman >= 0.4 in the accuracy table (mirna_cv_spearman_accuracy.csv)
- Out-of-fold predictions are recoverable from the miRSCAPE pred slot for every sample and miRNA in each cohort's 5-fold CV
- miRNA matrix was log-transformed only (justNorm = TRUE), not scaled, before training
- Validation is stratified: accuracy is computed within each of the 10 tumor/normal cohorts, not pooled across cohorts

## Limitations

- miRSCAPE needs to be trained on a large cohort of paired miRNA-mRNA data, unlike prior target-based approaches requiring no training
- Accuracy is slightly reduced when the cellular/tissue context of the application sample does not match the training cohort; a substantive loss is expected when contexts differ greatly (quantified in Figure S9)
- For scRNA-seq application, the large dropout rate may prevent using a pre-trained model; retraining on only reasonably detected genes may be required

## Evidence
