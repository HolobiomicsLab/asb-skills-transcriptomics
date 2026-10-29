---
name: information-leakage-avoidance-in-cv
description: Use when building predictive models on high-dimensional data (P >> n)
  where univariate feature filters like ttest_filter are applied inside nested cross-validation
  and unbiased outer-fold accuracy is required.
when_to_use_negative:
- You only need a filter ranking for descriptive/exploratory statistics on the full
  dataset and no CV performance estimate — running ttest_filter on all data is fine
  outside CV, but never feed those rankings into CV accuracy measurement
- Predictor count is modest relative to sample size and no feature selection step
  is embedded in the CV loop — leakage avoidance machinery is unnecessary
- Your workflow already pre-specifies the feature set a priori (e.g. force_vars only)
  so no data-driven filtering occurs within folds
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3518
tools:
- name: nestedcv
  role: Provides ttest_filter and other filter functions and ensures they are nested
    within the outer CV loop to avoid information leakage
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the elastic net regularized model on filtered features inside nestcv.glmnet's
    nested CV loops
- name: caret
  role: Alternative model-fitting framework used via nestcv.train with embedded filters
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:42f6bd0f45b69368e1e67ed62c0b9fbe48a6c3dfee3ca507ba9784fc60fc79d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - caret
  - glmnet
  - nestedcv
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# information-leakage-avoidance-in-cv

## Summary

Applies univariate feature-selection filters (ttest_filter and related nestedcv filters) independently within each outer CV fold so hold-out test rows never influence feature selection, avoiding the accuracy bias from filtering on the whole dataset.

## When to use

Use when building predictive models on high-dimensional data (P >> n) where univariate feature filters like ttest_filter are applied inside nested cross-validation and unbiased outer-fold accuracy is required.

## When NOT to use

- You only need a filter ranking for descriptive/exploratory statistics on the full dataset and no CV performance estimate — running ttest_filter on all data is fine outside CV, but never feed those rankings into CV accuracy measurement
- Predictor count is modest relative to sample size and no feature selection step is embedded in the CV loop — leakage avoidance machinery is unnecessary
- Your workflow already pre-specifies the feature set a priori (e.g. force_vars only) so no data-driven filtering occurs within folds

## Inputs

- Binary or multinomial outcome vector y (e.g. iris$Species)
- Numeric predictor matrix x (e.g. as.matrix(iris[, -5])), optionally P >> n transcriptomic data
- Filter parameters: nfilter (retention cap), force_vars (always-retained columns), p_cutoff, rsq_cutoff

## Outputs

- Vector of retained predictor indices from ttest_filter, computed per outer CV fold on training rows only
- Unbiased outer CV performance estimates from nestcv.glmnet/nestcv.train with embedded filtering
- Final model fitted on the whole dataset after a final round of CV for prediction on external data

## How to apply

Implement the filter as a function returning the indices of predictors in x to retain: run an independent two-sample t-test on every predictor column against y, rank columns by ascending p-value, always retain force_vars columns regardless of rank, cap retention at nfilter (e.g. ttest_filter(y, x, nfilter = 100)), drop columns with p-value above p_cutoff and columns whose squared correlation with y is below rsq_cutoff. Critically, pass the filter through nestcv.glmnet's filterFUN argument (options via filter_options) so it is applied only to the training rows of each outer fold, never to the whole dataset — filtering on the whole dataset creates bias when determining model accuracy (Vabalas et al, 2019). The nestedcv package ensures filters are nested within the outer CV loop: inner 10-fold CV tunes hyperparameters, the model is fitted on inner-fold training data with filtered features, and tested on the left-out outer fold. For mixed data types, swap in wilcoxon_filter, anova_filter, correl_filter, rf_filter/ranger_filter, relieff_filter, glmnet_filter, stat_filter, or bootstrapped versions like boot_ttest for ranking stability.

## Related tools

- **nestedcv** (Provides ttest_filter and other filter functions and ensures they are nested within the outer CV loop to avoid information leakage) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the elastic net regularized model on filtered features inside nestcv.glmnet's nested CV loops)
- **caret** (Alternative model-fitting framework used via nestcv.train with embedded filters)

## Examples

```
filt <- ttest_filter(y, x, nfilter = 100); res <- nestcv.glmnet(y, x, filterFUN = ttest_filter, filter_options = list(nfilter = 100), cv.cores = cores)
```

## Evaluation signals

- ttest_filter returns only column indices computed from training rows of the fold — verify hold-out rows were never touched during feature selection
- Forced columns (force_vars) appear in the retained set regardless of t-test rank
- Retained set size never exceeds nfilter; columns with p > p_cutoff or rsq < rsq_cutoff are absent
- Outer CV accuracy from summary(res) should be comparable across folds and not inflated relative to a known-baseline whole-data-filtered run
- Final model coefficients via coef(res) derive from a final CV round on the whole dataset, distinct from outer-fold evaluation

## Limitations

- Bootstrapped filter variants (boot_ttest) exist because univariate rankings can be unstable across folds
- Default outer/inner partition is 10 x 10 folds; very small samples may need custom fold indices (stratified via caret's createFolds by default)
- Filter options for mixed continuous/categorical data differ — use stat_filter rather than ttest_filter
- Discussion section of the source document is absent, so some caveats may be undocumented

## Evidence

- [abstract] the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets: "Fast filter functions for feature selection are provided and the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] it has been demonstrated that filtering on the whole dataset creates a bias when determining accuracy of models (Vabalas et al, 2019): "it has been
demonstrated that filtering on the whole dataset creates a bias when determining
accuracy of models (Vabalas et al, 2019)"
- [intro] filt <- ttest_filter(y, x, nfilter = 100): [evidence span withheld — anchor status: not_found]
- [readme] Filters can be used by setting the filterFUN argument. Options for the filter function are passed as a list through filter_options.: "Filters can be used by setting the `filterFUN` argument. Options for the filter 
function are passed as a list through `filter_options`."
- [intro] The inner fold CV, (default is 10-fold), is used to tune optimal hyperparameters for models. Then the model is fitted on the whole inner fold and tested on the left-out data from the outer fold.: "The inner fold CV, (default is 10-fold), is used to tune optimal
hyperparameters for models. Then the model is fitted on the whole inner fold and
tested on the left-out data from the outer fold."
