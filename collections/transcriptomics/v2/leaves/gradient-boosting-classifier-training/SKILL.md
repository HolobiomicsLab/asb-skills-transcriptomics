---
name: gradient-boosting-classifier-training
description: Use when you must train and benchmark GBDT (and sibling boosting classifiers
  like XGBoost/LightGBM) as base learners on sequence-encoded feature matrices and
  compare accuracy before vs. after Bayesian parameter tuning via 10-fold cross-validation.
when_to_use_negative:
- When the goal is the final stacked Staem5 prediction itself rather than benchmarking
  base classifiers — use the pre-trained stacked model via m5C_predict.py instead
- When a general cross-species m5C model is needed — the article notes species-specific
  models are required, so A. thaliana-tuned GBDT parameters do not transfer to M.
  musculus
- When input is already a feature table with final labels and the task is only ensemble
  stacking, not base-classifier training/tuning
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: Staem5
  role: Source of the training dataset, fused sequence encodings, and the five base
    classifiers (GBDT among them) evaluated in the stacking framework
  repo: https://github.com/Cxd-626/Staem5.git
- name: scikit-learn
  role: Implements the GBDT (GradientBoostingClassifier) base learner and 10-fold
    cross-validation evaluation
- name: XGBoost
  role: Gradient-boosting sibling classifier benchmarked alongside GBDT (xgboost==0.90
    in the Staem5 environment)
- name: LightGBM
  role: Light gradient boosting machine benchmarked alongside GBDT and selected with
    SVM for the final stacked model (lightgbm==2.3.0)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:6c70364fb1dd2876aad4448dcf0e502dd914959f7bc6ede715a8e0826cc8c32f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - LightGBM
  - Staem5
  - XGBoost
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gradient-boosting-classifier-training

## Summary

Trains gradient boosting decision tree (GBDT) base classifiers on encoded RNA sequence features and evaluates them with 10-fold cross-validation, optionally followed by Bayesian hyperparameter optimization, as done for the Staem5 m5C site prediction model on A. thaliana data.

## When to use

Use when you must train and benchmark GBDT (and sibling boosting classifiers like XGBoost/LightGBM) as base learners on sequence-encoded feature matrices and compare accuracy before vs. after Bayesian parameter tuning via 10-fold cross-validation.

## When NOT to use

- When the goal is the final stacked Staem5 prediction itself rather than benchmarking base classifiers — use the pre-trained stacked model via m5C_predict.py instead
- When a general cross-species m5C model is needed — the article notes species-specific models are required, so A. thaliana-tuned GBDT parameters do not transfer to M. musculus
- When input is already a feature table with final labels and the task is only ensemble stacking, not base-classifier training/tuning

## Inputs

- Positive and negative m5C site RNA sequences in FASTA format (A. thaliana training set: 5,298 positive / 5,298 negative, from the Staem5 repository)
- Feature matrix from fused PSP, k-mer (k=1,2,3), PCPseDNC, and PseEIIP encodings
- Pre-set GBDT hyperparameters from supplementary Table S2

## Outputs

- 10-fold cross-validation accuracy for the GBDT classifier before and after Bayesian optimization
- base_classifier_accuracies.csv tabulating before/after accuracies for SVM, XGBoost, LightGBM, ExtraTree, and GBDT
- Optimized GBDT model available as a base classifier for the stacking ensemble with logistic regression meta-classifier

## How to apply

Represent each m5C site candidate sequence (e.g., 5,298 positive and 5,298 negative A. thaliana sequences from the Staem5 repository) with the fused PSP, k-mer (k=1,2,3), PCPseDNC, and PseEIIP encoding schemes. Train a GBDT classifier in scikit-learn (sklearn==0.21.3, python 3.7 environment per the Staem5 README) using the pre-set hyperparameters listed in supplementary Table S2, and evaluate it with 10-fold cross-validation. Apply Bayesian optimization to tune GBDT hyperparameters, selecting the settings that maximize 10-fold cross-validation accuracy. Re-run 10-fold cross-validation for the optimized model and tabulate before/after accuracies for all five base classifiers (SVM, XGBoost, LightGBM, ExtraTree, GBDT). The expected pattern for A. thaliana is that all classifiers improve after optimization, with GBDT rising from 65.69% to 71.77% and SVM showing the largest gain (62.40% to 73.62%); GBDT-optimized performance feeds the subsequent stacking selection step.

## Related tools

- **Staem5** (Source of the training dataset, fused sequence encodings, and the five base classifiers (GBDT among them) evaluated in the stacking framework) — https://github.com/Cxd-626/Staem5.git
- **scikit-learn** (Implements the GBDT (GradientBoostingClassifier) base learner and 10-fold cross-validation evaluation)
- **XGBoost** (Gradient-boosting sibling classifier benchmarked alongside GBDT (xgboost==0.90 in the Staem5 environment))
- **LightGBM** (Light gradient boosting machine benchmarked alongside GBDT and selected with SVM for the final stacked model (lightgbm==2.3.0))

## Examples

```
from sklearn.ensemble import GradientBoostingClassifier; from sklearn.model_selection import cross_val_score; scores = cross_val_score(GradientBoostingClassifier(), X_fused, y, cv=10)
```

## Evaluation signals

- GBDT 10-fold cross-validation accuracy should improve from 65.69% to 71.77% after Bayesian optimization on the A. thaliana dataset
- The before/after comparison must show all five base classifiers improving after parameter optimization, with SVM showing the largest gain (62.40% to 73.62%)
- Optimized GBDT accuracy must be reproducible under 10-fold cross-validation on the 5,298:5,298 balanced A. thaliana training set
- The output CSV must contain accuracy rows for all five base classifiers (SVM, XGBoost, LightGBM, ExtraTree, GBDT) in a before/after schema
- Environment must match the Staem5 pinned versions (sklearn==0.21.3, xgboost==0.90, lightgbm==2.3.0, python 3.7) for parameter comparability

## Limitations

- Hyperparameters were pre-set according to previous experience rather than systematically derived, and the selected parameters are only available in supplementary Table S2
- GBDT alone underperforms SVM after optimization on A. thaliana 10-fold cross-validation tests; GBDT was not among the base classifiers (SVM, XGBoost, LightGBM) selected for the final stacked model
- Staem5 was inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent A. thaliana test set, so optimized base-classifier CV accuracy does not guarantee independent-test superiority

## Evidence
