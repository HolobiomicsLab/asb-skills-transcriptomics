---
name: glmnet-lasso-model-fitting
description: Use when fitting lasso or elastic-net glmnet models on high-dimensional
  data (P >> n, e.g. transcriptomics) where unbiased performance estimation requires
  nested CV with feature selection embedded in the outer loop.
when_to_use_negative:
- Datasets large enough for a conventional 2/3-1/3 train/test split where a simple
  hold-out evaluation suffices (though the paper notes biomedical P >> n datasets
  are usually too small)
- When the goal is only hyperparameter tuning without unbiased performance estimation
  — plain cv.glmnet inner-loop tuning is sufficient and nested CV is unnecessary overhead
- Models outside glmnet/caret scope; use nestedcv's caret or SuperLearner wrappers
  (nestcv.train, nestcv.SuperLearner) instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: nestedcv
  role: Wraps glmnet in fully nested k x l-fold CV, embedding feature-selection filters
    within the outer loop to avoid information leakage
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the lasso/elastic-net regularised linear models, with cross-validation
    of both lambda and the elastic net alpha parameter
- name: caret
  role: Alternative backend via nestcv.train for accessing a large array of other
    prediction methods within the same nested CV framework
- name: pROC
  role: Computes ROC AUC on concatenated outer test-fold predictions (as in the paper's
    noise-simulation comparison)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:8a6aba5bf821aa7e8eb689493cdfe3fdd88d7842dfddd2614f8df2259398ae7c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - caret
  - glmnet
  - nestedcv
  - pROC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# glmnet-lasso-model-fitting

## Summary

Fitting lasso/elastic-net regularised linear models with glmnet inside a fully nested cross-validation loop (via the nestedcv R package), so that hyperparameter (lambda, alpha) tuning and any embedded feature-selection filters never leak information into the outer test folds used for performance estimation.

## When to use

Use when fitting lasso or elastic-net glmnet models on high-dimensional data (P >> n, e.g. transcriptomics) where unbiased performance estimation requires nested CV with feature selection embedded in the outer loop.

## When NOT to use

- Datasets large enough for a conventional 2/3-1/3 train/test split where a simple hold-out evaluation suffices (though the paper notes biomedical P >> n datasets are usually too small)
- When the goal is only hyperparameter tuning without unbiased performance estimation — plain cv.glmnet inner-loop tuning is sufficient and nested CV is unnecessary overhead
- Models outside glmnet/caret scope; use nestedcv's caret or SuperLearner wrappers (nestcv.train, nestcv.SuperLearner) instead

## Inputs

- Numeric predictor matrix x with P >> n predictors (e.g. transcriptomic features)
- Outcome vector y (binary for logistic lasso; multinomial factors also supported)
- Optional filter function (e.g. ttest_filter) and filter_options passed to nestcv.glmnet

## Outputs

- nestcv.glmnet result object with outer_result per fold, tuned alpha/lambda values, and left-out outer-fold predictions
- Concatenated outer test-fold predictions for unbiased ROC AUC / accuracy metrics (summary(res))
- Final model coefficients via coef(res) for prediction on external data

## How to apply

Partition the dataset into outer and inner CV folds (nestedcv default is 10 x 10 folds, stratified on the outcome variable via caret's createFolds()). Use nestcv.glmnet to fit the elastic net model: inner folds tune lambda and the alpha parameter, the model is refitted on the whole inner training data (optionally after a filter such as ttest_filter applied only to training folds), and is then tested on the left-out outer fold; concatenate outer test-fold predictions to compute accuracy/ROC AUC without bias. This design directly addresses the leakage bias demonstrated in the paper's gaussian-noise simulation, where whole-dataset t-test filtering inflated hold-out AUC while nested CV with training-fold-only filtering reported AUC near 0.50. After the nested loop, a final round of CV on the whole dataset determines hyperparameters for the final model to apply to external data. Parallelise the outer loop with cv.cores (mclapply on unix/mac, parLapply on Windows) or via the future framework.

## Related tools

- **nestedcv** (Wraps glmnet in fully nested k x l-fold CV, embedding feature-selection filters within the outer loop to avoid information leakage) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the lasso/elastic-net regularised linear models, with cross-validation of both lambda and the elastic net alpha parameter)
- **caret** (Alternative backend via nestcv.train for accessing a large array of other prediction methods within the same nested CV framework)
- **pROC** (Computes ROC AUC on concatenated outer test-fold predictions (as in the paper's noise-simulation comparison))

## Examples

```
res <- nestcv.glmnet(y, x, family = "binomial", filterFUN = ttest_filter, cv.cores = cores); summary(res)
```

## Evaluation signals

- On pure-noise simulation data (P >> n, random balanced binary outcome), concatenated outer-fold ROC AUC should be close to 0.50, unlike whole-dataset-filtered hold-out AUC (Fig. 2A red line)
- summary(res) reports outer-fold performance computed only on left-out outer test folds, with no filter fitted on test data
- Tuning traces exist for every outer fold (plot_alphas(res), plot_lambdas(res), plot(res$outer_result[[1]]$cvafit))
- Default fold structure is 10 x 10, stratified on the outcome; custom fold indices are honoured when supplied
- coef(res) returns a final model fitted on the whole dataset after a final round of CV for hyperparameters

## Limitations

- Feature-selection filters applied on the whole dataset before CV bias performance estimates; filters must be embedded inside the outer loop
- Small biomedical test sets suffer high variance if a single train/test split is used; nested CV mitigates this by using all samples in outer testing
- Elastic net alpha must itself be cross-validated; nestedcv handles this but plain glmnet usage does not
- Parallelisation of the outer loop is platform-dependent (mclapply on unix/mac, parLapply on Windows)

## Evidence

- [abstract] The nestedcv R package implements fully nested k × l-fold CV for lasso and elastic-net regularized linear models: "The nestedcv R package implements fully nested k × l-fold CV for lasso and elastic-net regularized linear models"
- [intro] The inner fold CV, (default is 10-fold), is used to tune optimal hyperparameters for models. Then the model is fitted on the whole inner fold and tested on the left-out data from the outer fold.: "The inner fold CV, (default is 10-fold), is used to tune optimal hyperparameters for models. Then the model is fitted on the whole inner fold and tested on the left-out data from the outer fold."
- [readme] Nested cross-validation (CV) for the glmnet and caret packages. With glmnet this includes cross-validation of elastic net alpha parameter.: "Nested cross-validation (CV) for the glmnet and caret packages. With glmnet this includes cross-validation of elastic net alpha parameter."
- [readme] Filters can be used by setting the filterFUN argument. Options for the filter function are passed as a list through filter_options.: "Filters can be used by setting the `filterFUN` argument. Options for the filter 
function are passed as a list through `filter_options`."
