---
name: confusion-matrix-metric-calculation
description: Use when you have TP/TN/FP/FN counts (or raw positive/negative predictions
  plus labels) from m5C site predictors on the Chen et al. A. thaliana or Yang et
  al. M. musculus training/independent test sets and need Table 3/Table 4-style benchmark
  metrics.
when_to_use_negative:
- You only have raw RNA sequences and no trained model or predictions yet — you must
  first generate predictions (e.g., via m5C_predict.py) before metrics can be computed
- You are evaluating on datasets other than the same training and independent test
  datasets of A. thaliana and M. musculus used in this benchmark — cross-dataset comparability
  of the published Table 3/Table 4 values would be lost
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Staem5
  role: Stacked ensemble predictor (SVM, XGBoost, LightGBM bases + logistic regression
    meta-classifier) whose cross-validation and independent-test confusion matrices
    are scored
  repo: https://github.com/Cxd-626/Staem5.git
- name: scikit-learn
  role: Python library (sklearn 0.21.3) in which the stacked model and its meta-classifier
    are built; source of confusion-matrix and metric utilities
- name: m5CPred-SVM
  role: SVM-based baseline predictor evaluated on identical A. thaliana and M. musculus
    datasets
- name: iRNA-m5C_SVM
  role: SVM-based baseline predictor evaluated on the identical A. thaliana training
    and independent test datasets
- name: iRNA-m5C
  role: RF-based baseline predictor evaluated on the identical A. thaliana datasets;
    its independent-test AUC is unavailable
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:5da7a8a01cdf79cd40562ab99f5288a6ba3d440b002b75efdf11ac26347fbddb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - Staem5
  - iRNA-m5C
  - iRNA-m5C_SVM
  - m5CPred-SVM
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# confusion-matrix-metric-calculation

## Summary

Computes the standard binary-classifier metrics (Sn, Sp, Acc, MCC, AUC) from confusion-matrix counts (TP, TN, FP, FN) so that Staem5's performance can be compared side by side with state-of-the-art m5C predictors (iRNA-m5C, m5CPred-SVM, iRNA-m5C_SVM) on identical A. thaliana and M. musculus benchmark datasets.

## When to use

Use when you have TP/TN/FP/FN counts (or raw positive/negative predictions plus labels) from m5C site predictors on the Chen et al. A. thaliana or Yang et al. M. musculus training/independent test sets and need Table 3/Table 4-style benchmark metrics.

## When NOT to use

- You only have raw RNA sequences and no trained model or predictions yet — you must first generate predictions (e.g., via m5C_predict.py) before metrics can be computed
- You are evaluating on datasets other than the same training and independent test datasets of A. thaliana and M. musculus used in this benchmark — cross-dataset comparability of the published Table 3/Table 4 values would be lost

## Inputs

- Per-predictor classification outcomes on each dataset: TP, TN, FP, FN counts
- Staem5 predictions from the stacked model (SVM/XGBoost/LightGBM base classifiers, logistic regression meta-classifier) via m5C_predict.py
- Baseline predictor outputs: iRNA-m5C, m5CPred-SVM, iRNA-m5C_SVM on the same A. thaliana datasets; m5CPred-SVM on M. musculus datasets
- Dataset labels for Chen et al. A. thaliana (training 5,298 pos/5,298 neg; test 1,000/1,000) and Yang et al. M. musculus (training 4,563 pos/4,563 neg; test 1,000/1,000)

## Outputs

- benchmark_comparison.csv with side-by-side Sn, Sp, Acc, MCC, AUC values per predictor per dataset
- Tables mirroring the paper's Table 3 (A. thaliana cross-validation and independent test) and Table 4 (M. musculus)

## How to apply

Run each predictor (Staem5, iRNA-m5C, m5CPred-SVM, iRNA-m5C_SVM) on the identical A. thaliana training (5,298/5,298) and independent test (1,000/1,000) datasets, and on the M. musculus training (4,563/4,563) and 1,000/1,000 test samples. For 10-fold cross-validation on training sets, aggregate predictions across folds; for the independent sets, use single-pass predictions. From the confusion-matrix counts compute Sn = TP/(TP+FN), Sp = TN/(TN+FP), Acc = (TP+TN)/(TP+TN+FP+FN), MCC = (TP*TN - FP*FN)/sqrt((TP+FP)*(TP+FN)*(TN+FP)*(TN+FN)), and AUC from prediction scores where available. Assemble side-by-side tables mirroring Table 3 (A. thaliana, cross-validation and independent test) and Table 4 (M. musculus) and write them to benchmark_comparison.csv. If a baseline does not report a score-based AUC (e.g., iRNA-m5C on the A. thaliana independent test set), leave that cell empty rather than imputing it.

## Related tools

- **Staem5** (Stacked ensemble predictor (SVM, XGBoost, LightGBM bases + logistic regression meta-classifier) whose cross-validation and independent-test confusion matrices are scored) — https://github.com/Cxd-626/Staem5.git
- **scikit-learn** (Python library (sklearn 0.21.3) in which the stacked model and its meta-classifier are built; source of confusion-matrix and metric utilities)
- **m5CPred-SVM** (SVM-based baseline predictor evaluated on identical A. thaliana and M. musculus datasets)
- **iRNA-m5C_SVM** (SVM-based baseline predictor evaluated on the identical A. thaliana training and independent test datasets)
- **iRNA-m5C** (RF-based baseline predictor evaluated on the identical A. thaliana datasets; its independent-test AUC is unavailable)

## Examples

```
acc = (TP+TN)/(TP+TN+FP+FN); MCC = (TP*TN - FP*FN)/((TP+FP)*(TP+FN)*(TN+FP)*(TN+FN))**0.5
```

## Evaluation signals

- Computed metrics for Staem5 on A. thaliana 10-fold cross-validation reproduce the published values 73.89% Acc and 0.479 MCC
- Recomputed Sn/Sp/Acc/MCC/AUC values match the published Table 3 (A. thaliana) and Table 4 (M. musculus) cell by cell
- The AUC cell for iRNA-m5C on the A. thaliana independent test set is left empty (dash), matching the paper's 'iRNA-m5C 72.40 75.60 74.10 0.481 –' row
- All four metric denominators check out: Sn and Sp computed from positives and negatives respectively, Acc over 2,598 (A. thaliana CV per-predictor totals) or 2,000 independent test samples
- Cross-validation and independent-test results are reported separately — A. thaliana independent-test rows should show Staem5 inferior to iRNA-m5C_SVM and m5CPred-SVM despite its CV superiority

## Limitations

- AUC is not available for iRNA-m5C on the independent test dataset (marked with a dash), so AUC-based comparison is incomplete for that predictor
- Metrics derived this way exposed that Staem5 was inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent test set of A. thaliana — cross-validation superiority does not guarantee independent-test superiority
- Species-specific models are required; Staem5 offers only 'Arabidopsis' and 'Mouse' options, so metrics on other species cannot be generated with this benchmark

## Evidence
