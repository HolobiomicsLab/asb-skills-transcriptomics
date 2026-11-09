---
name: supervised-multiclass-classifier-training
description: Use when a labeled global RNA-seq gene-expression matrix (e.g., 19,737
  TCGA genes) must be mapped to predefined classes and no reference sample cohort
  is available, requiring a comparative ML survey of classifiers.
when_to_use_negative:
- The task is single-sample reference-based calling where an established predefined
  reference (e.g., AWCA-PAM50) is sufficient — this skill is a comparative training
  survey, not a reference-based caller.
- Input is unlabeled expression data with no subtype labels, since supervised training
  on five classes requires ground-truth labels.
- Only PAM50-panel genes are available and the goal is reproducing the original PAM50
  assay rather than surveying learners on the global transcriptome.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn GridSearchCV
  role: Exhaustive hyperparameter grid search with tenfold stratified cross-validation
    for each of the five learner Pipelines
- name: regularized multiclass Logistic Regression (mLR)
  role: Best-performing learner in the survey; combined with feature selection and
    embedded Lasso and Ridge regularizers
  repo: github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: Decision Forest
  role: Candidate multiclass learner benchmarked in the survey
- name: Decision Jungle
  role: Candidate multiclass learner benchmarked in the survey (86% CV accuracy baseline)
- name: Feed-Forward Neural Network
  role: Candidate multiclass learner benchmarked in the survey
- name: Support Vector Machine
  role: Candidate multiclass learner benchmarked in the survey
- name: pandas
  role: Loading the TCGA RNA-seq expression training matrix with subtype labels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:79634620c28fd37b89d603ec3cf76972ff5ee96bfaa46a4505164974378bb727
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Decision Forest
  - Decision Jungle
  - Feed-Forward Neural Network
  - Support Vector Machine
  - pandas
  - regularized multiclass Logistic Regression (mLR)
  - scikit-learn GridSearchCV
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# supervised-multiclass-classifier-training

## Summary

Train and benchmark five supervised multiclass learners (regularized multiclass Logistic Regression, Decision Forest, Decision Jungle, Feed-Forward Neural Network, SVM) with tenfold stratified cross-validation and hyperparameter grid search to assign breast cancer RNA-seq samples to the five intrinsic subtypes (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like).

## When to use

Use when a labeled global RNA-seq gene-expression matrix (e.g., 19,737 TCGA genes) must be mapped to predefined classes and no reference sample cohort is available, requiring a comparative ML survey of classifiers.

## When NOT to use

- The task is single-sample reference-based calling where an established predefined reference (e.g., AWCA-PAM50) is sufficient — this skill is a comparative training survey, not a reference-based caller.
- Input is unlabeled expression data with no subtype labels, since supervised training on five classes requires ground-truth labels.
- Only PAM50-panel genes are available and the goal is reproducing the original PAM50 assay rather than surveying learners on the global transcriptome.

## Inputs

- TCGA RNA-seq expression training matrix with 19,737 genes and intrinsic subtype labels (pandas DataFrame)
- Held-out TCGA test set expression matrix
- Hyperparameter grids for each of the five learners

## Outputs

- Best-scoring fitted estimator per learner (optimal hyperparameters from 10-fold GridSearchCV)
- Accuracy table (classifier_performance_results.csv) with one row per classifier and columns for CV accuracy and held-out test accuracy
- Predicted intrinsic subtype labels (Luminal A, Luminal B, Basal, Her2-enriched, Normal-like) for test samples

## How to apply

