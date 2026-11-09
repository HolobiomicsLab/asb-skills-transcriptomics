---
name: tcga-clinical-metadata-harmonization
description: Use when bulk tumor expression features (e.g., 64-dim Miscell latent
  features) must be paired with TCGA clinical outcomes (OS time/status, Age, Sex,
  TNM stage) to test prognostic associations.
when_to_use_negative:
- Working with single-cell data alone and no bulk tumor samples with TCGA survival
  endpoints are available
- Latent feature values have not yet been extracted from bulk expression with the
  Miscell encoder — harmonization must wait until features exist
- The clinical table lacks required survival covariates (OS time/status, Age, Sex,
  TNM stage) needed for the adjusted Cox model
edam_operation: http://edamontology.org/operation_0336
tools:
- name: Miscell
  role: Pre-trained encoder used to compute latent feature values from the TCGA pan-cancer
    bulk expression matrix prior to joining with clinical metadata
  repo: github.com/lixiangchun/Miscell
- name: pandas
  role: Used to join the TCGA clinical table to the latent features by TCGA sample
    barcode and to compile the significant-feature results table
- name: numpy
  role: Supporting array handling for latent feature matrices loaded from npz files
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:3f9cb5f02eefac09ab204bbdfadd0199b2e4b78b34d48db0cc09ad923ecab796
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
  edam_operation: http://edamontology.org/operation_0336
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# tcga-clinical-metadata-harmonization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Join TCGA pan-cancer clinical survival metadata (overall survival time and status, Age, Sex, TNM stage) to Miscell latent feature values computed for tumor samples, keyed by TCGA sample barcode, enabling Kaplan-Meier and covariate-adjusted Cox survival analyses.

## When to use

Use when bulk tumor expression features (e.g., 64-dim Miscell latent features) must be paired with TCGA clinical outcomes (OS time/status, Age, Sex, TNM stage) to test prognostic associations.

## When NOT to use

- Working with single-cell data alone and no bulk tumor samples with TCGA survival endpoints are available
- Latent feature values have not yet been extracted from bulk expression with the Miscell encoder — harmonization must wait until features exist
- The clinical table lacks required survival covariates (OS time/status, Age, Sex, TNM stage) needed for the adjusted Cox model

## Inputs

- TCGA pan-cancer bulk expression matrix for tumor samples
- Pre-trained Miscell encoder checkpoint (e.g., checkpoint_0199.pth.tar)
- TCGA clinical table with overall survival time, survival status, Age, Sex, and TNM stage
- TCGA sample barcodes linking expression/latent features to clinical rows

## Outputs

- Clinical-annotated latent feature table (samples x latent features + survival covariates)
- Kaplan-Meier overall survival curves for high/low latent-feature groups (km_curve_feature54.png)
- Log-rank test p-value (reported p = 0.004 for the 54th latent feature)
- Table of significant features from covariate-adjusted Cox models (cox_significant_features.csv)

## How to apply

Load the TCGA pan-cancer bulk expression matrix, run it through the pre-trained Miscell encoder to compute latent feature values for all tumor samples, then load the TCGA clinical table (overall survival time and status, Age, Sex, TNM stage) and join the two tables by TCGA sample barcode with pandas. Split tumors into high- and low-value groups by a chosen latent feature (e.g., the 54th) and fit Kaplan-Meier overall survival curves per group, running a log-rank test between groups. Next, fit a Cox proportional hazards model of overall survival on each latent feature adjusted for Age, Sex and TNM stage, recording which features remain significant. The rationale is that transferring knowledge learned from single-cells to bulk tumor expression data yields features whose prognostic value can only be assessed once clinical endpoints are correctly aligned to sample barcodes.

## Related tools

- **Miscell** (Pre-trained encoder used to compute latent feature values from the TCGA pan-cancer bulk expression matrix prior to joining with clinical metadata) — github.com/lixiangchun/Miscell
- **pandas** (Used to join the TCGA clinical table to the latent features by TCGA sample barcode and to compile the significant-feature results table)
- **numpy** (Supporting array handling for latent feature matrices loaded from npz files)

## Examples

```
merged = pd.merge(latent_features, clinical, left_on='sample_barcode', right_on='submitter_id', how='inner')
```

## Evaluation signals

- The merge on TCGA sample barcode produces no (or minimal) dropped tumor samples — verify row counts of the joined table against the latent feature matrix
- The 54th latent feature's log-rank p-value reproduces the reported p = 0.004
- The covariate-adjusted (Age, Sex, TNM stage) significant-feature list matches the paper's reported listing (the 54th latent feature plus 16 other features)
- Output files km_curve_feature54.png and cox_significant_features.csv are produced with the expected schema
- Kaplan-Meier curves are split into exactly two groups (high vs. low value of the 54th latent feature) with annotated survival curves

## Limitations

- Clinically significant latent features extracted by Miscell lack explainability; future work is required to address this
- The biological mechanisms underlying the latent features trained on single-cell datasets remain unknown and necessitate exploration in future study
- Miscell demands a large sample size to obtain good performance, which limits use on small datasets

## Evidence

- [methods] Load the TCGA clinical table (overall survival time and status, Age, Sex, TNM stage) and join it to the latent features by TCGA sample barcode with pandas.: [evidence span withheld — anchor status: not_found]
