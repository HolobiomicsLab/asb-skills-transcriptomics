---
name: r-function-verification-on-labelled-data
description: Use when implementing or testing a univariate feature-selection filter
  function in R (e.g. t-test ranked predictors with forced variables, count caps and
  correlation cutoffs) against labelled data before embedding it in nested CV.
when_to_use_negative:
- Filtering predictors on the whole dataset before cross-validation — this creates
  bias when determining model accuracy (Vabalas et al, 2019)
- Mixed continuous/categorical predictor data, where the stat_filter univariate filter
  or bootstrapped variants like boot_ttest are the intended choice instead
- Regression outcomes needing correlation-based screening, where correl_filter (Pearson/Spearman)
  is the appropriate filter
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: nestedcv
  role: Source of the ttest_filter function and the nested-CV framework that applies
    it per outer fold to avoid information leakage
  repo: https://CRAN.R-project.org/package=nestedcv
- name: R
  role: Language in which the t-test ranking and filter options are implemented and
    verified
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:9ce95ce275b74c7bab9d6fadef12c5d003796ace90b320675dab71f8698faa55
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - nestedcv
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# r-function-verification-on-labelled-data

## Summary

Reconstructs and verifies the nestedcv ttest_filter contract — univariate two-sample t-tests ranking predictors, with force_vars, nfilter, p_cutoff and rsq_cutoff options — on labelled data such as iris, using only training rows so hold-out folds remain untouched.

## When to use

Use when implementing or testing a univariate feature-selection filter function in R (e.g. t-test ranked predictors with forced variables, count caps and correlation cutoffs) against labelled data before embedding it in nested CV.

## When NOT to use

- Filtering predictors on the whole dataset before cross-validation — this creates bias when determining model accuracy (Vabalas et al, 2019)
- Mixed continuous/categorical predictor data, where the stat_filter univariate filter or bootstrapped variants like boot_ttest are the intended choice instead
- Regression outcomes needing correlation-based screening, where correl_filter (Pearson/Spearman) is the appropriate filter

## Inputs

- Binary or multinomial outcome vector y (e.g. iris$Species)
- Numeric predictor matrix x (e.g. as.matrix(iris[, -5]))
- Filter contract parameters: nfilter, force_vars, p_cutoff, rsq_cutoff
- Training-fold row indices (never hold-out fold rows)

## Outputs

- Vector of retained predictor column indices (e.g. written to retained_features.csv)
- Ranked t-test p-value table per predictor column

## How to apply

Split the labelled dataset (e.g. iris) into an outcome vector y and a numeric predictor matrix x, then run an independent two-sample t-test on every predictor column against y, ranking columns by ascending p-value to recreate the ttest_filter contract from the nestedcv package. Apply the documented options: always retain force_vars columns regardless of rank, cap retention at nfilter (mirroring the documented call 'ttest_filter(y, x, nfilter = 100)'), drop columns with p-value above p_cutoff, and drop columns whose squared correlation with y falls below rsq_cutoff. Critically, apply the filter only to training rows within each outer CV fold — the nestedcv package ensures filters are nested within the outer CV loop precisely so hold-out test data never influences feature selection. In nestedcv itself, filters are wired in via the filterFUN argument with options passed as a list through filter_options, so verification of the standalone function establishes the contract before it is embedded.

## Related tools

- **nestedcv** (Source of the ttest_filter function and the nested-CV framework that applies it per outer fold to avoid information leakage) — https://CRAN.R-project.org/package=nestedcv
- **R** (Language in which the t-test ranking and filter options are implemented and verified)

## Examples

```
library(nestedcv); data(iris); y <- iris$Species; x <- as.matrix(iris[, -5]); filt <- ttest_filter(y, x, nfilter = 100)
```

## Evaluation signals

- On iris, the function returns the expected number and identity of retained column indices given nfilter, p_cutoff and rsq_cutoff settings
- force_vars columns appear in the output indices regardless of their t-test rank
- No retained index depends on hold-out fold rows: re-running the filter on training rows of each outer fold yields fold-specific indices
- Columns with p-value above p_cutoff and squared correlation below rsq_cutoff are absent from the retained set
- Embedded in nestcv.glmnet, the filter runs per outer fold via filterFUN/filter_options rather than once on the full matrix

## Limitations

- Bootstrapped versions of the univariate filters (e.g. boot_ttest) exist because plain univariate ranking can be unstable
- Filtering on the whole dataset before CV biases model accuracy estimates, so the filter must never see hold-out rows
- ttest_filter suits binary comparisons; multi-class or mixed data need anova_filter or stat_filter instead

## Evidence

- [abstract] Filters nested within outer CV loop: "the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] Whole-dataset filtering bias: "it has been demonstrated that filtering on the whole dataset creates a bias when determining accuracy of models (Vabalas et al, 2019)"
- [intro] Documented ttest_filter call: "filt <- ttest_filter(y, x, nfilter = 100)"
- [readme] How filters are wired into nestcv.glmnet: "Filters can be used by setting the `filterFUN` argument. Options for the filter function are passed as a list through `filter_options`."
