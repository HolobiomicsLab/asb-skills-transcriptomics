---
name: cross-validation-grid-search-hyperparameter-tuning
description: Use when choosing among several candidate classifiers on a high-dimensional
  expression matrix (e.g. 19,737-gene TCGA RNA-seq with subtype labels) and you need
  unbiased per-model hyperparameter selection plus held-out generalization estimates.
when_to_use_negative:
- The labels themselves were produced by the same assay you are tuning against (e.g.
  PAM50-derived calls on the same gene panel), which the article notes biases accuracy
  estimates
- You need a single-sample classifier applied with a predefined external reference
  (AWCA-PAM50 or trained mLR models from the repo) rather than model selection among
  learners
- The dataset lacks matched subtype training labels or is not five-class intrinsic
  subtyping of breast cancer
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn GridSearchCV
  role: Runs exhaustive tenfold stratified cross-validation grid search to select
    optimal hyperparameters per learner
- name: regularized multiclass Logistic Regression (mLR)
  role: Best-performing learner in the survey, trained with feature selection combined
    with embedded Lasso and Ridge regularizers
  repo: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:cb66eb17a70e1ff8bc52b774b2818059d627833ca0347ade85cebfcbb2004d07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/BC_Intrinsic_subtyping
  tools:
  - regularized multiclass Logistic Regression (mLR)
  - scikit-learn GridSearchCV
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-validation-grid-search-hyperparameter-tuning

## Summary

Benchmarks multiple supervised learners (regularized multiclass logistic regression, Decision Forest, Decision Jungle, FFNN, SVM) on high-dimensional RNA-seq data by coupling each to a scikit-learn Pipeline hyperparameter grid and selecting best settings with tenfold stratified GridSearchCV, then comparing held-out test accuracy to pick the strongest classifier.

## When to use

Use when choosing among several candidate classifiers on a high-dimensional expression matrix (e.g. 19,737-gene TCGA RNA-seq with subtype labels) and you need unbiased per-model hyperparameter selection plus held-out generalization estimates.

## When NOT to use

- The labels themselves were produced by the same assay you are tuning against (e.g. PAM50-derived calls on the same gene panel), which the article notes biases accuracy estimates
- You need a single-sample classifier applied with a predefined external reference (AWCA-PAM50 or trained mLR models from the repo) rather than model selection among learners
- The dataset lacks matched subtype training labels or is not five-class intrinsic subtyping of breast cancer

## Inputs

- Training expression matrix (TCGA RNA-seq, 19,737 genes) with breast cancer intrinsic subtype labels in a pandas DataFrame
- Held-out TCGA test expression matrix
- Per-learner scikit-learn Pipeline definitions with hyperparameter grids for mLR, Decision Forest, Decision Jungle, FFNN, and SVM

## Outputs

- Fitted best-scoring estimators per learner (optimal hyperparameters selected by tenfold stratified CV)
- Accuracy table (classifier_performance_results.csv) with one row per classifier and columns for CV accuracy and held-out test accuracy

## How to apply

Load the expression matrix (e.g. 19,737 TCGA genes) with class labels into a pandas DataFrame, then build a scikit-learn Pipeline per learner — regularized multiclass Logistic Regression (with embedded Lasso/Ridge regularizers), Decision Forest, Decision Jungle, Feed-Forward Neural Network, and SVM — each paired with its hyperparameter grid. Run exhaustive GridSearchCV with cv=10 using stratified folds so each fold preserves subtype class proportions, a critical detail for unbalanced five-class intrinsic subtyping (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like). Refit each best-scoring estimator on the full training set and predict labels for the held-out TCGA test set, recording both CV accuracy and test accuracy per learner. Rationale: gene-selection/feature space combined with embedded Lasso and Ridge regularizers is what makes mLR generalize best on global transcriptome data, so the regularizer strength must be tuned, not fixed. Tabulate results with one row per classifier and columns for CV and test accuracy (reported baselines: mLR 88% CV / 85% test; Decision Jungle 86% CV).

## Related tools

- **scikit-learn GridSearchCV** (Runs exhaustive tenfold stratified cross-validation grid search to select optimal hyperparameters per learner)
- **regularized multiclass Logistic Regression (mLR)** (Best-performing learner in the survey, trained with feature selection combined with embedded Lasso and Ridge regularizers) — https://github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Examples

```
from sklearn.model_selection import GridSearchCV; gs = GridSearchCV(mlr_pipeline, param_grid, cv=10); gs.fit(X_train, y_train); acc = gs.best_estimator_.score(X_test, y_test)
```

## Evaluation signals

- Tenfold stratified CV is used (cv=10 in GridSearchCV) so subtype class proportions are preserved in every fold
- Tabulated baselines reproduce: mLR 88% CV accuracy / 85% held-out test accuracy, Decision Jungle 86% CV accuracy
- Output CSV has exactly one row per classifier with both CV accuracy and test accuracy columns
- mLR is the top-ranked model on the unseen TCGA test set, exceeding all four other best-trained learners
- Each refit estimator corresponds to the best mean CV score from its grid (grid search returned non-default tuned hyperparameters)

## Limitations

- Accuracies are biased because published subtype labels were themselves obtained with the PAM50 assay on the same gene panel
- Small number of training samples versus vast number of features limits subtyping capability for hard-to-recognize cases
- Normal-like class has undetermined clinical significance and degrades classification performance

## Evidence

- [abstract] Among five supervised learners surveyed, regularized multiclass logistic regression (mLR) displayed the best performance: [evidence span withheld — anchor status: not_found]
- [abstract] mLR classifications reached 90% concordance with PAM50-based calls, without need of reference sample: "mLR classifications reached 90% concordance with PAM50-based calls, without need of reference sample"
- [discussion] feature selection combined with Lasso and Ridge regularizers: "particularly when a feature selection able to provide a feature space of relevant genes is combined with embedded Lasso and Ridge regularizers"
- [discussion] the higher accuracies reached with mLRs using PAM50 genes or with the AWCA-PAM50 method are biased by the nature of the published subtype calls: "the higher accuracies reached with mLRs using PAM50 genes or with the AWCA-PAM50 method are biased by the nature of the published subtype calls"
- [readme] README: mLR intrinsic subtyping models: "or our regularized multiclass Logistic Regression models ([mLR intrinsic subtyping](https://github.com/DEIB-GECO/BC_Intrinsic_subtyping/wiki/mLR_BC_subtyping))"
