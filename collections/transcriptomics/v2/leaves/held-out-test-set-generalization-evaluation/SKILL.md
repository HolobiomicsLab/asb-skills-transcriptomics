---
name: held-out-test-set-generalization-evaluation
description: Use when a supervised classifier trained with 10-fold stratified CV on
  the TCGA 19,737-gene training matrix must be assessed for true generalization on
  unseen held-out TCGA test samples (e.g. confirming mLR's 85% test accuracy).
when_to_use_negative:
- Input is a single-sample classification task needing a deployable caller (use the
  AWCA/mLR subtyping code from the DEIB-GECO repository instead of re-running the
  survey)
- Only cross-validated performance is needed and no unseen test partition exists —
  test-set accuracy cannot be computed and CV accuracy would be a biased substitute
- Subtype labels themselves were produced by PAM50 on the same gene panel, so accuracy
  estimates inherit label bias and are not a clean generalization measure
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3473
tools:
- name: scikit-learn GridSearchCV
  role: Runs tenfold stratified cross-validation with exhaustive grid search (cv=10)
    to select optimal hyperparameters per learner before the held-out test evaluation
- name: pandas
  role: Loads the TCGA RNA-seq training matrix with intrinsic subtype labels and writes
    the accuracy table to classifier_performance_results.csv
- name: AWCA_PAM50 / mLR subtyping R-code (DEIB-GECO)
  role: Repository code providing the AWCA-PAM50 and regularized multiclass Logistic
    Regression single-sample callers whose five-subtype predictions the survey benchmarked
  repo: github.com/DEIB-GECO/BC_Intrinsic_subtyping
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:acc8fd677b9eec58c1554b08754e9d56ecc545ea059dde785a6bfef36d2e927f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AWCA_PAM50 / mLR subtyping R-code (DEIB-GECO)
  - pandas
  - scikit-learn GridSearchCV
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# held-out-test-set-generalization-evaluation

## Summary

Evaluates multiclass classifiers for breast cancer intrinsic subtyping (mLR, Decision Forest, Decision Jungle, FFNN, SVM) by refitting the best cross-validated estimator on the 19,737-gene TCGA training set and measuring generalization on a separate held-out TCGA test set, so that reported accuracy is not only cross-validation optimism.

## When to use

Use when a supervised classifier trained with 10-fold stratified CV on the TCGA 19,737-gene training matrix must be assessed for true generalization on unseen held-out TCGA test samples (e.g. confirming mLR's 85% test accuracy).

## When NOT to use

- Input is a single-sample classification task needing a deployable caller (use the AWCA/mLR subtyping code from the DEIB-GECO repository instead of re-running the survey)
- Only cross-validated performance is needed and no unseen test partition exists — test-set accuracy cannot be computed and CV accuracy would be a biased substitute
- Subtype labels themselves were produced by PAM50 on the same gene panel, so accuracy estimates inherit label bias and are not a clean generalization measure

## Inputs

- TCGA RNA-seq expression training matrix (19,737 genes) with breast cancer intrinsic subtype labels
- Held-out TCGA test set expression matrix with subtype labels (or unlabeled features plus reference labels for scoring)
- scikit-learn Pipeline definitions for the five learners with coupled hyperparameter grids

## Outputs

- classifier_performance_results.csv with one row per classifier and columns for CV accuracy and held-out test accuracy
- Per-learner best hyperparameter settings and fitted estimators
- Predicted intrinsic subtype labels for each held-out test sample

## How to apply

Configure each of the five learners (regularized multiclass Logistic Regression, Decision Forest, Decision Jungle, Feed-Forward Neural Network, SVM) as a scikit-learn Pipeline with its hyperparameter grid, and run GridSearchCV with cv=10 stratified folds on the TCGA training matrix of 19,737 genes. Take each learner's best-scoring estimator, refit it on the full training set, then predict intrinsic subtype labels (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) for the held-out TCGA test set. The rationale is that CV accuracy alone (e.g. mLR 88%, Decision Jungle 86%) can rank models differently than unseen data; the held-out test set is the arbitrating benchmark, and the article's primacy claim rests on mLR reaching 85% test accuracy over all other best-trained models. Tabulate CV and test accuracy per classifier and write them to classifier_performance_results.csv, checking the reported baselines before accepting any ranking.

## Related tools

- **scikit-learn GridSearchCV** (Runs tenfold stratified cross-validation with exhaustive grid search (cv=10) to select optimal hyperparameters per learner before the held-out test evaluation)
- **pandas** (Loads the TCGA RNA-seq training matrix with intrinsic subtype labels and writes the accuracy table to classifier_performance_results.csv)
- **AWCA_PAM50 / mLR subtyping R-code (DEIB-GECO)** (Repository code providing the AWCA-PAM50 and regularized multiclass Logistic Regression single-sample callers whose five-subtype predictions the survey benchmarked) — github.com/DEIB-GECO/BC_Intrinsic_subtyping

## Examples

```
best_model = GridSearchCV(pipeline, param_grid, cv=10, scoring='accuracy').fit(X_train, y_train); test_acc = best_model.score(X_test, y_test)
```

## Evaluation signals

- Regularized multiclass logistic regression reproduces the reported baseline of 88% CV accuracy and 85% held-out TCGA test accuracy, exceeding all other best-trained models on the test set
- Decision Jungle reproduces its reported 86% cross-validation accuracy as a secondary check of the CV protocol
- classifier_performance_results.csv contains exactly five rows (one per learner) with both a CV-accuracy and a test-accuracy column populated
- Every classifier is refit on the full training set after grid search before predicting test labels, so no test-sample information leaks into hyperparameter selection
- External concordance with PAM50-based calls approaches the article's reported ~90% level for mLR classifications

## Limitations

- Subtype labels are themselves PAM50-derived, so reported accuracies are biased toward PAM50-consistent calls and overstate absolute correctness
- The Normal-like class has undetermined clinical significance and degrades classification performance
- Few training samples relative to 19,737 gene features limit subtyping capability, especially for hard-to-recognize cases
- PAM50 accuracy depends heavily on the composition of the reference cohort, so test-set scores transfer poorly to cohorts with different composition

## Evidence

- [abstract] Among five supervised learners surveyed, regularized multiclass logistic regression (mLR) displayed the best performance, further increased by ad-hoc gene selection on the global transcriptome.: "regularized multiclass logistic regression (mLR) displayed the best performance, further increased by ad-hoc gene selection on the global transcriptome"
- [discussion] Regularized multiclass logistic regression with feature selection discussion.: "particularly when a feature selection able to provide a feature space of relevant genes is combined with embedded Lasso and Ridge regularizers"
- [readme] README describing the repository's five-subtype classification code.: "The R-code we provide allows to use our _AWCA_ versions of PAM50 method"
- [discussion] Biased labels limitation from the discussion.: "the higher accuracies reached with mLRs using PAM50 genes or with the AWCA-PAM50 method are biased by the nature of the published subtype calls"
