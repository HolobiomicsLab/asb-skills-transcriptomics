---
name: bulk-tumor-latent-feature-extraction
description: Use when a pre-trained Miscell checkpoint is available and bulk tumor
  expression (e.g., TCGA pan-cancer) should be encoded into latent features and tested
  for association with overall survival adjusting for Age, Sex, and TNM stage.
when_to_use_negative:
- Input is single-cell data without a pre-trained checkpoint — Miscell training requires
  at least two GPUs and large sample size, and this skill concerns transferring to
  bulk data
- Bulk dataset is too small — 'Miscell demands a large sample size to obtain good
  performance'
- Batch effects are present and uncorrected — Miscell does not include batch correction,
  so it must be handled as a preprocessing step before feature extraction
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Miscell
  role: Pre-trained self-supervised densenet encoder used to extract latent feature
    representations from bulk tumor expression data
  repo: github.com/lixiangchun/Miscell
- name: pandas
  role: Joins the latent feature matrix to the TCGA clinical table (OS time/status,
    Age, Sex, TNM stage) by TCGA sample barcode
- name: numpy
  role: Loads the expression matrix from the .npz archive (X = np.load(...)['x'])
    for feature extraction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:97dc38e61cf803979ee63288c1b9d56f832b6076c3bd015191ef5e7ab713c2ca
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# bulk-tumor-latent-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies a pre-trained Miscell self-supervised encoder (deep neural network latent feature extractor) to TCGA pan-cancer bulk tumor expression data to transfer knowledge learned from single cells into bulk-sample latent features, then tests their association with overall survival (Kaplan-Meier, log-rank, Cox proportional hazards).

## When to use

Use when a pre-trained Miscell checkpoint is available and bulk tumor expression (e.g., TCGA pan-cancer) should be encoded into latent features and tested for association with overall survival adjusting for Age, Sex, and TNM stage.

## When NOT to use

- Input is single-cell data without a pre-trained checkpoint — Miscell training requires at least two GPUs and large sample size, and this skill concerns transferring to bulk data
- Bulk dataset is too small — 'Miscell demands a large sample size to obtain good performance'
- Batch effects are present and uncorrected — Miscell does not include batch correction, so it must be handled as a preprocessing step before feature extraction

## Inputs

- TCGA pan-cancer bulk tumor expression matrix (e.g., HVG-normalized .npz with key 'x', matching the encoder's in_features dimension)
- Pre-trained Miscell model checkpoint (e.g., checkpoint_0199.pth.tar for a densenet arch)
- TCGA clinical table with overall survival time/status, Age, Sex, and TNM stage, keyed by TCGA sample barcode

## Outputs

- Latent feature matrix (64 features per tumor sample, aligned to TCGA barcodes)
- Annotated Kaplan-Meier overall survival curve image (km_curve_feature54.png)
- Log-rank test p-value for the 54th latent feature (expected ~0.004)
- Table of covariate-adjusted significant features (cox_significant_features.csv)

## How to apply

Load the TCGA pan-cancer bulk expression matrix and the pre-trained Miscell encoder (e.g., a densenet arch loaded via utils.load_pretrained_model with return_feature=True, asserting in_features equals the HVG dimension of the input, e.g. 1686), then run utils.extract_features to obtain the moco-dim 64 latent feature values per tumor sample. Join the latent features to the TCGA clinical table (OS time and status, Age, Sex, TNM stage) by TCGA sample barcode with pandas. Split tumors into high/low groups by the 54th latent feature, fit Kaplan-Meier overall survival curves for each group, and run a log-rank test to reproduce the reported p = 0.004, saving the annotated curve as km_curve_feature54.png. Finally, fit a Cox proportional hazards model of overall survival on each latent feature adjusted for Age, Sex and TNM stage, and compile the 17 significant features (including the 54th) into cox_significant_features.csv.

## Related tools

- **Miscell** (Pre-trained self-supervised densenet encoder used to extract latent feature representations from bulk tumor expression data) — github.com/lixiangchun/Miscell
- **pandas** (Joins the latent feature matrix to the TCGA clinical table (OS time/status, Age, Sex, TNM stage) by TCGA sample barcode)
- **numpy** (Loads the expression matrix from the .npz archive (X = np.load(...)['x']) for feature extraction)

## Examples

```
model = utils.load_pretrained_model(arch, in_features, 'checkpoint_0199.pth.tar', return_feature=True); features = utils.extract_features(model, np.load('data/x_new_scanpy_HVG.npz')['x'])
```

## Evaluation signals

- in_features assertion passes: in_features equals X.shape[1] of the loaded expression matrix (e.g., 1686 HVGs)
- Log-rank p-value for the 54th latent feature reproduces the reported p = 0.004
- Cox significant-feature list (adjusted for Age, Sex, TNM stage) matches the paper's reported listing (54th feature plus 16 others)
- Barcode join between latent features and clinical table yields one clinical record per tumor sample with no unmatched barcodes
- KM curve image km_curve_feature54.png shows two survival curves for the high/low feature-54 groups with annotation

## Limitations

- Miscell demands a large sample size to obtain good performance, limiting use on small datasets
- Miscell does not internally include any batch-correction methods; batch effects must be corrected before running Miscell
- The clinically significant latent features extracted by Miscell lack explainability; biological mechanisms underlying the latent features remain unknown and require future exploration

## Evidence

- [intro] Transfer of single-cell knowledge to bulk data: "transferring knowledge learned from single-cells to extract biologically meaningful features from bulk tumor expression data."
- [readme] Feature extraction with pre-trained model (README): "model = utils.load_pretrained_model(arch, in_features, checkpoint, return_feature=True)"
- [discussion] Large sample size requirement: "Analogous to methods built upon deep neural networks, Miscell demands a large sample size to obtain good performance."
- [discussion] Explainability of clinical latent features: "The clinically significant latent feature extracted by Miscell is lack of explainability and future works are required to address this issue."
