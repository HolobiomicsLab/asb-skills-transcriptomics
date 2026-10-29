---
name: gene-subset-model-retraining
description: Use when you must determine whether a reduced or stratified gene feature
  set (e.g., pan-cancer-common vs. dataset-unique DEGs) retains a trained malignant-vs-normal
  classifier's performance.
when_to_use_negative:
- The goal is to identify malignant cells in new scRNA-seq/ST data with the existing
  model — use scMalignantFinder's predict workflow instead of retraining.
- The training labels are not curated malignant/normal annotations (e.g., cells lack
  'Malignant'/'Tumor' vs 'Normal' labels in adata.obs['Raw_annotation']); retraining
  would inherit unreliable labels.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scMalignantFinder
  role: Source of the pretrained logistic-regression classifier, 2707-gene DEG feature
    set, training data, and evaluation datasets; retrained with model_method='LogisticRegression'
    and norm_type=True (total-count normalization to 1e4).
  repo: github.com/Jonyyqn/scMalignantFinder
- name: ikarus
  role: Comparable logistic-classifier tumor/normal method referenced when positioning
    the retraining benchmark against existing supervised approaches.
  repo: BIMSBbioinfo/ikarus
- name: PreCanCell
  role: Ensemble k-NN method using commonly identified DEGs across cancer datasets;
    a comparator motivating the common-vs-unique DEG importance question.
  repo: WangX-Lab/PreCanCell
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:220cf9adab7fde41216602b683acaefe8009ed62f29db68ddada46b29b74fc87
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  tools:
  - PreCanCell
  - ikarus
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 20
---

# Gene-subset model retraining

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrains a pretrained logistic-regression classifier (scMalignantFinder, 2707-gene DEG feature set) on gene subsets — common DEGs (≥2 cancer types), dataset-unique DEGs, or top-k highest-coefficient genes — to test whether the subset preserves classification performance (AUROC, balanced accuracy).

## When to use

Use when you must determine whether a reduced or stratified gene feature set (e.g., pan-cancer-common vs. dataset-unique DEGs) retains a trained malignant-vs-normal classifier's performance.

## When NOT to use

- The goal is to identify malignant cells in new scRNA-seq/ST data with the existing model — use scMalignantFinder's predict workflow instead of retraining.
- The training labels are not curated malignant/normal annotations (e.g., cells lack 'Malignant'/'Tumor' vs 'Normal' labels in adata.obs['Raw_annotation']); retraining would inherit unreliable labels.

## Inputs

- model.joblib — pretrained logistic-regression classifier from Zenodo record 17888140
- ordered_feature.tsv — ordered list of the 2707 model features
- combined_tumor_up_down_degs.txt — combined malignant-vs-normal DEG feature list
- combine_training.h5ad — training data with cell labels in adata.obs['Raw_annotation']
- test_cancerCellLine.h5ad — malignant cells from a cancer cell line (test set)
- test_TabulaSapiens.h5ad — normal epithelial cells from healthy tissue (test set)

## Outputs

- Per-gene logistic-regression coefficient/importance table aligned to ordered_feature.tsv
- DEG grouping: common DEGs (present in ≥2 cancer types) vs. dataset-unique DEGs
- Performance comparison (AUROC, balanced accuracy) for full vs. top-1500 vs. common-DEG-only vs. unique-DEG-only models, e.g. deg_importance_performance.csv

## How to apply

Load the pretrained logistic-regression model from model.joblib with joblib and align its per-gene coefficients to ordered_feature.tsv to obtain feature importances. Parse combined_tumor_up_down_degs.txt and split the 2707 DEGs into common DEGs (differential in ≥2 cancer types) and dataset-unique DEGs. Retrain logistic-regression classifiers (model_method='LogisticRegression', norm_type=True) on combine_training.h5ad, taking labels from adata.obs['Raw_annotation'] with 'Malignant'/'Tumor' as malignant and 'Normal' as normal, using (a) the top-1500 highest-importance genes, (b) common DEGs only, and (c) unique DEGs only. Evaluate each retrained model against the full 2707-gene model on test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad, recording AUROC and balanced accuracy to a table such as deg_importance_performance.csv.

## Related tools

- **scMalignantFinder** (Source of the pretrained logistic-regression classifier, 2707-gene DEG feature set, training data, and evaluation datasets; retrained with model_method='LogisticRegression' and norm_type=True (total-count normalization to 1e4).) — github.com/Jonyyqn/scMalignantFinder
- **ikarus** (Comparable logistic-classifier tumor/normal method referenced when positioning the retraining benchmark against existing supervised approaches.) — BIMSBbioinfo/ikarus
- **PreCanCell** (Ensemble k-NN method using commonly identified DEGs across cancer datasets; a comparator motivating the common-vs-unique DEG importance question.) — WangX-Lab/PreCanCell

## Examples

```
import joblib; clf = joblib.load('model.joblib'); feats = [l.strip() for l in open('ordered_feature.tsv')]; coefs = dict(zip(feats, clf.coef_[0]))
```

## Evaluation signals

- The common-DEG-only and top-1500-gene models show AUROC and balanced accuracy comparable to the full 2707-gene model on test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad.
- The unique-DEG-only model shows a significant drop in AUROC/balanced accuracy relative to the full model.
- Coefficient/importance distributions confirm common DEGs receive higher logistic-regression coefficients than unique DEGs.
- The DEG split accounts for all 2707 genes with no overlap between common and unique groups.
- Coefficients from model.joblib align 1:1 in order with ordered_feature.tsv.

## Limitations

- The feature importance is only directly interpretable for linear models; coefficient extraction assumes the pretrained classifier is logistic regression.
- Results are specific to the training corpus (combine_training.h5ad) and its two test sets; generalization to other cancer types or normal tissues is not established by this procedure.
- Retraining requires accurately labeled malignancy status ('Malignant'/'Tumor' vs 'Normal'); performance is otherwise compromised by suboptimal reference labels.

## Evidence

- [results] A model built solely from common DEGs performed comparably to the original model (similar AUROC and balanced accuracy), whereas the model built from unique DEGs exhibited significantly poorer performance.: [evidence span withheld — anchor status: not_found]
