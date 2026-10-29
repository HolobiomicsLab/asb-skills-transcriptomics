---
name: logistic-regression-coefficient-extraction
description: Use when you need per-gene importance from the 2707-DEG scMalignantFinder
  model (model.joblib) or must verify that a reduced feature set (common DEGs, top-1500
  genes) matches full-model AUROC/balanced accuracy.
when_to_use_negative:
- The classifier is non-linear (e.g. random forest, k-NN as in PreCanCell) so raw
  coefficients are not interpretable per-gene importances — use model-native importance
  measures instead.
- You already possess a per-gene importance table and only need to retrain on a feature
  subset — skip straight to retraining on combine_training.h5ad.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMalignantFinder
  role: Supplies the pretrained logistic-regression classifier (model.joblib), ordered
    feature list, DEG lists, and training/test h5ad datasets used for coefficient
    extraction and retraining
  repo: github.com/Jonyyqn/scMalignantFinder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:ff7d972a257adc7fba6bfce0fb07e25dddf7b876116271f4ea2b7cf2471e4cd6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# logistic-regression-coefficient-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts per-gene feature importance from scMalignantFinder's pretrained logistic-regression classifier by reading model.joblib coefficients aligned to ordered_feature.tsv, then tests whether DEG subsets (common across ≥2 cancer types vs dataset-unique, or the top-1500 genes) preserve full-model classification performance.

## When to use

Use when you need per-gene importance from the 2707-DEG scMalignantFinder model (model.joblib) or must verify that a reduced feature set (common DEGs, top-1500 genes) matches full-model AUROC/balanced accuracy.

## When NOT to use

- The classifier is non-linear (e.g. random forest, k-NN as in PreCanCell) so raw coefficients are not interpretable per-gene importances — use model-native importance measures instead.
- You already possess a per-gene importance table and only need to retrain on a feature subset — skip straight to retraining on combine_training.h5ad.

## Inputs

- model.joblib (pretrained logistic-regression classifier, Zenodo 17888140)
- ordered_feature.tsv (ordered 2707-gene feature list)
- combined_tumor_up_down_degs.txt (combined tumor up/down DEG list)
- combine_training.h5ad (signature-calibrated training data with adata.obs['Raw_annotation'])
- test_cancerCellLine.h5ad (malignant cells from a cancer cell line)
- test_TabulaSapiens.h5ad (normal epithelial cells from healthy tissue)

## Outputs

- Per-gene coefficient/importance table aligned to ordered_feature.tsv
- DEG grouping: common (≥2 cancer types) vs unique DEGs
- Model performance comparison (AUROC, balanced accuracy) for top-1500, common-DEG-only, unique-DEG-only vs full 2707-gene model
- deg_importance_performance.csv

## How to apply

Download model.joblib, ordered_feature.tsv, combined_tumor_up_down_degs.txt, and combine_training.h5ad from zenodo.org/records/17888140. Load the pretrained logistic-regression classifier with joblib and map its coefficients onto genes via ordered_feature.tsv. Parse combined_tumor_up_down_degs.txt to split the 2707 DEGs into common DEGs (differentially expressed in ≥2 cancer types) and unique (cancer-type-specific) DEGs, and compare the two coefficient distributions. Retrain logistic-regression classifiers (model_method='LogisticRegression', norm_type=True, total-count normalization to target_sum=1e4) on combine_training.h5ad using labels from adata.obs['Raw_annotation'] with 'Malignant'/'Tumor' as malignant and 'Normal' as normal, restricted to (a) top-1500 highest-importance genes, (b) common DEGs only, and (c) unique DEGs only. Evaluate each retrained model against the full 2707-gene model on test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad; the rationale is that common DEGs should carry higher importance than unique DEGs, so common-DEG-only and top-1500 models should be comparable to the full model while the unique-DEG model degrades significantly.

## Related tools

- **scMalignantFinder** (Supplies the pretrained logistic-regression classifier (model.joblib), ordered feature list, DEG lists, and training/test h5ad datasets used for coefficient extraction and retraining) — github.com/Jonyyqn/scMalignantFinder

## Examples

```
import joblib; clf = joblib.load('model.joblib'); coefs = clf.coef_[0]; genes = [l.split()[0] for l in open('ordered_feature.tsv')]; imp = dict(zip(genes, coefs))
```

## Evaluation signals

- Extracted coefficients are 1:1 aligned to ordered_feature.tsv (2707 genes, no missing/mismatched features).
- Coefficient/importance distribution of common DEGs (≥2 cancer types) is shifted higher than that of unique DEGs.
- Top-1500-gene and common-DEG-only retrained models show AUROC and balanced accuracy comparable to the full 2707-gene model on test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad.
- Unique-DEG-only model shows a significant performance drop relative to the full model, matching the reported finding.
- Retraining uses norm_type=True with sc.pp.normalize_total(adata, target_sum=1e4) and labels 'Malignant'/'Tumor' as malignant and 'Normal' as normal.

## Limitations

- The pretrained scMalignantFinder classifier requires prerequisite identification of basic cell types and is designed to identify malignant cells only from cancerous lineages.
- Capabilities are untested for cancers with unknown or controversial origins (e.g., synovial sarcoma and certain brain tumors).
- Coefficient-based importance is only valid for the linear logistic-regression model; swapping model_method invalidates the interpretation.

## Evidence
