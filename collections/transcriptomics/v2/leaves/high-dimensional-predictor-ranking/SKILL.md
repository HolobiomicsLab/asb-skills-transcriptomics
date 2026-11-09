---
name: high-dimensional-predictor-ranking
description: Use when a numeric predictor matrix x has far more columns than rows
  (transcriptomics-style P >> n) and you must rank/select predictors by univariate
  tests (t-test, Wilcoxon, ANOVA, correlation) before model fitting inside nested
  cross-validation.
when_to_use_negative:
- When filtering on the whole dataset outside CV loops — this biases outer-fold accuracy
  because left-out hold-out rows are touched during feature selection
- When predictors are fewer than samples and full-model selection (e.g. glmnet alone
  without an embedded filter) suffices
- When the outcome is multinomial and a strictly binary two-sample t-test contract
  is required without switching to anova_filter or stat_filter
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2269
tools:
- name: nestedcv
  role: Provides ttest_filter and sibling filter functions and nests them within the
    outer CV loop via the filterFUN argument to avoid information leakage
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Downstream regularized model (lasso/elastic net) fitted on the filtered predictors
    within nestcv.glmnet
- name: caret
  role: Alternative model framework for nestcv.train receiving the filtered predictor
    set (e.g. ranger random forest)
- name: CORElearn
  role: Backend for relieff_filter, an alternative univariate/instance-based ranking
    filter
- name: ranger
  role: Backend for ranger_filter, random forest variable-importance ranking of predictors
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:d5d5e5de5d5c724c6ad893460720fd3b99e46927f25c48eaa0317c44244212d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CORElearn
  - R
  - caret
  - glmnet
  - nestedcv
  - ranger
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# high-dimensional-predictor-ranking

## Summary

Ranks high-dimensional predictors (P >> n) by univariate association with the outcome using nestedcv's filter functions such as ttest_filter, which returns the indices of predictors in x to retain, with contract options force_vars, nfilter, p_cutoff and rsq_cutoff.

## When to use

Use when a numeric predictor matrix x has far more columns than rows (transcriptomics-style P >> n) and you must rank/select predictors by univariate tests (t-test, Wilcoxon, ANOVA, correlation) before model fitting inside nested cross-validation.

## When NOT to use

- When filtering on the whole dataset outside CV loops — this biases outer-fold accuracy because left-out hold-out rows are touched during feature selection
- When predictors are fewer than samples and full-model selection (e.g. glmnet alone without an embedded filter) suffices
- When the outcome is multinomial and a strictly binary two-sample t-test contract is required without switching to anova_filter or stat_filter

## Inputs

- Outcome vector y (factor for classification; iris$Species in the worked example)
- Numeric predictor matrix x (e.g. as.matrix(iris[, -5]); transcriptomics expression matrix in P >> n settings)
- Filter options: nfilter, force_vars, p_cutoff, rsq_cutoff

## Outputs

- Vector of retained predictor indices into x (returned by ttest_filter)
- retained_features.csv (vector of retained predictor indices, per the task contract)
- Filtered training matrix used within each outer CV fold via filterFUN

## How to apply

Run an independent two-sample t-test on every predictor column of x against the outcome vector y and rank columns by ascending p-value; ttest_filter returns the indices of predictors in x to retain. Apply the contract options: always retain force_vars columns regardless of rank, cap retention at nfilter (mirroring the documented call ttest_filter(y, x, nfilter = 100)), drop columns whose p-value exceeds p_cutoff, and drop columns whose squared correlation with y falls below rsq_cutoff. Crucially, embed the filter within the outer CV loop: apply it independently to each outer fold's training rows only, never on the whole dataset, because filtering on the whole dataset creates a bias when determining model accuracy (Vabalas et al, 2019). Within nestedcv, pass the filter via the filterFUN argument of nestcv.glmnet or nestcv.train, with options passed as a list through filter_options. Bootstrapped versions of the univariate filters (e.g. boot_ttest) are available for improved ranking stability, and stat_filter handles mixed continuous and categorical data.

## Related tools

- **nestedcv** (Provides ttest_filter and sibling filter functions and nests them within the outer CV loop via the filterFUN argument to avoid information leakage) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Downstream regularized model (lasso/elastic net) fitted on the filtered predictors within nestcv.glmnet)
- **caret** (Alternative model framework for nestcv.train receiving the filtered predictor set (e.g. ranger random forest))
- **CORElearn** (Backend for relieff_filter, an alternative univariate/instance-based ranking filter)
- **ranger** (Backend for ranger_filter, random forest variable-importance ranking of predictors)

## Examples

```
library(nestedcv); filt <- ttest_filter(y, x, nfilter = 100); res <- nestcv.glmnet(y, x, family = "multinomial", filterFUN = ttest_filter, filter_options = list(nfilter = 100))
```

## Evaluation signals

- On iris, ttest_filter returns the expected number and identity of retained column indices using only training rows, never hold-out folds
- force_vars columns appear in the retained index vector regardless of their t-test rank
- Length of the retained index vector never exceeds nfilter (e.g. 100 in the documented call)
- No column with p-value above p_cutoff, or squared correlation with y below rsq_cutoff, appears in the retained indices
- Outer CV performance is unbiased: the filter is applied independently per outer fold, and hold-out rows are never touched during feature selection

## Limitations

- Univariate filters rank one predictor at a time and can be unstable; bootstrapped versions such as boot_ttest are provided for improved ranking stability
- Filtering on the whole dataset instead of within the outer CV loop creates bias when determining model accuracy
- Discussion-section documentation (changelog) is absent from the synthesized source, so version-specific filter behaviour is not verifiable here

## Evidence

- [abstract] Fast filter functions for feature selection are provided and the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets: "Fast filter functions for feature selection are provided and the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] ttest_filter documented call: [evidence span withheld — anchor status: not_found]
- [readme] Filter functions embedded in outer loop: "A number of feature selection filter functions (t-test, Wilcoxon test, ANOVA, Pearson/Spearman correlation, random forest, ReliefF) for feature selection are provided and can be embedded within the"
- [readme] Filter options passed through filter_options: "Filters can be used by setting the `filterFUN` argument. Options for the filter 
function are passed as a list through `filter_options`."
- [intro] Whole-dataset filtering bias warning: "it has been demonstrated that filtering on the whole dataset creates a bias when determining accuracy of models (Vabalas et al, 2019)"
- [intro] Bootstrapped filter stability: "Bootstrapped versions of the univariate filters are available [see `boot_ttest()`.]"
