---
name: svm-kernel-hyperparameter-tuning
description: Use when training an RBF-kernel SVM on fused sequence-encoded features
  (e.g., PSP, k-mer, PCPseDNC, PseEIIP) and default hyperparameters underperform,
  so C and gamma must be optimized against 10-fold CV accuracy.
when_to_use_negative:
- The input is already a feature table with a different base learner chosen (e.g.,
  XGBoost or LightGBM tuning) — this skill covers only SVM RBF kernel parameters C
  and gamma
- A general cross-species m5C predictor is needed — the paper notes it is 'difficult
  to develop a general model for cross-species prediction' and species-specific models
  are required
- Independent-test-set performance is the target — Staem5 was inferior to iRNA-m5C_SVM
  and m5CPred-SVM on the independent A. thaliana test set, so CV tuning gains do not
  transfer there
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: Staem5
  role: Source framework whose SVM base classifier (RBF kernel) is tuned via Bayesian
    optimization within the stacking pipeline
  repo: https://github.com/Cxd-626/Staem5.git
- name: scikit-learn
  role: Implements the SVM with radial basis kernel (SVC) and the 10-fold cross-validation
    used as the optimization objective
- name: Bayesian optimization
  role: Hyperparameter search strategy that selects C and gamma by maximizing 10-fold
    cross-validation accuracy
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:8c60024a21fe622a4fe83198843542b3bf087d321cbaa6edbae7336996d59387
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - Bayesian optimization
  - Staem5
  - iLearn
  - mlxtend
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# svm-kernel-hyperparameter-tuning

## Summary

Tune the regularization parameter C and RBF kernel parameter gamma of an SVM classifier via Bayesian optimization, maximizing 10-fold cross-validation accuracy, as done in Staem5 for m5C site prediction on the A. thaliana dataset.

## When to use

Use when training an RBF-kernel SVM on fused sequence-encoded features (e.g., PSP, k-mer, PCPseDNC, PseEIIP) and default hyperparameters underperform, so C and gamma must be optimized against 10-fold CV accuracy.

## When NOT to use

- The input is already a feature table with a different base learner chosen (e.g., XGBoost or LightGBM tuning) — this skill covers only SVM RBF kernel parameters C and gamma
- A general cross-species m5C predictor is needed — the paper notes it is 'difficult to develop a general model for cross-species prediction' and species-specific models are required
- Independent-test-set performance is the target — Staem5 was inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent A. thaliana test set, so CV tuning gains do not transfer there

## Inputs

- Balanced m5C training dataset (FASTA sequences of 41-bp segments around candidate cytosines; 5,298 positive and 5,298 negative for A. thaliana)
- Feature matrix from fused PSP, k-mer (k=1,2,3), PCPseDNC, and PseEIIP encodings with F-score-based dimension reduction (step of 5)
- Pre-set SVM hyperparameter values from supplementary Table S2

## Outputs

- Optimized SVM hyperparameters (C and gamma), reproducible via supplementary Table S2-style reporting
- Before/after Bayesian-optimization 10-fold CV accuracy table (e.g., base_classifier_accuracies.csv)
- Optimized SVM prediction scores consumed as base-learner input to the logistic-regression meta-classifier stack

## How to apply

Train the SVM (radial basis kernel) with pre-set hyperparameters (per the paper's supplementary Table S2) in scikit-learn and record 10-fold cross-validation accuracy as the baseline (SVM: 62.40% on the A. thaliana dataset of 5,298 positive and 5,298 negative 41-bp segments). Apply Bayesian optimization over the SVM regularization parameter C and kernel parameter gamma, selecting the parameter pair that maximizes 10-fold cross-validation accuracy. Re-run 10-fold cross-validation with the optimized SVM and tabulate before/after accuracies alongside the other base classifiers (GBDT, XGBoost, LightGBM, ExtraTree). Correct application is confirmed when the optimized SVM shows the largest post-optimization accuracy gain among the five classifiers and reaches approximately 73.62% CV accuracy, which then supports its selection (with XGBoost and LightGBM) as a base classifier of the stacked model.

## Related tools

- **Staem5** (Source framework whose SVM base classifier (RBF kernel) is tuned via Bayesian optimization within the stacking pipeline) — https://github.com/Cxd-626/Staem5.git
- **scikit-learn** (Implements the SVM with radial basis kernel (SVC) and the 10-fold cross-validation used as the optimization objective)
- **Bayesian optimization** (Hyperparameter search strategy that selects C and gamma by maximizing 10-fold cross-validation accuracy)

## Examples

```
from sklearn.svm import SVC; clf = SVC(kernel='rbf', C=C_opt, gamma=gamma_opt)  # C, gamma chosen by Bayesian optimization maximizing 10-fold CV accuracy
```

## Evaluation signals

- Reproduced 10-fold CV accuracy of the optimized SVM is approximately 73.62%, up from approximately 62.40% before optimization (A. thaliana dataset)
- SVM exhibits the largest post-optimization accuracy gain among the five base classifiers (SVM, XGBoost, LightGBM, ExtraTree, GBDT), exceeding GBDT's improvement (65.69% to 71.77%)
- Optimized SVM achieves the best 10-fold CV performance among the five base classifiers for A. thaliana
- When SVM, XGBoost, and LightGBM are stacked with logistic regression, the ensemble reaches approximately 73.89% accuracy, 0.479 MCC, and approximately 0.810 AUC

## Limitations

- Pre-optimization hyperparameters were set 'according to previous experience' rather than systematically derived, and the selected parameters are only in supplementary Table S2
- Feature-selection and stacking comparison details are only in supplementary Tables S1, S3, S5–S7, complicating exact reproduction
- Performance gains are species-specific; no general cross-species model is supported
- Staem5 underperformed iRNA-m5C_SVM and m5CPred-SVM on the independent A. thaliana test set

## Evidence
