---
name: bioinformatics-classifier-metric-evaluation
description: Use when comparing candidate classifiers (e.g., SVM, XGBoost, LightGBM,
  ExtraTree, GBDT), feature selection methods (F score, MRMD, PCC), or feature dimensions
  on a balanced sequence classification dataset and you need cross-validated Sn/Sp/Acc/MCC/AUC
  tables.
when_to_use_negative:
- The task is single-classifier deployment/inference on new sequences rather than
  comparative benchmarking (use the Staem5 m5C_predict.py prediction entry point instead).
- The dataset is heavily imbalanced or not amenable to 10-fold cross-validation, making
  Acc/MCC comparison across balanced folds misleading.
- You need a general cross-species m5C model — the article notes species-specific
  models are necessary and a general model is difficult to develop.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Staem5
  role: Stacked ensemble framework (SVM, XGBoost, LightGBM base classifiers with logistic
    regression meta-classifier) whose models are evaluated under this skill
  repo: https://github.com/Cxd-626/Staem5.git
- name: scikit-learn
  role: Implements the base classifiers and 10-fold cross-validation; README pins
    sklearn == 0.21.3
- name: mlxtend
  role: Provides the stacking package used to combine base classifiers via logistic
    regression meta-classifier; README pins mlxtend == 0.17.3
- name: XGBoost
  role: Gradient-boosted base classifier in the stacked model being benchmarked
- name: LightGBM
  role: Light gradient boosting machine base classifier in the stacked model being
    benchmarked
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:3ba2577a39b32d2177ef903bd58f41d1522276f9a24ef069bd03ec0eadb71851
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - LightGBM
  - PCC
  - Staem5
  - XGBoost
  - iLearn
  - mlxtend
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# bioinformatics-classifier-metric-evaluation

## Summary

Systematic 10-fold cross-validation benchmarking of bioinformatics classifiers (base models and stacked ensembles) using Sn, Sp, Acc, MCC, and AUC, as done in Staem5 for m5C site prediction in M. musculus and A. thaliana.

## When to use

Use when comparing candidate classifiers (e.g., SVM, XGBoost, LightGBM, ExtraTree, GBDT), feature selection methods (F score, MRMD, PCC), or feature dimensions on a balanced sequence classification dataset and you need cross-validated Sn/Sp/Acc/MCC/AUC tables.

## When NOT to use

- The task is single-classifier deployment/inference on new sequences rather than comparative benchmarking (use the Staem5 m5C_predict.py prediction entry point instead).
- The dataset is heavily imbalanced or not amenable to 10-fold cross-validation, making Acc/MCC comparison across balanced folds misleading.
- You need a general cross-species m5C model — the article notes species-specific models are necessary and a general model is difficult to develop.

## Inputs

- Labeled training dataset of RNA sequences in FASTA format (e.g., M. musculus m5C set: 4,563 positive and 4,563 negative samples)
- Fused feature matrix from PSP, k-mer (k=1,2,3), PCPseDNC (38 physicochemical properties via iLearn), and PseEIIP encodings
- Feature rankings from F score, MRMD, or PCC selection methods
- Independent test sets (e.g., Test1157, DatasetHT) for held-out evaluation

## Outputs

- Per-dimension table of Sn, Sp, Acc, MCC, and AUC from 10-fold cross-validation for each classifier or feature-selection method
- Best feature dimension and best accuracy per selection method (e.g., F score: 77.42% at 185 features; MRMD: 77.21% at 230; PCC: 77.21% at 280)
- Selected base classifier combination and meta-classifier for the stacked model
- Comparison table against state-of-the-art predictors (e.g., iRNA-m5C_SVM, m5CPred-SVM, PEA-m5C) on training and independent test sets

## How to apply

