---
name: deg-feature-importance-analysis
description: Use when a pretrained malignant-cell classifier's feature list must be
  decomposed into common vs dataset-unique DEGs and their contribution to classification
  performance (AUROC, balanced accuracy) needs to be quantified.
when_to_use_negative:
- The goal is de novo DEG detection from raw scRNA-seq rather than re-analyzing the
  precomputed 2707-gene DEG set already deposited on Zenodo.
- The task is annotating malignant cells in new data (use scMalignantFinder's predict
  workflow directly) rather than interpreting feature importance of the trained model.
- The cancer type has unknown or controversial origins (e.g., synovial sarcoma, certain
  brain tumors), where the DEG features and classifier are untested.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMalignantFinder
  role: Provides the pretrained logistic-regression model, feature lists, training
    h5ad, and test h5ad files used for coefficient extraction and subset-model retraining
  repo: github.com/Jonyyqn/scMalignantFinder
- name: ikarus
  role: Comparable tumor-vs-normal logistic classifier using a tumor-specific gene
    set as feature reference for feature-importance framing
  repo: BIMSBbioinfo/ikarus
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:c4955bc15ee7cda07ac1ba9e85ad31aede7d60a3c5a4fb7576d5af102fc417bd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - ikarus
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# deg-feature-importance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts per-gene logistic-regression coefficients from the pretrained scMalignantFinder model (model.joblib, 2707 DEG features) to test whether pan-cancer common DEGs (DE in ≥2 cancer types) carry higher importance than dataset-unique DEGs, and retrains subset models to confirm common-DEG and top-1500-gene models match the full model.

## When to use

Use when a pretrained malignant-cell classifier's feature list must be decomposed into common vs dataset-unique DEGs and their contribution to classification performance (AUROC, balanced accuracy) needs to be quantified.

## When NOT to use

- The goal is de novo DEG detection from raw scRNA-seq rather than re-analyzing the precomputed 2707-gene DEG set already deposited on Zenodo.
- The task is annotating malignant cells in new data (use scMalignantFinder's predict workflow directly) rather than interpreting feature importance of the trained model.
- The cancer type has unknown or controversial origins (e.g., synovial sarcoma, certain brain tumors), where the DEG features and classifier are untested.

## Inputs

- model.joblib (pretrained logistic-regression classifier from zenodo.org/records/17888140)
- ordered_feature.tsv (ordered 2707-gene feature list)
- combined_tumor_up_down_degs.txt (up/down DEG list per cancer type)
- combine_training.h5ad (training AnnEx/h5ad with obs['Raw_annotation'])
- test_cancerCellLine.h5ad (malignant cells from a cancer cell line)
- test_TabulaSapiens.h5ad (normal epithelial cells from healthy tissue)

## Outputs

- deg_importance_performance.csv (coefficient table, DEG grouping, model performance comparison)
- Per-gene logistic-regression coefficient/importance values aligned to ordered_feature.tsv
- Common vs unique DEG grouping of the 2707 features
- AUROC and balanced-accuracy metrics for top-1500, common-DEG-only, unique-DEG-only, and full models

## How to apply

Download model.joblib, ordered_feature.tsv, combined_tumor_up_down_degs.txt, and combine_training.h5ad from Zenodo record 17888140. Load the pretrained logistic-regression classifier with joblib and align its coefficients to the gene order in ordered_feature.tsv. Parse combined_tumor_up_down_degs.txt to split the 2707 DEGs into common DEGs (differential in ≥2 cancer types) and unique (cancer-type-specific) DEGs, then compare the coefficient distributions of the two groups. Retrain logistic-regression classifiers (model_method='LogisticRegression', norm_type=True applying sc.pp.normalize_total(adata, target_sum=1e4), labels from adata.obs['Raw_annotation'] with 'Malignant'/'Tumor' as malignant and 'Normal' as normal) on (a) the top-1500 highest-importance genes, (b) common DEGs only, and (c) unique DEGs only. Evaluate each subset model against the full 2707-gene model on test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad using AUROC and balanced accuracy.

## Related tools

- **scMalignantFinder** (Provides the pretrained logistic-regression model, feature lists, training h5ad, and test h5ad files used for coefficient extraction and subset-model retraining) — github.com/Jonyyqn/scMalignantFinder
- **ikarus** (Comparable tumor-vs-normal logistic classifier using a tumor-specific gene set as feature reference for feature-importance framing) — BIMSBbioinfo/ikarus

## Examples

```
import joblib; model = joblib.load('model.joblib'); coefs = dict(zip(features, model.coef_[0]))
```

## Evaluation signals

- The common-DEG-only model achieves AUROC and balanced accuracy comparable to the full 2707-gene model on test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad.
- The unique-DEG-only model shows a significant drop in AUROC and balanced accuracy relative to the full model.
- The top-1500 highest-importance gene model reproduces full-model performance.
- Common DEGs receive systematically higher coefficient/importance values than unique DEGs in the distribution comparison.
- Splitting combined_tumor_up_down_degs.txt into common (≥2 cancer types) and unique groups accounts for all 2707 features with no overlap or loss.

## Limitations

- Coefficients reflect a single logistic-regression fit on the signature-calibrated training set; importance conclusions are tied to that model and the 2707-DEG union of per-dataset DEGs.
- The original feature selection (union of DEGs across datasets) may overlook substantial inter-tumor heterogeneity, so unique-DEG importance could be underestimated for specific cancer types.
- The analysis depends on files deposited on Zenodo (record 17888140); labels must be mapped from adata.obs['Raw_annotation'] with 'Malignant'/'Tumor' vs 'Normal' conventions.

## Evidence

- [readme] Logistic classifier over gene-set features in a comparable tool: [evidence span withheld — anchor status: external_doc]
