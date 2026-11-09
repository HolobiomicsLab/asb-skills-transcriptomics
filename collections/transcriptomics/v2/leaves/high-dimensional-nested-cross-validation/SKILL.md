---
name: high-dimensional-nested-cross-validation
description: Use when a dataset has far more predictors than samples (P >> n, e.g.
  RNA-seq with >16,000 genes in n=68 samples) and you need hyperparameter tuning plus
  an unbiased estimate of model accuracy or AUC.
when_to_use_negative:
- A held-out external test set already exists, so unbiased performance can be measured
  directly without nested CV
- Predictor count is modest relative to sample size (P not >> n), making a conventional
  2/3-1/3 split or simple CV sufficient
- You plan to filter or select predictors on the whole dataset before CV — the article
  notes this 'creates a bias when determining accuracy of models'
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: nestedcv
  role: Core implementation of the nested k × l-fold CV loop, fold stratification,
    embedded filters, and outer-CV performance summary
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the lasso/elastic-net models; nestcv.glmnet additionally cross-validates
    the alpha and lambda hyperparameters in the inner folds
- name: caret
  role: Alternative engine via nestcv.train; caret's createFolds() provides the outcome-stratified
    outer and inner folds
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:9894ac5f6f699ed2a989593889a3013bcde8ea73af537b4dc73fd5c9fd6d904a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - caret
  - glmnet
  - nestedcv
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# high-dimensional-nested-cross-validation

## Summary

Fully nested k × l-fold cross-validation (default 10 × 10) for lasso/elastic-net models via nestcv.glmnet, in which inner folds tune hyperparameters (alpha, lambda) and outer-fold test predictions yield an unbiased performance estimate for P ≫ n transcriptomic data.

## When to use

Use when a dataset has far more predictors than samples (P >> n, e.g. RNA-seq with >16,000 genes in n=68 samples) and you need hyperparameter tuning plus an unbiased estimate of model accuracy or AUC.

## When NOT to use

- A held-out external test set already exists, so unbiased performance can be measured directly without nested CV
- Predictor count is modest relative to sample size (P not >> n), making a conventional 2/3-1/3 split or simple CV sufficient
- You plan to filter or select predictors on the whole dataset before CV — the article notes this 'creates a bias when determining accuracy of models'

## Inputs

- Outcome vector y (factor or binary class labels, e.g. R4RA treatment response)
- Predictor matrix x of high-dimensional features (e.g. >16,000 genes from E-MTAB-11611 RNA-seq after the median-expression cut-off, n=68 samples)
- Optional filter specification (filterFUN plus filter_options, e.g. ttest_filter nfilter=100)
- Optional custom fold indices or cv.cores setting for the outer loop

## Outputs

- nestedcv fit object with per-outer-fold tuned alpha/lambda and test predictions
- Unbiased outer-CV performance metrics (AUC, accuracy, balanced accuracy) computed on concatenated left-out outer-fold predictions, e.g. written to cv_performance_metrics.csv
- Final model fitted on the whole dataset after a final round of CV (coefficients accessible via coef())
- Diagnostic plots: plot_alphas(res), plot_lambdas(res), and per-fold glmnet CV plots

## How to apply

Prepare an outcome vector y and predictor matrix x (for the R4RA example: E-MTAB-11611 RNA-seq after applying the median-expression gene cut-off and subsetting to the rituximab arm, n=68). Call nestcv.glmnet, which partitions data into default 10 x 10 stratified outer/inner folds using caret's createFolds() stratified on the outcome; the inner 10-fold CV tunes the elastic-net alpha and lambda parameters, then the model fitted on the inner training data is tested on the left-out outer fold. Optionally supply filterFUN (e.g. ttest_filter with nfilter = 100) so feature selection is embedded inside the outer loop, preventing information leakage from performance test sets; the task card demonstrates the no-filter variant. Concatenate the left-out outer-fold test predictions and compute AUC, accuracy and balanced accuracy; optionally run a final round of CV on the whole dataset to fit the final model for prediction on external data. Parallelise the outer loop with cv.cores (mclapply on unix/mac, parLapply on windows).

## Related tools

- **nestedcv** (Core implementation of the nested k × l-fold CV loop, fold stratification, embedded filters, and outer-CV performance summary) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the lasso/elastic-net models; nestcv.glmnet additionally cross-validates the alpha and lambda hyperparameters in the inner folds)
- **caret** (Alternative engine via nestcv.train; caret's createFolds() provides the outcome-stratified outer and inner folds)

## Examples

```
cores <- parallel::detectCores(logical = FALSE); res <- nestcv.glmnet(y, x, cv.cores = cores); summary(res)
```

## Evaluation signals

- Reproduce the R4RA rituximab (n=68, no filter) outer-CV benchmark: AUC 0.7648, accuracy 0.7059, balanced accuracy 0.6773, and match cv_performance_metrics.csv to the nestedcv vignette values
- Confirm each outer fold's test data was left out exactly once and predictions were concatenated before computing metrics (no sample scored on its own training fold)
- Verify folds are stratified on the outcome via caret's createFolds(), default 10 outer x 10 inner folds
- If filterFUN is used, confirm filtering ran inside the outer CV loop on training data only, not on the full dataset
- Inspect tuning behaviour with plot_alphas(res) and plot_lambdas(res); each outer fold should show a converged inner-CV fit for alpha and lambda

## Limitations

- Nested CV is motivated by small biomedical datasets where a 2/3-1/3 split leaves too small a test set; with large n, simpler designs may be preferred
- Filtering predictors on the whole dataset creates bias when determining model accuracy — selection must stay nested within the outer CV loop
- Outer-loop parallelisation uses mclapply on unix/mac and parLapply on windows, so behaviour differs by platform

## Evidence

- [abstract] The nestedcv R package implements fully nested k × l-fold CV for lasso and elastic-net regularized linear models via the glmnet package: "The nestedcv R package implements fully nested k × l-fold CV for lasso and elastic-net regularized linear models"
- [abstract] Inner CV is used to tune models and outer CV is used to determine model performance without bias: "Inner CV is used to tune models and outer CV is used to determine model performance without bias"
- [intro] Fold stratification and default fold counts: "By default `nestedcv` stratifies outer & inner CV folds based on the outcome variable using caret's `createFolds()` function."
- [readme] Filter options passed via filterFUN: "Filters can be used by setting the `filterFUN` argument. Options for the filter 
function are passed as a list through `filter_options`."
- [intro] Parallelisation of the outer loop: "all allow parallelisation of the outer CV loop using `mclapply` from the
`parallel` package on unix/mac and `parLapply`"
