---
name: cross-validation-and-independent-test-evaluation
description: Use when a new sequence-based predictor (e.g., a stacked ensemble for
  m5C sites in A. thaliana or M. musculus) must be benchmarked against tools on identical
  training and independent test datasets.
when_to_use_negative:
- The goal is feature encoding or model training/parameter tuning (e.g., F-score feature
  selection or Bayesian optimization) rather than benchmark comparison of a trained
  predictor
- The datasets differ from the published benchmark sets (different species, sizes,
  or construction), so metrics would not be comparable to Table 3/Table 4 values
- Only cross-validation is needed and no independent test or comparison against iRNA-m5C,
  m5CPred-SVM, or iRNA-m5C_SVM is required
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: Staem5
  role: 'Stacked ensemble predictor being evaluated: SVM, XGBoost, and LightGBM base
    classifiers with a logistic regression meta-classifier, run via 10-fold cross-validation
    and independent test prediction'
  repo: https://github.com/Cxd-626/Staem5.git
- name: iRNA-m5C
  role: Random forest-based baseline predictor benchmarked on the A. thaliana training
    and independent test datasets
- name: m5CPred-SVM
  role: SVM-based baseline predictor benchmarked on the A. thaliana and M. musculus
    training and independent test datasets
- name: iRNA-m5C_SVM
  role: SVM-based baseline predictor benchmarked on the A. thaliana training and independent
    test datasets
- name: scikit-learn
  role: Framework used to build the stacking ensemble with logistic regression as
    the meta-classifier and to run 10-fold cross-validation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:2467381667d55bb7cea49d0adcf0f5c2ab0114b4cdce382431a9d4c2d87f9c9f
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-validation-and-independent-test-evaluation

## Summary

Evaluates an m5C site predictor such as Staem5 by 10-fold cross-validation on training sets and prediction on species-specific independent test sets, computing Sn, Sp, Acc, MCC, and AUC from confusion-matrix counts (TP, TN, FP, FN) to enable side-by-side comparison with existing predictors.

## When to use

Use when a new sequence-based predictor (e.g., a stacked ensemble for m5C sites in A. thaliana or M. musculus) must be benchmarked against state-of-the-art tools on identical training and independent test datasets.

## When NOT to use

- The goal is feature encoding or model training/parameter tuning (e.g., F-score feature selection or Bayesian optimization) rather than benchmark comparison of a trained predictor
- The datasets differ from the published benchmark sets (different species, sizes, or construction), so metrics would not be comparable to Table 3/Table 4 values
- Only cross-validation is needed and no independent test or comparison against iRNA-m5C, m5CPred-SVM, or iRNA-m5C_SVM is required

## Inputs

- A. thaliana training dataset (5,298 positive / 5,298 negative samples, Chen et al.)
- A. thaliana independent test dataset (1,000 positive / 1,000 negative samples)
- M. musculus training dataset (4,563 positive / 4,563 negative samples, Yang et al.)
- M. musculus independent test dataset (1,000 positive / 1,000 negative samples)
- Trained predictor models (e.g., Staem5 stacked ensemble; iRNA-m5C, m5CPred-SVM, iRNA-m5C_SVM baselines)

## Outputs

- Confusion-matrix counts (TP, TN, FP, FN) per predictor per dataset
- Performance metrics table: Sn, Sp, Acc, MCC, AUC per predictor on cross-validation and independent test sets
- benchmark_comparison.csv with side-by-side tables mirroring published Table 3 (A. thaliana) and Table 4 (M. musculus)

## How to apply

Load the benchmark datasets: the A. thaliana training set (5,298 positive and 5,298 negative samples) and independent test set (1,000/1,000) from Chen et al., and the M. musculus training set (4,563/4,563) with 1,000/1,000 test samples from Yang et al. Run the trained predictor — for Staem5, the stacked model with SVM, XGBoost, and LightGBM base classifiers and a logistic regression meta-classifier built in scikit-learn under Python 3.7 — using 10-fold cross-validation on the training sets and direct predictions on the independent test sets. Run baseline predictors (iRNA-m5C, m5CPred-SVM, iRNA-m5C_SVM for A. thaliana; m5CPred-SVM for M. musculus) on the identical datasets. Compute Sn, Sp, Acc, MCC, and AUC from the confusion-matrix counts for each predictor on each dataset, and assemble side-by-side comparison tables (mirroring Table 3 for A. thaliana and Table 4 for M. musculus). Validate by checking that computed metrics reproduce the published values, e.g., Staem5's 73.89% accuracy and 0.479 MCC on A. thaliana 10-fold cross-validation, and that missing AUC cells (e.g., iRNA-m5C on the A. thaliana independent test set) are left empty as dashes.

## Related tools

- **Staem5** (Stacked ensemble predictor being evaluated: SVM, XGBoost, and LightGBM base classifiers with a logistic regression meta-classifier, run via 10-fold cross-validation and independent test prediction) — https://github.com/Cxd-626/Staem5.git
- **iRNA-m5C** (Random forest-based baseline predictor benchmarked on the A. thaliana training and independent test datasets)
- **m5CPred-SVM** (SVM-based baseline predictor benchmarked on the A. thaliana and M. musculus training and independent test datasets)
- **iRNA-m5C_SVM** (SVM-based baseline predictor benchmarked on the A. thaliana training and independent test datasets)
- **scikit-learn** (Framework used to build the stacking ensemble with logistic regression as the meta-classifier and to run 10-fold cross-validation)

## Examples

```
python m5C_predict.py --input test.fa --output predictions.txt --species Arabidopsis
```

## Evaluation signals

- Computed Staem5 A. thaliana 10-fold cross-validation accuracy equals 73.89% and MCC equals 0.479, matching published values
- Metrics (Sn, Sp, Acc, MCC, AUC) derived from TP/TN/FP/FN counts reproduce the published Table 3 (A. thaliana) and Table 4 (M. musculus) values
- The AUC cell for iRNA-m5C on the A. thaliana independent test set is left empty (dash), matching the paper (72.40 / 75.60 / 74.10 / 0.481 / –)
- benchmark_comparison.csv contains side-by-side rows for all evaluated predictors on both training and independent test datasets
- Cross-validation and independent test predictions use the identical datasets as the baselines, ensuring a fair comparison

## Limitations

- Staem5 was inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent test set of A. thaliana, so cross-validation superiority does not guarantee independent-test superiority
- No general cross-species model exists; species-specific models are required for A. thaliana and M. musculus
- AUC is not reported for some baseline comparisons (iRNA-m5C on the A. thaliana independent test set is marked with a dash), so not all metric cells can be compared

## Evidence
