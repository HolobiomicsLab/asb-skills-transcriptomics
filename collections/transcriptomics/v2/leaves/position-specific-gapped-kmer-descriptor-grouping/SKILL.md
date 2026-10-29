---
name: position-specific-gapped-kmer-descriptor-grouping
description: Use when a combined feature table mixes several descriptor families (e.g.,
  position-specific gapped k-mer and position-specific k-mer modes plus GC-content)
  and you need per-group Random Forest feature selection rather than ranking all features
  jointly.
when_to_use_negative:
- Input is already a small, finalized feature vector — group-wise selection of top-100
  per mode would be redundant
- Features come from a single homogeneous descriptor family with no natural grouping,
  so joint (not group-wise) importance ranking is appropriate
- The downstream classifiers are frequency-based models combined with position-specific
  features — the article notes position-specific features do not work well in conjunction
  with existing frequency-based counterparts in single models
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: scikit-learn
  role: Provides the RandomForestClassifier whose Gini (mean-decrease-in-impurity)
    importance is used to rank features within each descriptor-mode group
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: pandas
  role: Loads All_Features_Dataset.csv, separates the four descriptor modes / GC-content
    / labels, and writes selected_feature_indices.csv
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: numpy
  role: Array handling for per-group feature importance matrices and index bookkeeping
    during selection
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: iRNA5hmC-PS
  role: The originating framework whose group-wise feature importance calculation
    strategy this skill reconstructs
  repo: https://github.com/zahid6454/iRNA5hmC-PS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2020.10.032
    title: Accurate prediction of RNA 5-hydroxymethylcytosine modification by utilizing
      novel position-specific gapped k-mer descriptors
  source_content_hash: sha256:f933088061b25216e116865b64934f54c7c4e6f119db0c474737fb5209cc2326
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zahid6454/iRNA5hmC-PS
  tools:
  - iRNA5hmC-PS
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# position-specific-gapped-kmer-descriptor-grouping

## Summary

Reconstructs iRNA5hmC-PS's group-wise dimensionality-reduction component: separate Random Forest selectors rank each position-specific descriptor mode (PSG k-mer and PS k-mer groups A-D), and the top-100 Gini-importance features per group plus GC-content yield a 401-feature reduced set.

## When to use

Use when a combined feature table mixes several descriptor families (e.g., position-specific gapped k-mer and position-specific k-mer modes plus GC-content) and you need per-group Random Forest feature selection rather than ranking all features jointly.

## When NOT to use

- Input is already a small, finalized feature vector — group-wise selection of top-100 per mode would be redundant
- Features come from a single homogeneous descriptor family with no natural grouping, so joint (not group-wise) importance ranking is appropriate
- The downstream classifiers are frequency-based models combined with position-specific features — the article notes position-specific features do not work well in conjunction with existing frequency-based counterparts in single models

## Inputs

- All_Features_Dataset.csv — combined feature table with four position-specific descriptor modes (A-D), a GC-content column, and binary class labels
- 5-fold cross-validation split definitions (training partitions per fold)

## Outputs

- selected_feature_indices.csv — 401 feature entries, each recording descriptor mode plus original column index (4 x 100 RF-selected features plus GC-content)
- Per-group ranked Gini importance lists from the four RandomForestClassifier selectors

## How to apply

Load All_Features_Dataset.csv with pandas and split its columns into the four position-specific descriptor modes (A-D), the GC-content column, and class labels. For each descriptor mode, fit an independent scikit-learn RandomForestClassifier on the training partition of each 5-fold cross-validation split and rank that mode's features by Gini (mean-decrease-in-impurity) importance — fitting on training partitions only prevents held-out fold leakage. Retain the top-100 highest-Gini-importance features from each of the four mode-wise selectors (400 features) and append the GC-content feature to form the 401-feature reduced set. Write the reduced feature-index set (descriptor mode plus original column index for each retained feature) to selected_feature_indices.csv with pandas. The rationale, per the article, is that a group-wise feature importance calculation strategy selects a small subset of features containing maximum discriminative information from the large combined feature set.

## Related tools

- **scikit-learn** (Provides the RandomForestClassifier whose Gini (mean-decrease-in-impurity) importance is used to rank features within each descriptor-mode group) — https://github.com/zahid6454/iRNA5hmC-PS
- **pandas** (Loads All_Features_Dataset.csv, separates the four descriptor modes / GC-content / labels, and writes selected_feature_indices.csv) — https://github.com/zahid6454/iRNA5hmC-PS
- **numpy** (Array handling for per-group feature importance matrices and index bookkeeping during selection) — https://github.com/zahid6454/iRNA5hmC-PS
- **iRNA5hmC-PS** (The originating framework whose group-wise feature importance calculation strategy this skill reconstructs) — https://github.com/zahid6454/iRNA5hmC-PS

## Examples

```
import pandas as pd; from sklearn.ensemble import RandomForestClassifier; df = pd.read_csv('All_Features_Dataset.csv'); rf = RandomForestClassifier().fit(X_train, y_train); top100 = pd.Series(rf.feature_importances_, index=mode_cols).nlargest(100)
```

## Evaluation signals

- selected_feature_indices.csv contains exactly 401 feature entries (4 x 100 plus GC-content)
- Each descriptor mode contributes exactly 100 retained features; no group exceeds or falls short of the top-100 cutoff
- Selectors were fit only on training partitions of each 5-fold split — no held-out fold data influenced Gini rankings
- Retained features trace back to valid original column indices in All_Features_Dataset.csv
- Downstream 5-fold CV with the reduced 401-feature set should reach roughly the reported LR performance of 78.3% accuracy / 0.56 MCC on 50-times 5-fold cross-validation

## Limitations

- The article reports that the proposed position-specific features do not work well in conjunction with existing frequency-based counterparts, deferring a feature subspace ensemble combination to future work
- Selectors are refit per cross-validation split, so the selected 401-feature set is fold-dependent and not a single global ranking
- Efficacy of the framework on other RNA modification site identification problems has not been evaluated

## Evidence

- [abstract] We also use a group-wise feature importance calculation strategy to select a small subset of features containing maximum discriminative information: "We also use a group-wise feature importance calculation strategy to select a small subset of features containing maximum discriminative information"
- [methods] Feature Generation notebook produces the combined feature table: ""Feature Generation.ipynb" uses "Dataset.csv" file to generate all the proposed features and stores that into a new file titled as "All_Features_Dataset.csv""
- [discussion] Position-specific features combine poorly with frequency-based features in single models: "our proposed position-specific features do not work well in conjunction with existing frequency-based counterparts"
- [results] Reported cross-validation metrics for the final LR model on the reduced feature set: "LR achieves 78.3%, 78.0%, 80.0%, 79.5%, 0.78, 0.56 in terms of Accuracy, Precision, Sensitivity, Specificity, F1-Score, and MCC, respectively"
- [readme] README: execute Model.ipynb after the combined feature table exists: "When "All_Features_Dataset.csv" file is available execute the "Model.ipynb" file just like "Feature Generation.ipynb" file was executed earlier."
