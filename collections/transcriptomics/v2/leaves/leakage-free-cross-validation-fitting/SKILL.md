---
name: leakage-free-cross-validation-fitting
description: Use when a pipeline includes data-driven steps such as feature ranking,
  selection, or scaling that must be re-fit inside every CV fold to avoid information
  leaking from held-out data into the model.
when_to_use_negative:
- Input is already a reduced/cleaned feature table with no data-dependent selection
  step remaining — a single global fit is then acceptable and per-fold refitting adds
  no protection
- You need a single fixed feature panel to deploy (e.g., for the iRNA5hmC-PS webserver);
  per-fold selectors differ across folds and are intended for unbiased performance
  estimation, not a final feature list
- The model itself performs embedded feature selection (e.g., L1-regularized logistic
  regression) inside the CV loop, making a separate Random Forest selector redundant
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: scikit-learn
  role: Provides the RandomForestClassifier whose Gini (mean-decrease-in-impurity)
    importances rank each descriptor mode's features on training partitions only
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: pandas
  role: Loads All_Features_Dataset.csv, separates the four descriptor modes and GC-content
    column, and writes selected_feature_indices.csv
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: numpy
  role: Array handling for fold partitions and feature-importance sorting
- name: iRNA5hmC-PS
  role: The prediction framework whose group-wise Random Forest feature-selection
    module this leakage-free fitting procedure reproduces
  repo: https://github.com/zahid6454/iRNA5hmC-PS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2020.10.032
    title: Accurate prediction of RNA 5-hydroxymethylcytosine modification by utilizing
      novel position-specific gapped k-mer descriptors
  source_content_hash: sha256:1a28af6e3729dbd2f4a7ddb8d57c73ed5e5af283734d4d3c537007f75f997f3c
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
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Leakage-free cross-validation fitting

## Summary

Fits data-dependent preprocessing/feature-selection steps (here, group-wise RandomForestClassifier Gini-importance selection for iRNA5hmC-PS) strictly on the training partition of each cross-validation fold, so that held-out folds remain untouched and performance estimates are unbiased.

## When to use

Use when a pipeline includes data-driven steps such as feature ranking, selection, or scaling that must be re-fit inside every CV fold to avoid information leaking from held-out data into the model.

## When NOT to use

- Input is already a reduced/cleaned feature table with no data-dependent selection step remaining — a single global fit is then acceptable and per-fold refitting adds no protection
- You need a single fixed feature panel to deploy (e.g., for the iRNA5hmC-PS webserver); per-fold selectors differ across folds and are intended for unbiased performance estimation, not a final feature list
- The model itself performs embedded feature selection (e.g., L1-regularized logistic regression) inside the CV loop, making a separate Random Forest selector redundant

## Inputs

- All_Features_Dataset.csv (combined feature table with four position-specific descriptor mode blocks, GC-content column, and binary class labels)
- Cross-validation fold assignment (5-fold splits, repeated 50 times in the paper's evaluation)

## Outputs

- selected_feature_indices.csv (401 entries: descriptor mode plus original column index for each retained feature)
- Fold-specific reduced feature matrices (training and test partitions) for downstream classifier training/evaluation

## How to apply

Load All_Features_Dataset.csv with pandas and split its columns into the four position-specific descriptor modes (A-D), the GC-content column, and the class labels. For each 5-fold cross-validation split, fit an independent scikit-learn RandomForestClassifier on only the training partition of that fold and rank each descriptor mode's features by Gini (mean-decrease-in-impurity) importance. Retain the top-100 highest-importance features per mode (4 x 100 = 400 features) and append the GC-content feature to form the 401-feature reduced set; the selector must never see held-out fold rows. Write the retained feature identities (descriptor mode plus original column index) to selected_feature_indices.csv with pandas, then train and evaluate the downstream classifier (LR/SVM/GNB) on the reduced set. The rationale is that selecting features using the full dataset would inflate cross-validation accuracy, so selection is folded into the CV loop itself.

## Related tools

- **scikit-learn** (Provides the RandomForestClassifier whose Gini (mean-decrease-in-impurity) importances rank each descriptor mode's features on training partitions only) — https://github.com/zahid6454/iRNA5hmC-PS
- **pandas** (Loads All_Features_Dataset.csv, separates the four descriptor modes and GC-content column, and writes selected_feature_indices.csv) — https://github.com/zahid6454/iRNA5hmC-PS
- **numpy** (Array handling for fold partitions and feature-importance sorting)
- **iRNA5hmC-PS** (The prediction framework whose group-wise Random Forest feature-selection module this leakage-free fitting procedure reproduces) — https://github.com/zahid6454/iRNA5hmC-PS

## Examples

```
from sklearn.ensemble import RandomForestClassifier; rf = RandomForestClassifier().fit(X_train_mode, y_train); top100 = np.argsort(rf.feature_importances_)[::-1][:100]
```

## Evaluation signals

- selected_feature_indices.csv contains exactly 401 feature entries (4 modes x 100 plus GC-content)
- No held-out fold data was used during selector fitting — feature selection is re-executed for every fold within the CV loop
- Each mode contributes exactly 100 top-Gini-importance features, and the GC-content feature is appended as the 401st entry
- Reported cross-validation metrics (e.g., LR: 78.3% accuracy, 0.56 MCC on 50-times 5-fold CV) are consistent with leakage-free estimates and do not show implausibly inflated values
- Retracing the workflow reproduces retained feature indices matching descriptor mode plus original column index per feature

## Limitations

- Position-specific features did not combine well with frequency-based k-mer features in single models; the authors defer a feature subspace ensemble approach to future work
- Per-fold selectors produce fold-specific feature sets, so the method yields an unbiased performance estimate rather than one deployable feature panel
- The 50-times 5-fold cross-validation protocol is computationally heavier than a single selection pass, since four RandomForest selectors must be re-fit for every fold and repeat

## Evidence

- [abstract] Group-wise feature importance calculation for feature selection: "We also use a group-wise feature importance calculation strategy to select a small subset of features containing maximum discriminative information"
- [methods] Stores combined features into All_Features_Dataset.csv: ""Feature Generation.ipynb" uses "Dataset.csv" file to generate all the proposed features and stores that into a new file titled as "All_Features_Dataset.csv""
- [results] Evaluation protocol uses repeated 5-fold cross-validation: "Results obtained on 5-fold cross-validation and the independent set for the three classifiers are reported in Table 5 and Table 6, respectively"