Build the labeled feature matrix (e.g., Staem5's fused PSP, k-mer (k=1,2,3), PCPseDNC, and PseEIIP encodings of the 4,563 positive / 4,563 negative M. musculus m5C training set). Train each candidate model with scikit-learn, combining SVM, XGBoost, and LightGBM base classifiers via an mlxtend StackingClassifier with logistic regression as the meta-model. Optimize hyperparameters with Bayesian optimization, selecting the configuration that maximizes 10-fold cross-validation accuracy; add base classifiers incrementally and stop when accuracy decreases. Evaluate every model by 10-fold cross-validation, recording Sn, Sp, Acc, MCC, and AUC per feature-selection method and dimension (e.g., sweeping top-ranked features in steps of 5). Correctness is judged by reproducing the reported values: best 10-fold CV accuracy of 77.42% with F score at 185 features for M. musculus, and 73.89% accuracy / 0.479 MCC for the A. thaliana stacked model with SVM, XGBoost, and LightGBM.

## Related tools

- **Staem5** (Stacked ensemble framework (SVM, XGBoost, LightGBM base classifiers with logistic regression meta-classifier) whose models are evaluated under this skill) — https://github.com/Cxd-626/Staem5.git
- **scikit-learn** (Implements the base classifiers and 10-fold cross-validation; README pins sklearn == 0.21.3)
- **mlxtend** (Provides the stacking package used to combine base classifiers via logistic regression meta-classifier; README pins mlxtend == 0.17.3)
- **XGBoost** (Gradient-boosted base classifier in the stacked model being benchmarked)
- **LightGBM** (Light gradient boosting machine base classifier in the stacked model being benchmarked)

## Examples

```
from mlxtend.classifier import StackingClassifier; from sklearn.model_selection import cross_val_score; scores = cross_val_score(StackingClassifier(classifiers=[svm, xgb, lgbm], meta_classifier=LogisticRegression()), X_fused, y, cv=10, scoring='accuracy')
```

## Evaluation signals

- Reported Sn, Sp, Acc, MCC, and AUC per fold and averaged over the 10 folds for every model/dimension combination
- Best 10-fold cross-validation accuracy matches 77.42% with F score at 185 features (and 77.21% with MRMD at 230 and PCC at 280) for M. musculus within tolerance
- Stacked model with SVM, XGBoost, and LightGBM reaches 73.89% accuracy and 0.479 MCC on A. thaliana 10-fold CV
- Adding base classifiers beyond the optimal combination causes accuracy to decrease, confirming the stopping rule was applied
- Independent test results (e.g., Test1157: 157 positive / 1,000 negative) are consistent with CV rankings; note iRNA-m5C AUC may be missing (dash) in comparison tables

## Limitations

- Hyperparameters were pre-set according to previous experience rather than systematically derived, and the selected parameters appear only in supplementary Table S2
- Detailed performance metrics appear only in supplementary Tables S1 and S3, not the main text
- Feature selection comparison results are only available in supplementary Tables S5–S7
- Staem5 underperformed iRNA-m5C_SVM and m5CPred-SVM on the independent A. thaliana test set, so CV ranking does not always transfer to held-out data
- AUC is not reported for some competitors (e.g., iRNA-m5C on the independent test, marked with a dash), limiting full-metric comparisons

## Evidence

- [results] The model integrates four kinds of encoding schemes, i.e., position-specific propensity (PSP), k-mer (k = 1, 2, and 3), parallel correlation pseudo dinucleotide composition (PCPseDNC): "The model integrates four kinds of encoding schemes, i.e., position-specific propensity (PSP), k-mer (k = 1, 2, and 3), parallel correlation pseudo dinucleotide composition (PCPseDNC)"
- [discussion] we used logistic regression as the meta-model to ensemble the base classifiers into a stacked model: "we used logistic regression as the meta-model to ensemble the base classifiers into a stacked model"
- [discussion] performance comparison results in terms of accuracy of the five base classifiers before and after parameter optimization on the 10-fold cross-validation tests: "performance comparison results in terms of accuracy of the five base classifiers before and after parameter optimization on the 10-fold cross-validation tests"
- [discussion] The selection of base classifiers was based on the accuracy of the model: "The selection of base classifiers was based on the accuracy of the model"
