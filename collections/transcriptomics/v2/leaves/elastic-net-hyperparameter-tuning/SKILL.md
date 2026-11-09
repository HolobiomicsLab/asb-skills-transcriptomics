---
name: elastic-net-hyperparameter-tuning
description: Use when fitting regularised glmnet models on high-dimensional data (P
  >> n, e.g. >16,000-gene RNA-seq with n=68) and both the mixing parameter alpha and
  shrinkage lambda must be cross-validated without biasing outer-CV accuracy.
when_to_use_negative:
- When the goal is a conventional 2/3-1/3 train/test split evaluation on a dataset
  large enough to support it rather than CV-based tuning
- When filtering predictors on the whole dataset before CV is intended — the article
  warns this creates bias when determining model accuracy
- When the model family is not supported by glmnet or caret (nestedcv's supported
  backends)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: nestedcv
  role: Provides nestcv.glmnet, which nests inner-CV tuning of glmnet alpha and lambda
    inside outer-CV performance estimation
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the elastic-net regularised models whose lambda (and alpha) are cross-validated
    on inner folds
- name: caret
  role: Alternative framework used by nestcv.train for tuning a large number of prediction
    methods within the same nested-CV scheme
- name: R
  role: Language in which the nested-CV tuning workflow is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:696411307a7f9e7b422b493ab0775c06b0b2857c21babecdaebf16a7a639d04c
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
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# elastic-net-hyperparameter-tuning

## Summary

Tunes both elastic-net penalisation parameters (alpha and lambda) for glmnet models via inner-CV inside nested cross-validation, so hyperparameter selection never touches outer-fold test data and performance estimates remain unbiased.

## When to use

Use when fitting regularised glmnet models on high-dimensional data (P >> n, e.g. >16,000-gene RNA-seq with n=68) and both the mixing parameter alpha and shrinkage lambda must be cross-validated without biasing outer-CV accuracy.

## When NOT to use

- When the goal is a conventional 2/3-1/3 train/test split evaluation on a dataset large enough to support it rather than CV-based tuning
- When filtering predictors on the whole dataset before CV is intended — the article warns this creates bias when determining model accuracy
- When the model family is not supported by glmnet or caret (nestedcv's supported backends)

## Inputs

- Outcome vector y (binary or multinomial factor)
- Predictor matrix x (e.g. processed RNA-seq expression matrix; dataframes with factors can be one-hot encoded via one_hot())
- family argument for glmnet (e.g. 'multinomial', binomial)
- Optional feature-filter function and filter_options list

## Outputs

- Nested-CV fit object with per-outer-fold tuned alpha and lambda
- Unbiased outer-CV metrics (AUC, accuracy, balanced accuracy) from concatenated left-out test predictions
- Final model fitted on the whole dataset with coefficients (via coef())
- Tuning diagnostic plots (plot_alphas, plot_lambdas, per-fold cvafit plots)

## How to apply

Run nestcv.glmnet from the nestedcv R package, which partitions the data into default 10 x 10 stratified outer/inner folds (stratified on the outcome via caret's createFolds(), custom fold indices may be supplied). The inner 10-fold CV tunes the elastic-net alpha parameter and lambda, then the model is fitted on the whole inner training data and tested on the left-out outer fold; outer test predictions are concatenated for unbiased AUC, accuracy and balanced accuracy (e.g. AUC 0.7648, accuracy 0.7059, balanced accuracy 0.6773 for the R4RA rituximab E-MTAB-11611 subset with no filter). Alpha and lambda tuning per outer fold can be inspected with plot_alphas(res), plot_lambdas(res), and plot(res$outer_result[[1]]$cvafit). Optionally set filterFUN (e.g. ttest_filter with nfilter) so feature selection is nested within the outer loop, and parallelise the outer loop with cv.cores using detected physical cores. After outer CV, a final round of CV on the whole dataset determines hyperparameters for the final model; coef() extracts its coefficients.

## Related tools

- **nestedcv** (Provides nestcv.glmnet, which nests inner-CV tuning of glmnet alpha and lambda inside outer-CV performance estimation) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the elastic-net regularised models whose lambda (and alpha) are cross-validated on inner folds)
- **caret** (Alternative framework used by nestcv.train for tuning a large number of prediction methods within the same nested-CV scheme)
- **R** (Language in which the nested-CV tuning workflow is executed)

## Examples

```
res <- nestcv.glmnet(y, x, family = "multinomial", cv.cores = cores); summary(res)
```

## Evaluation signals

- Outer-CV metrics should approximate published benchmark values: R4RA rituximab subset (n=68, >16,000 genes) with no filter gives AUC 0.7648, accuracy 0.7059, balanced accuracy 0.6773
- Tuned alpha/lambda values should vary per outer fold and be recoverable from res$outer_result[[i]]$cvafit
- summary(res) should report both inner (tuning) and outer (test) CV information; ROC curves can be plotted for binary outcomes from left-out folds
- Default fold structure must be 10 x 10 stratified folds unless custom fold indices were supplied
- Outer-loop parallelisation (cv.cores) should not change concatenated test predictions

## Limitations

- Full nested CV is computationally heavier than a single CV pass; parallelise the outer loop with cv.cores (mclapply on unix/mac, parLapply on windows)
- If hyperparameter tuning or feature filtering is done on the whole dataset instead of within CV loops, accuracy estimates become biased (Vabalas et al, 2019)
- Accuracy determination on left-out test folds suffers when datasets are large enough for a 2/3-1/3 split; nested CV is aimed at limited-sample, P >> n settings

## Evidence

- [abstract] Inner CV is used to tune models and outer CV is used to determine model performance without bias: "Inner CV is used to tune models and outer CV is used to determine model performance without bias"
- [intro] nestedcv folds and alpha tuning: "`nestedcv` partitions the dataset into outer and inner folds (default 10 x 10
folds)."
- [readme] README example tuning lambda and alpha: "using iris dataset (multinomial, 3 classes), we fit a glmnet
model, tuning both lambda and alpha with 10 x 10-fold nested CV."
- [intro] Final model refit on whole data: "A final round of CV is performed on the whole dataset to determine
hyperparameters to fit the final model to the whole data"
