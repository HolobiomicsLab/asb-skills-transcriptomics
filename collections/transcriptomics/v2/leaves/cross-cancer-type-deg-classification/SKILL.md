---
name: cross-cancer-type-deg-classification
description: Use when building or auditing a pan-cancer malignant-vs-normal single-cell
  classifier from unioned cross-dataset DEG features, and you need to quantify whether
  cross-cancer-type common DEGs (≥2 cancer types) drive performance versus cancer-type-unique
  DEGs.
when_to_use_negative:
- The goal is to annotate malignant cells in tissues of unknown or controversial origin
  (e.g. synovial sarcoma, certain brain tumors) — the paper states these capabilities
  require further testing.
- Input data are not single-cell/spatial transcriptomics of cancerous lineages with
  prerequisite basic cell-type identification; scMalignantFinder is designed for malignant
  cells within cancerous lineages only.
- The analysis goal is CNV-based malignancy inference (e.g. CopyKAT/inferCNV-style)
  rather than supervised DEG-feature classification.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scMalignantFinder
  role: Provides the pretrained logistic-regression classifier, DEG feature list,
    and training data used to extract coefficients and retrain common/unique/top-1500-DEG
    models
  repo: https://github.com/Jonyyqn/scMalignantFinder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:b2977ba6dca93624b12bd45c60ca5c35ff8a9c1c045287f180cb76e2b1f0273e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  repo_url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-cancer-type-deg-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains logistic-regression malignant-cell classifiers on the 2707-DEG feature set from multiple cancer types (scMalignantFinder) and tests whether DEGs common to ≥2 cancer types — rather than cancer-type-unique DEGs — carry the classification performance, using per-gene logistic-regression coefficients as importance.

## When to use

Use when building or auditing a pan-cancer malignant-vs-normal single-cell classifier from unioned cross-dataset DEG features, and you need to quantify whether cross-cancer-type common DEGs (≥2 cancer types) drive performance versus cancer-type-unique DEGs.

## When NOT to use

- The goal is to annotate malignant cells in tissues of unknown or controversial origin (e.g. synovial sarcoma, certain brain tumors) — the paper states these capabilities require further testing.
- Input data are not single-cell/spatial transcriptomics of cancerous lineages with prerequisite basic cell-type identification; scMalignantFinder is designed for malignant cells within cancerous lineages only.
- The analysis goal is CNV-based malignancy inference (e.g. CopyKAT/inferCNV-style) rather than supervised DEG-feature classification.

## Inputs

- model.joblib — pretrained logistic-regression classifier (Zenodo record 17888140)
- ordered_feature.tsv — ordered feature gene list
- combined_tumor_up_down_degs.txt — the 2707 up/down DEG feature list across cancer types
- combine_training.h5ad — signature-calibrated training set with Raw_annotation labels ('Malignant'/'Tumor' vs 'Normal')
- test_cancerCellLine.h5ad — malignant cells from a cancer cell line (test set)
- test_TabulaSapiens.h5ad — normal epithelial cells from healthy tissue (test set)

## Outputs

- Per-gene logistic-regression coefficient/importance table aligned to ordered_feature.tsv
- DEG grouping table separating common (≥2 cancer types) vs unique DEGs
- deg_importance_performance.csv — coefficient table, DEG grouping, and model performance comparison (AUROC, balanced accuracy) for full, top-1500, common-DEG-only, and unique-DEG-only models

## How to apply

Download model.joblib, ordered_feature.tsv, combined_tumor_up_down_degs.txt, and combine_training.h5ad from the Zenodo deposit (zenodo.org/records/17888140). Load the pretrained logistic-regression classifier with joblib and extract per-gene importance as coefficients aligned to ordered_feature.tsv; parse combined_tumor_up_down_degs.txt to split the 2707 DEGs into common (DE in ≥2 cancer types) and unique (cancer-type-specific) groups. Retrain logistic-regression classifiers on combine_training.h5ad (model_method='LogisticRegression', norm_type=True so counts are normalized via sc.pp.normalize_total to target_sum=1e4; labels from adata.obs['Raw_annotation'] with 'Malignant'/'Tumor' as malignant and 'Normal' as normal) using (a) the top-1500 highest-importance genes, (b) common DEGs only, and (c) unique DEGs only. Evaluate each retrained model against the full 2707-gene model on test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad by AUROC and balanced accuracy. The rationale is that the union of per-dataset DEGs taken as model features may be dominated by common genes, so partitioning by cross-cancer-type recurrence isolates the source of predictive signal.

## Related tools

- **scMalignantFinder** (Provides the pretrained logistic-regression classifier, DEG feature list, and training data used to extract coefficients and retrain common/unique/top-1500-DEG models) — https://github.com/Jonyyqn/scMalignantFinder

## Examples

```
import joblib, pandas as pd; clf = joblib.load('model.joblib'); feats = pd.read_csv('ordered_feature.tsv', sep='\t'); coefs = clf.coef_  # align to feats
```

## Evaluation signals

- Common-DEG-only model reproduces AUROC and balanced accuracy comparable to the full 2707-gene model on test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad
- Top-1500 highest-importance-gene model is also comparable to the full model
- Unique-DEG-only model shows a significant performance drop versus the full model
- Coefficient/importance distributions show common DEGs receive higher importance than unique DEGs
- All retrained models are trained with norm_type=True (sc.pp.normalize_total, target_sum=1e4) and identical labels from adata.obs['Raw_annotation']

## Limitations

- Unique (cancer-type-specific) DEGs alone are insufficient: a model built only from unique DEGs exhibits significantly poorer AUROC and balanced accuracy.
- The classifier relies on prerequisite identification of basic cell types and is restricted to malignant cells within cancerous lineages.
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors.

## Evidence
