---
name: imbalanced-class-oversampling-bias-analysis
description: Use when an imbalanced binary classification dataset (e.g. 150 samples,
  20,000 predictors, weak signal in ~30 predictors) is being balanced before model
  fitting and you must check whether balancing outside the CV loop leaks data and
  inflates ROC AUC.
when_to_use_negative:
- The dataset is already balanced, so oversampling is unnecessary and the leakage
  comparison is moot
- The model is not evaluated via nested cross-validation (e.g. a single train/test
  split), so 'outer-fold test predictions' cannot be concatenated
- The goal is unbiased performance estimation only — use plain nestcv.glmnet rather
  than a leakage-bias comparison
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: nestedcv
  role: Provides nestcv.glmnet with the balance argument to apply randomsample oversampling
    to training data inside the outer CV folds, enabling the unnested-vs-nested comparison
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the lasso logistic regression model whose apparent performance is compared
    between balancing strategies
- name: pROC
  role: Computes ROC AUC from concatenated outer-fold test predictions of each run
- name: R
  role: Execution environment for the comparison workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:0df3f9fa61083e144f8d43feaf2e13afbf0bd3f2e9db2ffcfb03d5ccfa1bb28a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  - pROC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# imbalanced-class-oversampling-bias-analysis

## Summary

Quantifies upward bias in apparent model performance caused by class-balancing (random oversampling, SMOTE) applied outside the nested CV loop versus balancing nested within outer CV folds, using nestedcv's nestcv.glmnet on imbalanced high-dimensional binary data.

## When to use

Use when an imbalanced binary classification dataset (e.g. 150 samples, 20,000 predictors, weak signal in ~30 predictors) is being balanced before model fitting and you must check whether balancing outside the CV loop leaks data and inflates ROC AUC.

## When NOT to use

- The dataset is already balanced, so oversampling is unnecessary and the leakage comparison is moot
- The model is not evaluated via nested cross-validation (e.g. a single train/test split), so 'outer-fold test predictions' cannot be concatenated
- The goal is unbiased performance estimation only — use plain nestcv.glmnet rather than a leakage-bias comparison

## Inputs

- Imbalanced binary classification dataset (e.g. 150 x 20,000 predictor matrix with weak signal in the first 30 predictors)
- Class labels y (two-class factor, imbalanced)
- R environment with nestedcv, glmnet, and pROC installed

## Outputs

- imbalance_auc_comparison.csv tabulating ROC AUC of unnested oversampling vs nested balance argument
- Concatenated outer-fold test predictions for each balancing strategy

## How to apply

Simulate or load the imbalanced binary dataset (per the nestedcv vignette: 150 samples, 20,000 predictors, only the first 30 predictors carrying weak signal, imbalanced classes). Run nestcv.glmnet (lasso logistic regression) twice: once after applying randomsample oversampling to the whole dataset so balancing happens outside the CV loops, and once with the balance argument so randomsample oversampling is applied only to training data inside the outer CV folds. Concatenate the left-out outer-fold test predictions from each run and compute ROC AUC with pROC. Tabulate both AUC values side by side; the unnested-oversampling AUC is expected to be upward-biased due to data leakage, while the nested-balancing AUC reflects unbiased performance.

## Related tools

- **nestedcv** (Provides nestcv.glmnet with the balance argument to apply randomsample oversampling to training data inside the outer CV folds, enabling the unnested-vs-nested comparison) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the lasso logistic regression model whose apparent performance is compared between balancing strategies)
- **pROC** (Computes ROC AUC from concatenated outer-fold test predictions of each run)
- **R** (Execution environment for the comparison workflow)

## Examples

```
res_nested <- nestcv.glmnet(y, x, family = "binomial", balance = "randomsample"); res_unnest <- nestcv.glmnet(y, randomsample(y, x)$x, family = "binomial")
```

## Evaluation signals

- The unnested-oversampling AUC is higher than the nested-balance AUC (upward bias from leakage) on the imbalanced simulated data
- Both AUC values are computed on concatenated left-out outer-fold test predictions, not on resampled data
- imbalance_auc_comparison.csv exists with the two AUC values tabulated side by side
- In the nested run, oversampling is confirmed to occur only on training data within outer folds (via the balance argument), never on outer test folds

## Limitations

- Result is demonstrated on a simulated dataset from the nestedcv vignette; magnitude of the upward bias depends on the imbalance ratio and signal strength (weak signal in only 30 of 20,000 predictors here)
- Biased AUC from unnested balancing may still look plausible, so the bias is only detectable by the side-by-side nested comparison
- nestedcv stratifies outer & inner CV folds on the outcome by default; changing fold construction may alter the comparison

## Evidence

- [other] unnested oversampling causes leakage: [evidence span withheld — anchor status: not_found]
- [abstract] nested CV avoids bias: "Inner CV is used to tune models and outer CV is used to determine model performance without bias"
- [abstract] leakage from performance test sets: "the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] outer fold predictions concatenated: "the unseen test predictions from the outer folds are compared against the true results for the outer test folds and the results concatenated"
- [intro] stratified folds: "By default `nestedcv` stratifies outer & inner CV folds based on the outcome variable using caret's `createFolds()` function."
