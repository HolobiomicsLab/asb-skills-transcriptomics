---
name: rna-seq-expression-feature-representation
description: Use when a TCGA-style RNA-seq expression matrix with five intrinsic subtype
  labels (Luminal A/B, Basal, Her2-enriched, Normal-like) must be turned into a classifier
  feature space for supervised single-sample subtyping.
when_to_use_negative:
- The task requires reference-based centroid classification (standard PAM50) rather
  than a trained single-sample caller, and a matched reference cohort is genuinely
  available
- No subtype labels exist for the expression data, since this representation depends
  on supervised training with the five intrinsic subtype classes
- Input is restricted to a small predefined gene panel (e.g. PAM50-only genes) when
  the goal is global-transcriptome feature selection with Lasso/Ridge regularization
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn Pipeline + GridSearchCV
  role: Implements the five learners (regularized mLR, Decision Forest, Decision Jungle,
    FFNN, SVM) with tenfold stratified CV and exhaustive hyperparameter grid search
    on the 19,737-gene matrix
- name: regularized multiclass Logistic Regression (mLR)
  role: Best-performing learner applied as a single-sample intrinsic subtyping model
    after ad-hoc gene selection on the global transcriptome
  repo: github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: AWCA_PAM50_BC_subtyping
  role: Alternative single-sample PAM50 variant (via AWCA to mitigate reference-cohort
    bias) offered in the same repository for comparison with the mLR caller
  repo: github.com/DEIB-GECO/BC_Intrinsic_subtyping
- name: limma
  role: Source of limma-derived gene signatures (including the limma50_BWE backward-elimination
    signature) used as reduced feature spaces in the learner survey
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-70832-2
    title: Machine learning for RNA sequencing-based intrinsic subtyping of breast
      cancer
  source_content_hash: sha256:9b5fb913700916ecf70c161cf945f73990572f317a16f8ea9cf10e0aebc01d73
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AWCA_PAM50_BC_subtyping
  - limma
  - regularized multiclass Logistic Regression (mLR)
  - scikit-learn Pipeline + GridSearchCV
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rna-seq-expression-feature-representation

## Summary

Build a supervised-learning feature representation for breast cancer intrinsic subtyping from the global TCGA RNA-seq transcriptome (19,737 genes), where ad-hoc gene selection plus Lasso/Ridge-regularized multiclass logistic regression (mLR) outperformed Decision Forest, Decision Jungle, FFNN and SVM learners. It replaces reference-cohort-dependent PAM50 centroid correlation with single-sample classification on the full gene expression matrix.

## When to use

Use when a TCGA-style RNA-seq expression matrix with five intrinsic subtype labels (Luminal A/B, Basal, Her2-enriched, Normal-like) must be turned into a classifier feature space for supervised single-sample subtyping.

## When NOT to use

- The task requires reference-based centroid classification (standard PAM50) rather than a trained single-sample caller, and a matched reference cohort is genuinely available
- No subtype labels exist for the expression data, since this representation depends on supervised training with the five intrinsic subtype classes
- Input is restricted to a small predefined gene panel (e.g. PAM50-only genes) when the goal is global-transcriptome feature selection with Lasso/Ridge regularization

## Inputs

- TCGA RNA-seq gene expression training matrix covering 19,737 genes with breast cancer intrinsic subtype labels
- Held-out TCGA RNA-seq test-set expression samples
- Optional gene signature definitions (PAM50 genes, limma-derived signatures, limma50_BWE backward-elimination signature)

## Outputs

- Fitted single-sample subtype classifier assigning each sample to Luminal A, Luminal B, Basal, Her2-enriched, or Normal-like
- Accuracy table with one row per classifier and columns for tenfold CV accuracy and held-out test accuracy (e.g. classifier_performance_results.csv)

## How to apply

Load the TCGA RNA-seq expression training matrix (19,737 genes) with intrinsic subtype labels into a pandas DataFrame. Configure scikit-learn Pipeline objects for the five learners — regularized multiclass Logistic Regression, Decision Forest, Decision Jungle, Feed-Forward Neural Network, and Support Vector Machine — each coupled to a hyperparameter grid. Run tenfold stratified cross-validation with GridSearchCV (cv=10) per learner, optionally combining feature selection with embedded Lasso and Ridge regularizers, since mLR with ad-hoc gene selection on the global transcriptome was the top performer. Refit the best estimator on the full training set and predict held-out TCGA test-set labels, checking against reported baselines (mLR 88% CV / 85% test; Decision Jungle 86% CV). The rationale is to avoid standard PAM50's dependence on the composition of the cohort used for reference construction.

## Related tools

- **scikit-learn Pipeline + GridSearchCV** (Implements the five learners (regularized mLR, Decision Forest, Decision Jungle, FFNN, SVM) with tenfold stratified CV and exhaustive hyperparameter grid search on the 19,737-gene matrix)
- **regularized multiclass Logistic Regression (mLR)** (Best-performing learner applied as a single-sample intrinsic subtyping model after ad-hoc gene selection on the global transcriptome) — github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **AWCA_PAM50_BC_subtyping** (Alternative single-sample PAM50 variant (via AWCA to mitigate reference-cohort bias) offered in the same repository for comparison with the mLR caller) — github.com/DEIB-GECO/BC_Intrinsic_subtyping
- **limma** (Source of limma-derived gene signatures (including the limma50_BWE backward-elimination signature) used as reduced feature spaces in the learner survey)

## Examples

```
df = pd.read_csv('tcga_train_19737.csv'); GridSearchCV(make_pipeline(LogisticRegression(penalty='l2', multi_class='multinomial')), param_grid, cv=10).fit(X, y)
```

## Evaluation signals

- Feature matrix contains the expected 19,737 gene columns with five-class intrinsic subtype labels
- Best mLR model reproduces the reported baselines of ~88% tenfold CV accuracy and ~85% held-out TCGA test accuracy
- Decision Jungle baseline reaches ~86% CV accuracy, and no other learner exceeds mLR's test accuracy
- Accuracy table written with exactly one row per classifier and columns for CV accuracy and test accuracy
- External-validation mLR calls reach ~90% concordance with PAM50-based labels without needing a reference sample

## Limitations

- The few amounts of training samples and the vast number of features influence subtyping capabilities, mainly for the hardest-to-recognize cases
- Performance is partly biased because published subtype labels were themselves obtained with the PAM50 assay, inflating accuracy of PAM50-gene-based models
- The Normal-like class, whose clinical significance remains undetermined, causes degradation of classification performance
- Standard PAM50 reference construction depends on cohort composition, limiting robustness and reproducibility of comparisons against it

## Evidence

- [other] workflow step 1: [evidence span withheld — anchor status: not_found]
- [other] workflow step 3: [evidence span withheld — anchor status: not_found]
- [abstract] mLR feature selection finding: "regularized multiclass logistic regression (mLR) displayed the best performance, further increased by ad-hoc gene selection on the global transcriptome"
- [discussion] feature space with regularizers: "particularly when a feature selection able to provide a feature space of relevant genes is combined with embedded Lasso and Ridge regularizers"
- [readme] repository usage: [evidence span withheld — anchor status: not_found]