Load the TCGA RNA-seq training matrix (19,737 genes) with intrinsic subtype labels into a pandas DataFrame and configure scikit-learn Pipeline objects for the five learners, each coupled to a hyperparameter grid. Run tenfold stratified cross-validation with exhaustive grid search (GridSearchCV, cv=10) to select optimal hyperparameters per learner. Refit each best-scoring estimator on the full training set and predict subtype labels for the held-out TCGA test set, tabulating CV accuracy and test accuracy per classifier. For the winning learner, combine feature selection with embedded Lasso and Ridge regularizers in the regularized multiclass Logistic Regression; the survey's rationale is that mLR avoids the reference-cohort bias that affects standard PAM50. Validate against reported baselines (mLR 88% CV / 85% test accuracy; Decision Jungle 86% CV) and write the table to classifier_performance_results.csv with one row per classifier and columns for CV and test accuracy.

## Related tools

- **scikit-learn GridSearchCV** (Exhaustive hyperparameter grid search with tenfold stratified cross-validation for each of the five learner Pipelines)
- **regularized multiclass Logistic Regression (mLR)** (Best-performing learner in the survey; combined with feature selection and embedded Lasso and Ridge regularizers) — github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **Decision Forest** (Candidate multiclass learner benchmarked in the survey)
- **Decision Jungle** (Candidate multiclass learner benchmarked in the survey (86% CV accuracy baseline))
- **Feed-Forward Neural Network** (Candidate multiclass learner benchmarked in the survey)
- **Support Vector Machine** (Candidate multiclass learner benchmarked in the survey)
- **pandas** (Loading the TCGA RNA-seq expression training matrix with subtype labels)

## Examples

```
gs = GridSearchCV(pipe_mlr, param_grid, cv=10); gs.fit(X_train, y_train); acc = gs.best_estimator_.score(X_test, y_test)
```

## Evaluation signals

- CV accuracy of the best mLR model is ~88% and held-out TCGA test accuracy is ~85%, exceeding all other best-trained models (Decision Jungle ~86% CV).
- classifier_performance_results.csv exists with one row per classifier and columns for CV accuracy and test accuracy.
- Each learner's Pipeline was refit on the full training set with its GridSearchCV-selected best hyperparameters before test-set prediction.
- Subtype predictions cover exactly the five classes: Luminal A, Luminal B, Basal, Her2-enriched, Normal-like.
- Cross-validation used tenfold stratified splits (cv=10) so class proportions are preserved per fold.

## Limitations

- Published subtype labels were themselves obtained with the PAM50 assay, so higher accuracies with mLR using PAM50 genes or AWCA-PAM50 are biased by the nature of the published calls.
- The Normal-like class has undetermined clinical significance and causes degradation of classification performance.
- The few training samples and the vast number of features influence subtyping capabilities, mainly for the hardest-to-recognize cases; ambiguous samples poorly or comparably correlated to more than one class centroid reduce stability.

## Evidence

- [abstract] Among five supervised learners surveyed, regularized multiclass logistic regression (mLR) displayed the best performance, further increased by ad-hoc gene selection on the global transcriptome.: [evidence span withheld — anchor status: not_found]
- [discussion] particularly when a feature selection able to provide a feature space of relevant genes is combined with embedded Lasso and Ridge regularizers: "particularly when a feature selection able to provide a feature space of relevant genes is combined with embedded Lasso and Ridge regularizers"
- [results] The best-trained regularized multiclass logistic regression confirmed its primacy on the unseen TCGA test set samples, reaching 85% accuracy: [evidence span withheld — anchor status: not_found]
- [methods] Run tenfold stratified cross-validation with exhaustive grid search (scikit-learn GridSearchCV, cv=10) for each learner to select optimal hyperparameters per model.: [evidence span withheld — anchor status: not_found]
- [readme] our regularized multiclass Logistic Regression models ([mLR intrinsic subtyping](https://github.com/DEIB-GECO/BC_Intrinsic_subtyping/wiki/mLR_BC_subtyping)) to assign breast cancer (BC) samples to the five prognosis-related breast cancer _intrinsic_ subtypes: "our regularized multiclass Logistic Regression models ([mLR intrinsic subtyping](https://github.com/DEIB-GECO/BC_Intrinsic_subtyping/wiki/mLR_BC_subtyping))"
