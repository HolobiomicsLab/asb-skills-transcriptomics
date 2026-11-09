---
name: nested-cross-validation-model-fitting
description: Use when fitting lasso/elastic-net or caret-based models on high-dimensional
  data (e.g. RNA-seq with far more predictors than samples) and you need unbiased
  performance estimates plus feature selection without information leakage.
when_to_use_negative:
- Dataset is large enough for a conventional 2/3–1/3 train/test split — nested CV's
  main rationale is biomedical data too small for a fixed split, where the left-out
  test set is too small for reliable accuracy determination
- Predictors have already been filtered on the whole dataset before cross-validation
  — this reintroduces the selection bias the skill is designed to avoid; filters must
  be nested within the outer CV loop
- The goal is unsupervised analysis or simple univariate testing rather than supervised
  predictive model fitting and performance estimation
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: nestedcv
  role: Core package providing nestcv.glmnet and nestcv.train for nested k × l-fold
    CV with embedded filter functions and final whole-dataset model refit
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits lasso/elastic-net regularized models inside each fold, with nested cross-validation
    of both lambda and the alpha parameter
- name: caret
  role: Backend framework through which nestcv.train tunes a large array of other
    machine learning methods (e.g. random forest via ranger)
- name: randomForest
  role: Provides random forest variable importance for the rf_filter feature-selection
    filter and random forest models via nestcv.train
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:e9ae3e170b5b5aa6e6d14244de79b805f46c99061d28f2c81540a878886589de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - caret
  - glmnet
  - nestedcv
  - pROC
  - randomForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# nested-cross-validation-model-fitting

## Summary

Fit predictive models with fully nested k × l-fold cross-validation using the nestedcv R package, tuning hyperparameters (e.g. glmnet lambda and alpha) on inner folds and estimating unbiased performance on left-out outer folds, with feature-selection filters nested inside the outer loop. Designed for high-dimensional biomedical data (P ≫ n) where a single train/test split is unreliable.

## When to use

Use when fitting lasso/elastic-net or caret-based models on high-dimensional data (e.g. RNA-seq with far more predictors than samples) and you need unbiased performance estimates plus feature selection without information leakage.

## When NOT to use

- Dataset is large enough for a conventional 2/3–1/3 train/test split — nested CV's main rationale is biomedical data too small for a fixed split, where the left-out test set is too small for reliable accuracy determination
- Predictors have already been filtered on the whole dataset before cross-validation — this reintroduces the selection bias the skill is designed to avoid; filters must be nested within the outer CV loop
- The goal is unsupervised analysis or simple univariate testing rather than supervised predictive model fitting and performance estimation

## Inputs

- Outcome vector y (binary class labels, multinomial classes, or continuous response)
- Predictor matrix or data.frame x (e.g. RNA-seq expression matrix; factor/character columns one-hot encoded via one_hot())
- Dataset such as the R4RA rheumatoid arthritis RNA-Seq cohort (E-MTAB-11611), iris, Sonar, or BostonHousing2
- Optional filter function (filterFUN) with filter_options list (e.g. ttest_filter, nfilter = 300) and custom outer/inner fold indices

## Outputs

- nestcv.glmnet / nestcv.train result object with summary metrics including outer-fold ROC AUC and innercv_roc AUC
- Concatenated left-out outer-fold predictions vs true labels for unbiased accuracy measures
- Final model refitted on the whole dataset with ranked nonzero coefficients (e.g. 21 genes plus intercept for the rituximab glmnet model)
- Diagnostic plots: outer/inner ROC curves, plot_alphas(res) and plot_lambdas(res) deviance tuning plots, per-fold glmnet cvafit plots

## How to apply

Partition the outcome vector y and predictor matrix x into outer and inner stratified folds (default 10 × 10 folds, stratified on the outcome via caret's createFolds, or supply custom fold indices). For each outer fold, use the inner 10-fold CV to tune hyperparameters — for glmnet this includes cross-validation of the elastic net alpha parameter as well as lambda — then fit the model on the whole inner fold and predict the left-out outer fold; concatenate outer-fold predictions against true labels for accuracy measures such as ROC AUC. Embed univariate or model-based filters (ttest_filter with e.g. nfilter = 300, wilcoxon_filter, anova_filter, correl_filter, rf_filter, relieff_filter, glmnet_filter) via the filterFUN argument so filtering happens only within the outer training data, avoiding the whole-dataset filtering bias described by Vabalas et al. After outer CV completes, a final round of CV on the whole dataset determines hyperparameters and refits the final model, whose nonzero coefficients are exposed via coef(). Report the outer-fold AUC as the unbiased performance estimate and expect it to be lower than the optimistically biased inner-CV AUC (e.g. 0.783 vs 0.863 on the R4RA rituximab cohort).

## Related tools

- **nestedcv** (Core package providing nestcv.glmnet and nestcv.train for nested k × l-fold CV with embedded filter functions and final whole-dataset model refit) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits lasso/elastic-net regularized models inside each fold, with nested cross-validation of both lambda and the alpha parameter)
- **caret** (Backend framework through which nestcv.train tunes a large array of other machine learning methods (e.g. random forest via ranger))
- **randomForest** (Provides random forest variable importance for the rf_filter feature-selection filter and random forest models via nestcv.train)

## Examples

```
res <- nestcv.glmnet(y, x, family = "multinomial", cv.cores = cores); summary(res); coef(res)
```

## Evaluation signals

- Outer-fold (unbiased) AUC must be lower than the left-out inner-CV AUC; on the R4RA rituximab cohort with ttest_filter (nfilter = 300) and glmnet, expect outer AUC ≈ 0.783 vs inflated inner AUC ≈ 0.863
- summary(res) reports both outer and inner CV metrics; coef(res) on the final whole-dataset fit yields the ranked nonzero coefficients
- Final rituximab glmnet model (no filter, AUC 0.7648) should retain 21 genes plus intercept ranked AC016582.3, PCBP3, TMEM170B, EIF4E3, SEC14L6, CEP85, APLF down to DIRAS3, matching the published vignette output
- Verify fold construction: default 10 × 10 folds stratified on the outcome (caret createFolds), with filters applied inside each outer-fold training set only
- If parallelised with cv.cores, speedup must be positive (>1x) and plateau near the number of physical cores

## Limitations

- Filtering predictors on the whole dataset before CV creates bias when determining model accuracy; feature selection must be performed only on training data within the CV loops
- Outer-CV parallelisation speedup is model-dependent — up to 5.0x for nestcv.train random forest but only up to 2.5x for nestcv.glmnet, and it plateaus near the physical core count
- ROC curves and AUC from left-out folds are only available for binary comparisons (multiclass fits like the iris multinomial example do not produce ROC curves)

## Evidence

- [abstract] The nestedcv R package implements fully nested k × l-fold CV for lasso and elastic-net regularized linear models via the glmnet package: "The nestedcv R package implements fully nested k × l-fold CV for lasso and elastic-net regularized linear models via the glmnet package"
- [abstract] Filters nested in outer loop: "the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] Small test sets in conventional splits: "medical datasets may be too small for this, and so determination of accuracy in the left-out test set suffers because the test set is small."
- [intro] Default folds and stratification: "`nestedcv` partitions the dataset into outer and inner folds (default 10 x 10
folds)."
- [readme] README: alpha tuning and caret support: "Nested cross-validation (CV) for the glmnet and caret packages. With glmnet this
includes cross-validation of elastic net alpha parameter."
- [readme] README: filterFUN usage: "Filters can be used by setting the `filterFUN` argument. Options for the filter 
function are passed as a list through `filter_options`."
