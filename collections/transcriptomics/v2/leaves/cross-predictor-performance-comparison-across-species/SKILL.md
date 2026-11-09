---
name: cross-predictor-performance-comparison-across-species
description: Use when comparing Staem5's m5C prediction performance against SVM/RF-based
  baselines on the same A. thaliana and M. musculus training and independent test
  datasets to reproduce Tables 3 and 4.
when_to_use_negative:
- Input is already a metric/feature table rather than raw m5C sequence datasets that
  need prediction runs
- The goal is developing or tuning a new m5C predictor (base-classifier selection,
  Bayesian optimization, F score feature selection) rather than comparing existing
  trained predictors
- Prediction is needed for species other than A. thaliana or M. musculus, since Staem5
  only accepts 'Arabidopsis' or 'Mouse' and species-specific models are required
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: Staem5
  role: Stacked ensemble model (SVM, XGBoost, LightGBM base classifiers with logistic
    regression meta-classifier) run via m5C_predict.py on both species' training and
    independent test sets
  repo: https://github.com/Cxd-626/Staem5.git
- name: iRNA-m5C
  role: Random forest-based baseline predictor evaluated on the identical A. thaliana
    training and independent test datasets
- name: m5CPred-SVM
  role: SVM-based baseline predictor evaluated on the A. thaliana datasets and the
    M. musculus datasets
- name: iRNA-m5C_SVM
  role: SVM-based baseline predictor evaluated on the identical A. thaliana training
    and independent test datasets
- name: scikit-learn
  role: Framework in which the Staem5 stacking ensemble with logistic regression meta-classifier
    was built
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:7c753950ee88a38d5db6a8eac2fae52d3bddfcc88649432992b617f4d7fd5339
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cross-predictor-performance-comparison-across-species

## Summary

Benchmarking skill for evaluating the Staem5 stacked m5C predictor against state-of-the-art predictors (iRNA-m5C, m5CPred-SVM, iRNA-m5C_SVM) on identical A. thaliana and M. musculus training and independent test datasets, computing Sn, Sp, Acc, MCC, and AUC from confusion-matrix counts.

## When to use

Use when comparing Staem5's m5C prediction performance against SVM/RF-based baselines on the same A. thaliana and M. musculus training and independent test datasets to reproduce Tables 3 and 4.

## When NOT to use

- Input is already a metric/feature table rather than raw m5C sequence datasets that need prediction runs
- The goal is developing or tuning a new m5C predictor (base-classifier selection, Bayesian optimization, F score feature selection) rather than comparing existing trained predictors
- Prediction is needed for species other than A. thaliana or M. musculus, since Staem5 only accepts 'Arabidopsis' or 'Mouse' and species-specific models are required

## Inputs

- A. thaliana training dataset (5,298 positive / 5,298 negative m5C sequences) from Chen et al.
- A. thaliana independent test dataset (1,000 positive / 1,000 negative sequences)
- M. musculus training dataset (4,563 positive / 4,563 negative m5C sequences) from Yang et al.
- M. musculus independent test dataset (1,000 positive / 1,000 negative sequences)
- Query RNA sequences in FASTA format for Staem5 prediction runs

## Outputs

- benchmark_comparison.csv with side-by-side Sn, Sp, Acc, MCC, AUC per predictor per dataset
- Table 3 (A. thaliana cross-validation and independent test comparison)
- Table 4 (M. musculus comparison)
- Per-predictor confusion-matrix counts (TP, TN, FP, FN)

## How to apply

Load the A. thaliana training set (5,298 positive / 5,298 negative, from Chen et al.) and independent test set (1,000/1,000), plus the M. musculus training set (4,563/4,563) with 1,000/1,000 test samples (Yang et al.). Run Staem5 (Python 3.7; SVM, XGBoost, LightGBM base classifiers with a logistic regression meta-classifier in scikit-learn) under 10-fold cross-validation on the training sets and prediction on the independent test sets via m5C_predict.py with --species 'Arabidopsis' or 'Mouse'. Run iRNA-m5C, m5CPred-SVM, and iRNA-m5C_SVM on the identical A. thaliana datasets, and m5CPred-SVM on the M. musculus datasets. Compute Sn, Sp, Acc, MCC, and AUC from TP/TN/FP/FN confusion-matrix counts for each predictor on each dataset. Assemble side-by-side tables mirroring Table 3 (A. thaliana, cross-validation and independent test) and Table 4 (M. musculus) and write them to benchmark_comparison.csv, validating that Staem5 reproduces 73.89% accuracy and 0.479 MCC on A. thaliana 10-fold CV.

## Related tools

- **Staem5** (Stacked ensemble model (SVM, XGBoost, LightGBM base classifiers with logistic regression meta-classifier) run via m5C_predict.py on both species' training and independent test sets) — https://github.com/Cxd-626/Staem5.git
- **iRNA-m5C** (Random forest-based baseline predictor evaluated on the identical A. thaliana training and independent test datasets)
- **m5CPred-SVM** (SVM-based baseline predictor evaluated on the A. thaliana datasets and the M. musculus datasets)
- **iRNA-m5C_SVM** (SVM-based baseline predictor evaluated on the identical A. thaliana training and independent test datasets)
- **scikit-learn** (Framework in which the Staem5 stacking ensemble with logistic regression meta-classifier was built)

## Examples

```
python m5C_predict.py --input arabidopsis_test.fa --output preds.csv --species Arabidopsis
```

## Evaluation signals

- Computed metrics for Staem5 reproduce published values, including 73.89% accuracy and 0.479 MCC on the A. thaliana 10-fold cross-validation
- The AUC cell for iRNA-m5C on the A. thaliana independent test set is left empty (dash) as in the paper (iRNA-m5C 72.40, 75.60, 74.10, 0.481, dash)
- Each benchmark_comparison.csv row's Sn, Sp, Acc, MCC are correctly derived from the confusion-matrix counts (TP, TN, FP, FN)
- M. musculus comparison includes m5CPred-SVM only (the sole baseline run on Yang et al. datasets), mirroring Table 4
- Cross-validation versus independent-test results are separated per predictor, so A. thaliana test-set inferiority of Staem5 to iRNA-m5C_SVM and m5CPred-SVM is visible despite CV superiority

## Limitations

- Staem5 was inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent test set of A. thaliana, so CV superiority does not guarantee test-set superiority
- AUC is not reported for iRNA-m5C on the A. thaliana independent test dataset (marked with a dash), so full metric coverage is not available
- Selected hyperparameters are only in supplementary Table S2 and detailed metric/stacking results in Tables S1 and S3, not in the main text

## Evidence
