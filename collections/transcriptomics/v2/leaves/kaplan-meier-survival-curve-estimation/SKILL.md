---
name: kaplan-meier-survival-curve-estimation
description: Use when you have overall survival time/status for bulk tumor samples
  plus a per-sample numeric score (e.g., the 54th Miscell latent feature) and covariates
  Age, Sex, and TNM stage, and you need to test whether the score stratifies prognosis.
when_to_use_negative:
- Input samples lack overall survival time/status or clinical covariates, so no survival
  stratification or Cox adjustment is possible
- You only have single-cell data and no pre-trained Miscell encoder to transfer features
  to bulk expression — the features being tested come from the single-cell-to-bulk
  transfer
- 'You need interpretable biomarkers: the clinically significant Miscell latent features
  lack explainability'
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Miscell
  role: Pre-trained self-supervised encoder used to compute latent feature values
    for TCGA bulk tumor samples before survival stratification
  repo: github.com/lixiangchun/Miscell
- name: pandas
  role: Joins the latent feature table to the TCGA clinical table by TCGA sample barcode
    and compiles the Cox results table
- name: numpy
  role: Loads and handles the expression matrix and extracted latent feature arrays
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:0cd14e42972704f64f43e6e44b23b8803551e944f116d3f186cd7759c05fbd16
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  tools:
  - Miscell
  - numpy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# kaplan-meier-survival-curve-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates Kaplan-Meier overall survival curves for tumor sample groups stratified by a per-sample score (here, a Miscell latent feature applied to TCGA pan-cancer bulk expression) and tests group separation with a log-rank test, then confirms covariate-adjusted associations with a Cox model.

## When to use

Use when you have overall survival time/status for bulk tumor samples plus a per-sample numeric score (e.g., the 54th Miscell latent feature) and covariates Age, Sex, and TNM stage, and you need to test whether the score stratifies prognosis.

## When NOT to use

- Input samples lack overall survival time/status or clinical covariates, so no survival stratification or Cox adjustment is possible
- You only have single-cell data and no pre-trained Miscell encoder to transfer features to bulk expression — the features being tested come from the single-cell-to-bulk transfer
- You need interpretable biomarkers: the clinically significant Miscell latent features lack explainability

## Inputs

- TCGA pan-cancer bulk tumor expression matrix
- Pre-trained Miscell encoder checkpoint (e.g., checkpoint_0199.pth.tar, densenet arch, in_features matching HVG count)
- TCGA clinical table with overall survival time/status, Age, Sex, TNM stage, and TCGA sample barcodes

## Outputs

- Kaplan-Meier overall survival curves per high/low feature group (km_curve_feature54.png)
- Log-rank test p-value (reported: p = 0.004 for the 54th latent feature)
- cox_significant_features.csv — table of latent features significant in covariate-adjusted Cox models

## How to apply

Compute latent features for all TCGA pan-cancer tumor samples by passing the bulk expression matrix through the pre-trained Miscell encoder (e.g., a densenet checkpoint loaded with return_feature=True). Join the latent feature table to the TCGA clinical table (OS time, OS status, Age, Sex, TNM stage) by TCGA sample barcode with pandas. Split tumors into high- and low-value groups by the target feature (e.g., latent feature 54), fit Kaplan-Meier overall survival curves per group, and run a log-rank test between them — the reported result is p = 0.004 for feature 54. Annotate and save the curve (km_curve_feature54.png), then fit a Cox proportional hazards model of overall survival on each latent feature adjusted for Age, Sex and TNM stage to list which features remain significant.

## Related tools

- **Miscell** (Pre-trained self-supervised encoder used to compute latent feature values for TCGA bulk tumor samples before survival stratification) — github.com/lixiangchun/Miscell
- **pandas** (Joins the latent feature table to the TCGA clinical table by TCGA sample barcode and compiles the Cox results table)
- **numpy** (Loads and handles the expression matrix and extracted latent feature arrays)

## Examples

```
model = utils.load_pretrained_model(arch, in_features, checkpoint, return_feature=True); features = utils.extract_features(model, X)
```

## Evaluation signals

- The 54th latent feature's log-rank p-value reproduces the reported p = 0.004
- The covariate-adjusted Cox significant-feature list matches the paper's reported listing (17 significant features including feature 54)
- Latent feature values for all tumor samples are successfully joined to clinical rows by TCGA sample barcode with no unmatched samples
- KM curves are produced for both high- and low-value groups and saved as km_curve_feature54.png
- Cox models are adjusted for Age, Sex and TNM stage before declaring features significant

## Limitations

- The clinically significant latent feature extracted by Miscell lacks explainability; future work is required to address this
- Miscell demands a large sample size to obtain good performance, limiting use on small datasets
- The biological mechanisms underlying the latent features remain unknown and necessitate exploration in future study

## Evidence
