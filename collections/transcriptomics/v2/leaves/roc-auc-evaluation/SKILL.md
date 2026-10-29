---
name: roc-auc-evaluation
description: Use when you need an unbiased performance estimate (ROC AUC) for a glmnet/caret
  model fitted on high-dimensional data (P >> n) by concatenating left-out outer CV
  fold predictions, or when comparing outer vs inner CV AUC to detect optimistic bias
  from whole-dataset filtering or unbalanced resampling.
when_to_use_negative:
- Performance measured on left-out inner CV folds should not be reported as the model's
  unbiased accuracy — it shares data with the filter and hyperparameter tuning and
  is optimistically biased
- AUC computed after whole-dataset preprocessing (e.g. randomsample oversampling or
  t-test filtering applied outside the CV loop) is leakage-contaminated and not a
  valid performance estimate
- Multinomial problems beyond binary classification need other metrics; ROC curves
  here apply to binary comparisons (e.g. iris 3-class example uses multinomial family
  without ROC AUC)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3518
tools:
- name: nestedcv
  role: Runs fully nested 10x10-fold CV with glmnet, ensuring filters and resampling
    are nested within the outer loop so outer-fold predictions give unbiased AUC;
    exposes outer and innercv ROC metrics
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the lasso/elastic-net logistic regression whose outer-fold test predictions
    are scored by AUC
  repo: https://CRAN.R-project.org/package=nestedcv
- name: pROC
  role: Computes ROC AUC on concatenated outer-fold test predictions and on inner-fold
    predictions
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:e8c540924a4a350d0ba6e8363812c106a1150ffa9005b7e6300103565ad8d3a1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  - pROC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# roc-auc-evaluation

## Summary

Evaluates binary classification model performance by ROC AUC computed on concatenated left-out outer-fold test predictions from nested CV (via pROC), yielding an unbiased performance estimate, and distinguishes it from optimistically biased inner-fold AUC.

## When to use

Use when you need an unbiased performance estimate (ROC AUC) for a glmnet/caret model fitted on high-dimensional data (P >> n) by concatenating left-out outer CV fold predictions, or when comparing outer vs inner CV AUC to detect optimistic bias from whole-dataset filtering or unbalanced resampling.

## When NOT to use

- Performance measured on left-out inner CV folds should not be reported as the model's unbiased accuracy — it shares data with the filter and hyperparameter tuning and is optimistically biased
- AUC computed after whole-dataset preprocessing (e.g. randomsample oversampling or t-test filtering applied outside the CV loop) is leakage-contaminated and not a valid performance estimate
- Multinomial problems beyond binary classification need other metrics; ROC curves here apply to binary comparisons (e.g. iris 3-class example uses multinomial family without ROC AUC)

## Inputs

- Binary outcome vector y and predictor matrix x (e.g. RNA-seq expression matrix, simulated imbalanced data with 150 samples x 20,000 predictors, Sonar Class/feature columns)
- Fitted nestedcv model object (nestcv.glmnet output) with summary metrics and outer_result predictions
- Concatenated outer-fold test predictions with matched true class labels

## Outputs

- Outer-fold ROC AUC from the nestcv.glmnet object's summary metrics
- Inner-CV (innercv_roc) AUC for bias comparison
- Side-by-side AUC comparison tables written to CSV (e.g. imbalance_auc_comparison.csv, noise_simulation_auc.csv, rituximab_nestedcv_auc.csv)

## How to apply

Fit the model with nestcv.glmnet (default 10 x 10 stratified outer/inner folds, stratified on the outcome using caret's createFolds); the inner loop tunes hyperparameters (lambda, optionally alpha) while the outer loop produces test predictions on left-out folds. Extract the outer-fold ROC AUC from the fitted object's summary metrics, and separately extract the innercv_roc AUC, which shares training data with any filter and is therefore optimistically biased. If predictions were assembled manually, concatenate the outer-fold test predictions across folds against true outcomes and compute ROC AUC with pROC (e.g. roc(response, predictor)). Validate unbiasedness by checking that the outer-fold AUC is lower than the inner-fold AUC (e.g. R4RA rituximab example: outer 0.783 vs inner 0.863) and, on pure-noise data, that nested CV with filters embedded in the outer loop returns AUC near 0.50. Tabulate side-by-side AUC comparisons (e.g. unnested vs nested oversampling) and write to CSV for reporting.

## Related tools

- **nestedcv** (Runs fully nested 10x10-fold CV with glmnet, ensuring filters and resampling are nested within the outer loop so outer-fold predictions give unbiased AUC; exposes outer and innercv ROC metrics) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the lasso/elastic-net logistic regression whose outer-fold test predictions are scored by AUC) — https://CRAN.R-project.org/package=nestedcv
- **pROC** (Computes ROC AUC on concatenated outer-fold test predictions and on inner-fold predictions)

## Examples

```
res <- nestcv.glmnet(y, x, filterFUN = ttest_filter, filter_options = list(nfilter = 300)); summary(res)  # outer-fold vs innercv_roc AUC
```

## Evaluation signals

- On pure gaussian-noise data with P >> n, outer-fold AUC from fully nested CV with embedded ttest_filter should be close to 0.50, while whole-dataset filtering yields substantially inflated AUC
- Outer-fold AUC must be lower than the innercv_roc AUC for the same fitted object (e.g. 0.783 vs 0.863 in the R4RA rituximab example)
- Unnested whole-dataset oversampling produces higher AUC than balance-nested oversampling on the imbalanced 150 x 20,000-predictor simulation — the gap is the leakage bias
- Outer test predictions concatenated across all outer folds must cover every sample exactly once (no sample appears in both training and test contributions to the AUC)

## Limitations

- Whole-dataset filtering or balancing before CV creates information leakage and upward bias in apparent AUC; all preprocessing must be nested within the outer loop
- Inner CV fold AUC is optimistically biased and must not be reported as unbiased model performance
- Biomedical datasets with limited sample size give small outer test sets; nested CV is used precisely because a conventional 2/3-1/3 split yields unstable accuracy estimates

## Evidence

- [abstract] Inner CV is used to tune models and outer CV is used to determine model performance without bias: "Inner CV is used to tune models and outer CV is used to determine model performance without bias"
- [abstract] filters nested in outer loop avoid leakage: "the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] outer test predictions concatenated: "the unseen test predictions from the outer folds are compared against the true results for the outer test folds and the results concatenated"
- [intro] whole-dataset filtering creates bias: "it has been demonstrated that filtering on the whole dataset creates a bias when determining accuracy of models (Vabalas et al, 2019)"
- [readme] README: ROC curves from left-out folds: [evidence span withheld — anchor status: external_doc]
