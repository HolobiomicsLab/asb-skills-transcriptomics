---
name: binary-classification-metric-computation
description: Use when comparing binary RNA-modification classifiers (e.g., m5C predictors
  on A. thaliana or M. musculus datasets) and you need side-by-side Sn, Sp, Acc, MCC,
  AUC tables from TP/TN/FP/FN counts.
when_to_use_negative:
- The task is multi-class classification or regression — Sn/Sp/MCC formulas here assume
  a two-class m5C vs non-m5C prediction setting
- Input is already a metric table (e.g., published Table 3/Table 4 values); recompute
  only when raw TP/TN/FP/FN counts or predictions are available
- Datasets are highly imbalanced or species other than those with species-specific
  Staem5 models ('Arabidopsis' or 'Mouse'), since the article notes no general cross-species
  model exists
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3307
tools:
- name: Staem5
  role: Stacked ensemble (SVM, XGBoost, LightGBM base classifiers, logistic regression
    meta-classifier) whose cross-validation and independent-test predictions supply
    the TP/TN/FP/FN counts to be converted into Sn, Sp, Acc, MCC, AUC
  repo: https://github.com/Cxd-626/Staem5.git
- name: scikit-learn
  role: Used (with mlxtend) to build the base classifiers, stacking model, and to
    compute the classification metrics under 10-fold cross-validation
- name: m5CPred-SVM
  role: SVM-based baseline predictor evaluated on the identical A. thaliana and M.
    musculus training and test datasets for metric comparison
- name: iRNA-m5C
  role: RF-based baseline predictor run on the A. thaliana datasets; its AUC on the
    independent test set is reported as missing (dash)
- name: iRNA-m5C_SVM
  role: SVM-based baseline predictor evaluated on the identical A. thaliana training
    and independent test datasets
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:7896d5a960215d52b6dfa6c8890536f0a9fd0bc7ad9d5f46c8bbbdc4ed1d5db0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - Staem5
  - iRNA-m5C
  - iRNA-m5C_SVM
  - m5CPred-SVM
  - mlxtend
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# binary-classification-metric-computation

## Summary

Compute Sn, Sp, accuracy, MCC, and AUC from confusion-matrix counts (TP, TN, FP, FN) to benchmark binary m5C site predictors such as Staem5 against iRNA-m5C, m5CPred-SVM, and iRNA-m5C_SVM on identical training and independent test datasets.

## When to use

Use when comparing binary RNA-modification classifiers (e.g., m5C predictors on A. thaliana or M. musculus datasets) and you need side-by-side Sn, Sp, Acc, MCC, AUC tables from TP/TN/FP/FN counts.

## When NOT to use

- The task is multi-class classification or regression — Sn/Sp/MCC formulas here assume a two-class m5C vs non-m5C prediction setting
- Input is already a metric table (e.g., published Table 3/Table 4 values); recompute only when raw TP/TN/FP/FN counts or predictions are available
- Datasets are highly imbalanced or species other than those with species-specific Staem5 models ('Arabidopsis' or 'Mouse'), since the article notes no general cross-species model exists

## Inputs

- Confusion-matrix counts (TP, TN, FP, FN) or raw binary predictions for each predictor on each dataset
- Balanced 41-bp m5C benchmark datasets: A. thaliana training (5,298 positive / 5,298 negative) and independent test (1,000 / 1,000); M. musculus training (4,563 / 4,563) with 1,000 / 1,000 test samples
- Prediction outputs from Staem5 (stacked SVM, XGBoost, LightGBM with logistic regression meta-classifier) and baseline predictors iRNA-m5C, m5CPred-SVM, iRNA-m5C_SVM

## Outputs

- Side-by-side metric tables of Sn, Sp, Acc, MCC, and AUC per predictor per dataset (e.g., benchmark_comparison.csv mirroring Tables 3 and 4)
- Per-cell AUC values with empty entries where the source predictor does not report AUC

## How to apply

