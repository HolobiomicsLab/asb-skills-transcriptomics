---
name: voting-classifier-ensemble-comparison
description: Use when benchmarking whether a stacked ensemble outperforms majority-vote
  aggregation over combinations of base classifiers (SVM, XGBoost, LightGBM, ExtraTree,
  GBDT) under 10-fold cross-validation.
when_to_use_negative:
- The goal is single-algorithm baseline comparison rather than ensemble-vs-ensemble
  comparison (the base classifiers are first compared individually with Bayesian-optimized
  parameters before ensembling).
- The task is m5C prediction on new query sequences rather than ensemble benchmarking
  — use the trained Staem5 model via m5C_predict.py instead.
- A general cross-species model is needed — the article notes species-specific models
  are necessary and no general cross-species model was developed.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Staem5
  role: The stacking ensemble framework in which base-classifier combinations are
    evaluated; source code and data for reproducing the comparison
  repo: https://github.com/Cxd-626/Staem5.git
- name: scikit-learn
  role: Trains the SVM (RBF kernel), GBDT, XGBoost, LightGBM, and ExtraTree base classifiers
    and generates out-of-fold predictions under 10-fold cross-validation
- name: mlxtend
  role: Implements the stacking models with logistic regression as the meta-classifier
    over each base-classifier combination
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:c5e896632cd8fa883daa2af407b382ebc3acecfa36851cf705f4fba53c363e43
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - Staem5
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
n_claims: 6
---

# voting-classifier-ensemble-comparison

## Summary

A benchmarking procedure from the Staem5 m5C prediction work that builds majority-vote voting classifiers and logistic-regression stacking ensembles over every combination of five base classifiers (SVM, XGBoost, LightGBM, ExtraTree, GBDT) and compares their 10-fold cross-validation performance to select the optimal ensemble.

## When to use

Use when benchmarking whether a stacked ensemble outperforms majority-vote aggregation over combinations of base classifiers (SVM, XGBoost, LightGBM, ExtraTree, GBDT) under 10-fold cross-validation.

## When NOT to use

- The goal is single-algorithm baseline comparison rather than ensemble-vs-ensemble comparison (the base classifiers are first compared individually with Bayesian-optimized parameters before ensembling).
- The task is m5C prediction on new query sequences rather than ensemble benchmarking — use the trained Staem5 model via m5C_predict.py instead.
- A general cross-species model is needed — the article notes species-specific models are necessary and no general cross-species model was developed.

## Inputs

- Balanced 41-bp RNA sequence training sets in FASTA-like form: A. thaliana (5,298 pos/5,298 neg) and M. musculus (4,563 pos/4,563 neg) derived from GSE90963, GSE93749, GSE83432
- Fused sequence feature vectors (PSP, k-mer k=1,2,3, PCPseDNC, PseEIIP) after F-score dimensionality reduction
- Out-of-fold predictions from the five tuned base classifiers under 10-fold cross-validation

## Outputs

- table2_stacking_vs_voting_cv_results.csv: voting-versus-stacking accuracy comparison per base-classifier combination for both species
- Per-combination performance metrics (Sn, Sp, accuracy, MCC, AUC) from 10-fold cross-validation
- Selected optimal base-classifier subset for the final stacked model (SVM + XGBoost + LightGBM)

## How to apply

Load the balanced 41-bp m5C training sets (A. thaliana: 5,298 pos/5,298 neg; M. musculus: 4,563 pos/4,563 neg) and train the five base classifiers — SVM with radial basis kernel (C/gamma tuned by Bayesian optimization), GBDT, XGBoost, LightGBM, and ExtraTree — in scikit-learn, generating out-of-fold predictions under 10-fold cross-validation. For every combination of base classifiers, build a majority-vote voting classifier and a corresponding stacking model that uses logistic regression as the meta-classifier, implemented with the mlxtend package. Compute Sn, Sp, accuracy, MCC, and AUC for each model from the cross-validation results. The decision rule follows the paper's base-classifier selection strategy: add classifiers to the stack only while accuracy increases, then stop — in the article this yielded SVM, XGBoost, and LightGBM as the final base set (73.89% accuracy, 0.479 MCC for A. thaliana). Tabulate the voting-vs-stacking accuracies for both species; stacking should slightly outperform voting for every combination.

## Related tools

- **Staem5** (The stacking ensemble framework in which base-classifier combinations are evaluated; source code and data for reproducing the comparison) — https://github.com/Cxd-626/Staem5.git
- **scikit-learn** (Trains the SVM (RBF kernel), GBDT, XGBoost, LightGBM, and ExtraTree base classifiers and generates out-of-fold predictions under 10-fold cross-validation)
- **mlxtend** (Implements the stacking models with logistic regression as the meta-classifier over each base-classifier combination)

## Examples

```
from mlxtend.classifier import StackingClassifier; stack = StackingClassifier(classifiers=[svm, xgb, lgbm], meta_classifier=LogisticRegression())
```

## Evaluation signals

- Stacking accuracy exceeds voting accuracy for every base-classifier combination (e.g., A. thaliana stacking 73.89% vs voting 73.68%; M. musculus stacking 77.12% vs voting 76.83%).
- The SVM+XGBoost+LightGBM stacking combination reproduces 73.89% accuracy and 0.479 MCC for A. thaliana under 10-fold cross-validation.
- All models report Sn, Sp, accuracy, MCC, and AUC computed from the same 10-fold cross-validation folds, so voting and stacking rows are directly comparable.
- The output table matches the schema of table2_stacking_vs_voting_cv_results.csv: one row per combination per species with voting and stacking accuracy columns.

## Limitations

- Hyperparameters were pre-set according to previous experience rather than systematically derived, and selected parameters are only available in supplementary Table S2.
- Detailed stacking comparison results are only provided in supplementary Table S3, not in the main text.
- No general cross-species model is feasible; models must be trained and evaluated separately per species (A. thaliana and M. musculus).
- Staem5 underperformed competing methods (iRNA-m5C_SVM and m5CPred-SVM) on the independent test set of A. thaliana.

## Evidence
