---
name: recursive-feature-elimination-with-cross-validation
description: Use when a tabular feature matrix (e.g., 57 SIF/SSF/PF columns for mRNA
  vs lncRNA transcripts) must be reduced before training an XGBoost model, and class
  labels are imbalanced so oversampling must precede elimination.
when_to_use_negative:
- The input is already a compact, curated feature set (e.g., the 6 selected SSFs),
  so the CV elimination curve adds no value.
- Data are raw RNA sequences (FASTA) rather than an extracted tabular feature matrix
  — feature extraction (SIFs, SSFs, PFs) must happen first.
- The classes are already balanced and the domain tolerates keeping all features,
  making SMOTE-based elimination unnecessary.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Provides sklearn.feature_selection.RFECV used for recursive feature elimination
    with tenfold CV and accuracy scoring (v1.1.3)
- name: XGBoost
  role: Classifier wrapped by RFECV as the feature-importance estimator during elimination
    (v1.7.1)
- name: imbalanced-learn
  role: SMOTE oversampling applied before RFECV so elimination is not biased toward
    the majority class (v0.9.1)
- name: LncDC
  role: Downstream classifier whose XGBoost model consumes the selected 28-feature
    subset
  repo: https://github.com/lim74/LncDC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:88933b4f584f5f523828aaf7ee4790e3e74c41b44cc25b04e9b5b4cb9b1e43dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - LncDC
  - Python
  - XGBoost
  - imbalanced-learn
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# recursive-feature-elimination-with-cross-validation

## Summary

Select a parsimonious feature subset for the LncDC XGBoost classifier by running sklearn RFECV with tenfold cross-validation on the SMOTE-balanced H-Train matrix of 57 features (SIFs, SSFs, PFs), picking the smallest subset whose CV accuracy is within a negligible margin of the maximum.

## When to use

Use when a tabular feature matrix (e.g., 57 SIF/SSF/PF columns for mRNA vs lncRNA transcripts) must be reduced before training an XGBoost model, and class labels are imbalanced so oversampling must precede elimination.

## When NOT to use

- The input is already a compact, curated feature set (e.g., the 6 selected SSFs), so the CV elimination curve adds no value.
- Data are raw RNA sequences (FASTA) rather than an extracted tabular feature matrix — feature extraction (SIFs, SSFs, PFs) must happen first.
- The classes are already balanced and the domain tolerates keeping all features, making SMOTE-based elimination unnecessary.

## Inputs

- H-Train feature matrix with 57 columns (SIFs, SSFs, PFs) in a pandas DataFrame
- mRNA/lncRNA class labels for the same transcripts (imbalanced)
- XGBoost 1.7.1 classifier as the base estimator inside RFECV

## Outputs

- Optimal 28-feature subset (6 SSFs + 14 SIFs + 8 PFs) for the LncDC model
- Tenfold CV accuracy-versus-feature-count curve
- feature_selection_results.csv with selected feature names, group labels, and the CV accuracy curve

## How to apply

Load the 57-feature H-Train matrix (SIFs, SSFs, PFs) and mRNA/lncRNA class labels into pandas 1.5/numpy 1.23 DataFrames, then apply SMOTE (imbalanced-learn 0.9.1) so recursive elimination is not biased toward the majority class. Run sklearn.feature_selection.RFECV (scikit-learn 1.1.3) with an XGBoost 1.7.1 classifier, eliminating one feature per iteration under tenfold cross-validation with accuracy as the scoring metric to build an accuracy-versus-feature-count curve. The article's curve peaked at 53 features (0.9824), but the 28-feature subset scored 0.9823 — only 0.0001 lower — so the smaller subset was retained for LncDC. Tally the selected 28 features by group and confirm the composition: exactly 6 secondary structure features, 14 sequence intrinsic features, and 8 protein features.

## Related tools

- **scikit-learn** (Provides sklearn.feature_selection.RFECV used for recursive feature elimination with tenfold CV and accuracy scoring (v1.1.3))
- **XGBoost** (Classifier wrapped by RFECV as the feature-importance estimator during elimination (v1.7.1))
- **imbalanced-learn** (SMOTE oversampling applied before RFECV so elimination is not biased toward the majority class (v0.9.1))
- **LncDC** (Downstream classifier whose XGBoost model consumes the selected 28-feature subset) — https://github.com/lim74/LncDC

## Examples

```
from sklearn.feature_selection import RFECV; from xgboost import XGBClassifier; sel = RFECV(XGBClassifier(), step=1, cv=10, scoring='accuracy'); sel.fit(X_smote, y_smote)
```

## Evaluation signals

- The tenfold CV accuracy of the selected 28-feature subset reproduces the reported 0.9823 (within ~0.001).
- The accuracy-versus-feature-count curve shows a plateau: the 53-feature maximum (0.9824) is only 0.0001 above the 28-feature score.
- The selected subset decomposes into exactly 6 secondary structure features + 14 sequence intrinsic features + 8 protein features.
- SMOTE was applied to the training labels before RFECV, so both mRNA and lncRNA classes contribute equally to the CV folds.
- feature_selection_results.csv contains feature names, group labels, and the full CV accuracy curve.

## Limitations

- The selection sits on a plateau — the 28-feature subset is only 0.0001 below the 53-feature optimum, so the exact cut point is a parsimony judgment, not a sharp minimum.
- Further accuracy improvement was difficult because SIFs and PFs had already achieved a high accuracy, limiting the marginal value of additional features.
- RFECV with one-feature-per-iteration elimination over 57 features is computationally expensive, especially when secondary structure features require RNAfold calculation.

## Evidence

- [intro] RFECV with XGBoost and SMOTE: [evidence span withheld — anchor status: not_found]
- [methods] RFECV run parameters: [evidence span withheld — anchor status: not_found]
- [results] Selected subset accuracy and composition rationale: [evidence span withheld — anchor status: not_found]
