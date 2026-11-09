---
name: log-rank-survival-association-testing
description: Use when bulk tumor samples (e.g., TCGA pan-cancer) have matched overall
  survival time/status and clinical covariates (Age, Sex, TNM stage), and you need
  to test whether learned representation features are prognostic.
when_to_use_negative:
- Input is already a per-sample feature/clinical merged table with no need to recompute
  latent features from expression data
- The clinical data lack survival time/status or the required covariates (Age, Sex,
  TNM stage), so log-rank/Cox models cannot be fit
- The dataset is too small for Miscell, which demands a large sample size to obtain
  good performance
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0203
tools:
- name: Miscell
  role: Supplies the pre-trained self-supervised encoder that converts bulk tumor
    expression into latent features used as the survival-association predictors
  repo: github.com/lixiangchun/Miscell
- name: pandas
  role: Joins the latent feature table to the TCGA clinical table by sample barcode
    and assembles the Cox results table
- name: numpy
  role: Loads and handles the expression feature matrices (npz) fed to the encoder
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:f8b68f1e2cf9e8b48b109321568610a1d9b79eb9c1c0ab32a164c42e15ba82a2
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
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# log-rank-survival-association-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Test whether Miscell latent features extracted from TCGA pan-cancer bulk tumor expression data are associated with overall survival, using Kaplan-Meier curves with log-rank tests and covariate-adjusted Cox proportional hazards models.

## When to use

Use when bulk tumor samples (e.g., TCGA pan-cancer) have matched overall survival time/status and clinical covariates (Age, Sex, TNM stage), and you need to test whether learned representation features are prognostic.

## When NOT to use

- Input is already a per-sample feature/clinical merged table with no need to recompute latent features from expression data
- The clinical data lack survival time/status or the required covariates (Age, Sex, TNM stage), so log-rank/Cox models cannot be fit
- The dataset is too small for Miscell, which demands a large sample size to obtain good performance

## Inputs

- TCGA pan-cancer bulk tumor expression matrix (samples by genes)
- Pre-trained Miscell encoder checkpoint (e.g., checkpoint_0199.pth.tar)
- TCGA clinical table with overall survival time and status, Age, Sex, and TNM stage

## Outputs

- Annotated Kaplan-Meier overall survival curve image (km_curve_feature54.png) with log-rank p-value
- Significant-feature results table (cox_significant_features.csv) from covariate-adjusted Cox models

## How to apply

First compute latent feature values for all tumor samples by passing the TCGA bulk expression matrix through the pre-trained Miscell encoder (e.g., a densenet checkpoint loaded with utils.load_pretrained_model(..., return_feature=True)). Join the latent features to the TCGA clinical table (overall survival time and status, Age, Sex, TNM stage) by TCGA sample barcode using pandas. Split tumors into high- and low-value groups by the target latent feature (e.g., the 54th), fit Kaplan-Meier overall survival curves for each group, and run a log-rank test between them — the Miscell paper reports p = 0.004 for the 54th latent feature. To control for clinical confounders, fit a Cox proportional hazards model of overall survival on each latent feature adjusted for Age, Sex, and TNM stage, retaining only significant features in the results table.

## Related tools

- **Miscell** (Supplies the pre-trained self-supervised encoder that converts bulk tumor expression into latent features used as the survival-association predictors) — github.com/lixiangchun/Miscell
- **pandas** (Joins the latent feature table to the TCGA clinical table by sample barcode and assembles the Cox results table)
- **numpy** (Loads and handles the expression feature matrices (npz) fed to the encoder)

## Examples

```
model = utils.load_pretrained_model(arch, in_features, "checkpoint_0199.pth.tar", return_feature=True); features = utils.extract_features(model, X)
```

## Evaluation signals

- The 54th latent feature's log-rank p-value reproduces the reported association (p = 0.004)
- The covariate-adjusted (Age, Sex, TNM stage) significant-feature list matches the paper's reported set of the 54th latent feature plus 16 other features
- KM curve output is annotated with high/low latent-feature groups and saved as km_curve_feature54.png
- cox_significant_features.csv is produced with a schema listing each latent feature's Cox result and adjustment covariates
- Join of latent features to clinical table by TCGA sample barcode covers all tumor samples with survival data

## Limitations

- The clinically significant latent features extracted by Miscell lack explainability; the paper notes future work is required to address this
- The biological mechanisms underlying the latent features remain unknown and necessitate exploration in future study
- Miscell demands a large sample size to obtain good performance, and batch correction must be handled as a preprocessing step outside Miscell

## Evidence