For each predictor and dataset, derive the confusion-matrix counts (TP, TN, FP, FN) from 10-fold cross-validation on the training sets and from predictions on the independent test sets (e.g., 1,000 positive / 1,000 negative A. thaliana test samples). Compute sensitivity Sn = TP/(TP+FN), specificity Sp = TN/(TN+FP), accuracy Acc = (TP+TN)/(TP+TN+FP+FN), MCC = (TP*TN - FP*FN)/sqrt((TP+FP)(TP+FN)(TN+FP)(TN+FN)), and AUC from the ROC curve. Assemble the metrics into side-by-side comparison tables mirroring Table 3 (A. thaliana, cross-validation and independent test) and Table 4 (M. musculus) and write them to a CSV such as benchmark_comparison.csv. Where a predictor does not report a metric (e.g., AUC for iRNA-m5C on the A. thaliana independent test set), leave the cell empty rather than imputing a value.

## Related tools

- **Staem5** (Stacked ensemble (SVM, XGBoost, LightGBM base classifiers, logistic regression meta-classifier) whose cross-validation and independent-test predictions supply the TP/TN/FP/FN counts to be converted into Sn, Sp, Acc, MCC, AUC) — https://github.com/Cxd-626/Staem5.git
- **scikit-learn** (Used (with mlxtend) to build the base classifiers, stacking model, and to compute the classification metrics under 10-fold cross-validation)
- **m5CPred-SVM** (SVM-based baseline predictor evaluated on the identical A. thaliana and M. musculus training and test datasets for metric comparison)
- **iRNA-m5C** (RF-based baseline predictor run on the A. thaliana datasets; its AUC on the independent test set is reported as missing (dash))
- **iRNA-m5C_SVM** (SVM-based baseline predictor evaluated on the identical A. thaliana training and independent test datasets)

## Examples

```
from sklearn.metrics import confusion_matrix, matthews_corrcoef, roc_auc_score, accuracy_score; tn, fp, fn, tp = confusion_matrix(y_true, y_pred).ravel(); acc = accuracy_score(y_true, y_pred); mcc = matthews_corrcoef(y_true, y_pred)
```

## Evaluation signals

- Computed metrics reproduce the published Table 3 values, including Staem5's 73.89% accuracy and 0.479 MCC on A. thaliana 10-fold cross-validation
- Stacking-versus-voting comparison reproduces Table 2: A. thaliana stacking 73.89% vs voting 73.68% accuracy; M. musculus stacking 77.12% vs voting 76.83%
- The AUC cell for iRNA-m5C on the A. thaliana independent test set is left empty (dash), matching the paper's reporting
- Metric consistency check: Sn, Sp, and Acc computed from the same TP/TN/FP/FN counts satisfy Acc = (TP+TN)/N and the denominators sum to the dataset size (e.g., 2,000 test samples per species)
- Per-dataset MCC is undefined-safe: any denominator of zero in the MCC formula is handled rather than silently producing NaN

## Limitations

- Staem5 was inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent test set of A. thaliana, so cross-validation superiority does not guarantee independent-test superiority
- AUC was not reported for iRNA-m5C on the independent test dataset (marked with a dash), so comparisons involving that cell are incomplete
- Species-specific models are required; the article notes it could be difficult to develop a general model for cross-species prediction
- Detailed performance metrics are only available in supplementary Table S1 and detailed stacking comparison results in Table S3, not the main text

## Evidence

- [full_text] Compute Sn, Sp, Acc, MCC, and AUC from the confusion-matrix counts (TP, TN, FP, FN) for each predictor on each dataset.: [evidence span withheld — anchor status: not_found]
- [discussion] Staem5 inferior on independent test: "Staem5 was inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent test set of A. thaliana"
- [discussion] Missing AUC cell: "iRNA-m5C	72.40	75.60	74.10	0.481	–"
- [discussion] Stacked model accuracy and MCC: "we accordingly selected SVM, XGBoost, and LightGBM as the base classifiers for the stacked model, which achieved 73.89% accuracy and 0.479 MCC"
- [discussion] Cross-validation evaluation setup: "performance comparison results in terms of accuracy of the five base classifiers before and after parameter optimization on the 10-fold cross-validation tests"
