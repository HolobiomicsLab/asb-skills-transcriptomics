---
name: cox-proportional-hazards-covariate-adjustment
description: Use when you have per-sample features (e.g. Miscell latent features computed
  from TCGA bulk expression) joined to overall survival time/status plus confounders
  such as Age, Sex and TNM stage, and need covariate-adjusted prognostic associations
  per feature.
when_to_use_negative:
- Input is raw single-cell data on a small dataset — Miscell demands a large sample
  size to obtain good performance, so latent features suitable for survival modeling
  cannot be learned reliably
- The single-cell or bulk data have uncorrected batch effects — Miscell does not internally
  include batch correction and requires preprocessing beforehand
- The goal is biological interpretation of a prognostic feature — clinically significant
  Miscell latent features lack explainability in this framework
edam_operation: http://edamontology.org/operation_0301
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Miscell
  role: Pre-trained self-supervised encoder applied to bulk tumor expression to extract
    the latent features (e.g. the 54th) tested in the Cox models
  repo: https://github.com/lixiangchun/Miscell
- name: pandas
  role: Joins latent feature table to the TCGA clinical table (survival time/status,
    Age, Sex, TNM stage) by sample barcode and assembles the Cox model dataframe
- name: numpy
  role: Holds the feature matrix (e.g. x_new_scanpy_HVG.npz) passed through the encoder
    to compute latent features
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:e6b8961e73839774474f2e650e13c3a34e1d1fe574ed5a2ffded7e77cd048cd1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  repo_url: https://github.com/lixiangchun/Miscell
  tools:
  - Miscell
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0301
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cox-proportional-hazards-covariate-adjustment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tests whether latent features learned by Miscell from single-cell data are prognostic in TCGA pan-cancer bulk tumor expression data, by fitting Cox proportional hazards models of overall survival adjusted for Age, Sex, and TNM stage, complemented by Kaplan-Meier curves and log-rank tests (e.g. the 54th latent feature, log-rank p = 0.004).

## When to use

Use when you have per-sample features (e.g. Miscell latent features computed from TCGA bulk expression) joined to overall survival time/status plus confounders such as Age, Sex and TNM stage, and need covariate-adjusted prognostic associations per feature.

## When NOT to use

- Input is raw single-cell data on a small dataset — Miscell demands a large sample size to obtain good performance, so latent features suitable for survival modeling cannot be learned reliably
- The single-cell or bulk data have uncorrected batch effects — Miscell does not internally include batch correction and requires preprocessing beforehand
- The goal is biological interpretation of a prognostic feature — clinically significant Miscell latent features lack explainability in this framework

## Inputs

- TCGA pan-cancer bulk tumor expression matrix
- Pre-trained Miscell encoder checkpoint (e.g. checkpoint_0199.pth.tar)
- TCGA clinical table with overall survival time/status, Age, Sex, and TNM stage
- TCGA sample barcodes for joining latent features to clinical data

## Outputs

- Latent feature values per tumor sample (e.g. 64-dim Miscell features)
- Annotated Kaplan-Meier overall survival curve (km_curve_feature54.png)
- Log-rank test p-value per high/low latent-feature split
- cox_significant_features.csv listing covariate-adjusted significant features

## How to apply

Load the TCGA pan-cancer bulk expression matrix, run it through the pre-trained Miscell encoder to compute latent features for all tumor samples, and join these to the TCGA clinical table (overall survival time and status, Age, Sex, TNM stage) by TCGA sample barcode using pandas. First screen associations without covariates: split tumors into high- and low-value groups by a chosen latent feature (e.g. the 54th), fit Kaplan-Meier overall survival curves for each group, and run a log-rank test between them to reproduce the reported association (p = 0.004), saving the annotated curve as km_curve_feature54.png. Then fit a Cox proportional hazards model of overall survival on each latent feature adjusted for Age, Sex and TNM stage — the covariate adjustment is the rationale for isolating each feature's prognostic contribution beyond standard clinical confounders. Record which features remain significant after adjustment (the 54th latent feature and 16 others in the paper) and compile them into cox_significant_features.csv.

## Related tools

- **Miscell** (Pre-trained self-supervised encoder applied to bulk tumor expression to extract the latent features (e.g. the 54th) tested in the Cox models) — https://github.com/lixiangchun/Miscell
- **pandas** (Joins latent feature table to the TCGA clinical table (survival time/status, Age, Sex, TNM stage) by sample barcode and assembles the Cox model dataframe)
- **numpy** (Holds the feature matrix (e.g. x_new_scanpy_HVG.npz) passed through the encoder to compute latent features)

## Examples

```
features = utils.extract_features(model, X); df = pd.concat([features, clinical[['OS_time','OS_status','Age','Sex','TNM']]], axis=1)  # then fit Cox per latent feature adjusted for Age, Sex, TNM
```

## Evaluation signals

- The 54th latent feature's log-rank p-value reproduces the reported p = 0.004 between high/low groups
- The covariate-adjusted significant-feature list (54th latent feature plus 16 others, i.e. 17 features) matches the paper's reported listing
- The barcode join between latent features and the TCGA clinical table yields no missing survival time/status or covariate values
- Kaplan-Meier curve file km_curve_feature54.png is produced with high/low groups split by the 54th latent feature and the log-rank p-value annotated
- cox_significant_features.csv contains one row per significant feature with adjusted hazard ratios and p-values

## Limitations

- The clinically significant latent feature extracted by Miscell is lack of explainability and future works are required to address this issue
- Analogous to methods built upon deep neural networks, Miscell demands a large sample size to obtain good performance
- Miscell does not internally include any batch-correction methods; users must correct for batch effects before running the encoder
- The biological mechanisms underlying the latent features of Miscell trained on these single-cell datasets remain unknown and necessitate exploration in future study

## Evidence
