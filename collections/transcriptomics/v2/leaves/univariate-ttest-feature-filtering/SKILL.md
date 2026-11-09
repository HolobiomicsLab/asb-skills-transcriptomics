---
name: univariate-ttest-feature-filtering
description: Use when building a glmnet/caret predictive model on high-dimensional
  biomedical data (P >> n) with a binary outcome and you must restrict predictors
  to the strongest univariate signals without leaking test-fold information.
when_to_use_negative:
- The outcome is multinomial/regression rather than a two-class comparison — use anova_filter,
  correl_filter, or stat_filter for mixed continuous/categorical data instead
- You need a multivariate or stability-aware filter — glmnet_filter, rf_filter/ranger_filter,
  relieff_filter, or bootstrapped boot_ttest() are the article's alternatives
- You intend to run the filter once on the full dataset before CV — this leaks information
  and biases outer-fold performance estimates
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: nestedcv
  role: Provides the ttest_filter function and the nestcv.glmnet/nestcv.train wrappers
    that embed filters within the outer CV loop to avoid information leakage from
    performance test sets
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the elastic net regularized model on the ttest-filtered predictor matrix
    inside each nested CV fold
- name: caret
  role: Alternative model framework (e.g. nestcv.train with ranger) whose outer loop
    can host the ttest_filter feature-selection step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:31a844f587b615cd5642ef0a1e8c778e02fa63c61abf92b598fa38f3bc382389
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# univariate-ttest-feature-filtering

## Summary

A univariate t-test feature filter (nestedcv's ttest_filter) that ranks predictor columns by ascending p-value against a binary outcome and retains only the top predictors, embedded inside the outer loop of nested cross-validation so that hold-out test rows never influence feature selection.

## When to use

Use when building a glmnet/caret predictive model on high-dimensional biomedical data (P >> n) with a binary outcome and you must restrict predictors to the strongest univariate signals without leaking test-fold information.

## When NOT to use

- The outcome is multinomial/regression rather than a two-class comparison — use anova_filter, correl_filter, or stat_filter for mixed continuous/categorical data instead
- You need a multivariate or stability-aware filter — glmnet_filter, rf_filter/ranger_filter, relieff_filter, or bootstrapped boot_ttest() are the article's alternatives
- You intend to run the filter once on the full dataset before CV — this leaks information and biases outer-fold performance estimates

## Inputs

- Binary outcome vector y (factor or character class labels)
- Numeric predictor matrix x (columns = features; may come from datasets like iris, Sonar, or transcriptomics matrices)
- Filter arguments: nfilter (max retained features), force_vars (columns always kept), p_cutoff, rsq_cutoff

## Outputs

- Integer vector of retained predictor column indices (writable to retained_features.csv)
- Filtered predictor matrix passed to the inner-fold model fit within each outer CV fold

## How to apply

Load the outcome vector y and numeric predictor matrix x, then run an independent two-sample t-test on every predictor column against y, ranking columns by ascending p-value. Apply the contract options: force_vars columns are always retained regardless of rank, retention is capped at nfilter (as in the documented call ttest_filter(y, x, nfilter = 100)), columns with p-value above p_cutoff are dropped, and columns whose squared correlation with y is below rsq_cutoff are dropped. The function returns the indices of predictors in x to retain. Critically, run the filter independently on each outer CV fold using only training rows, because filtering on the whole dataset creates bias when determining model accuracy (Vabalas et al, 2019). Inside nestcv.glmnet or nestcv.train, supply the filter via the filterFUN argument with options passed as a list through filter_options.

## Related tools

- **nestedcv** (Provides the ttest_filter function and the nestcv.glmnet/nestcv.train wrappers that embed filters within the outer CV loop to avoid information leakage from performance test sets) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the elastic net regularized model on the ttest-filtered predictor matrix inside each nested CV fold)
- **caret** (Alternative model framework (e.g. nestcv.train with ranger) whose outer loop can host the ttest_filter feature-selection step)

## Examples

```
filt <- ttest_filter(y, x, nfilter = 100)
```

## Evaluation signals

- On iris (reduced to a binary outcome) the filter returns the expected number and identity of retained column indices computed from training rows only
- The returned indices always include force_vars columns regardless of their p-value rank
- The number of retained columns never exceeds nfilter and no retained column has p-value above p_cutoff or squared correlation below rsq_cutoff
- Re-running the filter per outer fold yields fold-specific feature sets; hold-out rows are never touched during feature selection
- If filters were correctly nested, outer CV accuracy is unbiased — applying the filter on the whole dataset beforehand should give noticeably (biased) better apparent accuracy

## Limitations

- Filtering predictors on the whole dataset creates bias when determining model accuracy; the filter must sit inside the outer CV loop
- Univariate t-test ranking can be unstable across folds; bootstrapped versions (boot_ttest) exist for improved ranking stability
- Designed for two-class comparisons; other outcomes need anova_filter, correl_filter, or stat_filter

## Evidence

- [abstract] Fast filter functions for feature selection are provided, nested within the outer CV loop to avoid information leakage: "Fast filter functions for feature selection are provided and the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] ttest_filter documented call with nfilter=100: "filt <- ttest_filter(y, x, nfilter = 100)"
- [intro] Rationale for training-only filtering: "it has been demonstrated that filtering on the whole dataset creates a bias when determining accuracy of models (Vabalas et al, 2019)"
- [readme] filterFUN/filter_options wiring in README: "Filters can be used by setting the `filterFUN` argument. Options for the filter function are passed as a list through `filter_options`."
