---
name: t-test-predictor-filtering
description: Use when a binary-outcome dataset has many predictors (P >> n) and you
  must pre-select features before lasso/elastic-net fitting while keeping performance
  estimates unbiased under nested CV.
when_to_use_negative:
- The predictor set is already small (P not >> n) or a hold-out split is large enough
  that whole-dataset filtering bias is immaterial — univariate pre-filtering adds
  risk without benefit
- 'The outcome is continuous or multi-class: the t-test filter targets binary group
  comparison; use correl_filter (Pearson/Spearman) for regression or anova_filter
  for multi-class instead'
- The goal is unbiased tuning of a non-regularized model with no feature pre-selection
  step — the filter has no role in that design
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: nestedcv
  role: Provides ttest_filter and ensures the filter is nested within the outer CV
    loop to avoid information leakage from performance test sets
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the lasso/elastic-net model on the t-test-filtered predictor matrix within
    each inner training fold
  repo: https://CRAN.R-project.org/package=nestedcv
- name: pROC
  role: Computes ROC AUC on concatenated outer test-fold predictions to check for
    filtering-induced bias
- name: R
  role: Implementation language for the nestedcv filter functions and the leakage
    simulation
  repo: https://CRAN.R-project.org/package=nestedcv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:3e9697cde61d93328c1839864be5eabc0e9db39c99353cbe6fc24fa0784b061a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  - pROC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# t-test-predictor-filtering

## Summary

Univariate two-sample t-test ranking of predictors (nestedcv's ttest_filter) to reduce a high-dimensional predictor matrix (P >> n) to a small top-k feature subset before regularized regression, embedded inside the outer loop of nested cross-validation so the filter never sees outer test-fold data.

## When to use

Use when a binary-outcome dataset has many predictors (P >> n) and you must pre-select features before lasso/elastic-net fitting while keeping performance estimates unbiased under nested CV.

## When NOT to use

- The predictor set is already small (P not >> n) or a hold-out split is large enough that whole-dataset filtering bias is immaterial — univariate pre-filtering adds risk without benefit
- The outcome is continuous or multi-class: the t-test filter targets binary group comparison; use correl_filter (Pearson/Spearman) for regression or anova_filter for multi-class instead
- The goal is unbiased tuning of a non-regularized model with no feature pre-selection step — the filter has no role in that design

## Inputs

- Numeric predictor matrix x with predictors far outnumbering samples (P >> n), e.g. transcriptomics RNA-Seq expression matrix
- Balanced binary outcome vector y (factor or 0/1)
- Optional filter_options list, e.g. nfilter = 100 top-ranked predictors

## Outputs

- Filtered predictor matrix restricted to the top nfilter t-test-ranked variables
- Nested-CV performance estimate (e.g. concatenated outer-fold ROC AUC via pROC), unbiased on noise data (~0.50)
- Fitted lasso/elastic-net model on the filtered feature set

## How to apply

Call ttest_filter(y, x, nfilter = 100) on training data only: it ranks each predictor by a two-sample t-test between the binary outcome classes and retains the top nfilter variables. Embed it in nested CV by passing filterFUN = ttest_filter to nestcv.glmnet (10 x 10 folds by default), so the filter is recomputed within each outer training fold — the paper stresses that 'filtering on the whole dataset creates a bias when determining accuracy of models' and that nestedcv ensures filters sit within the outer loop to avoid information leakage. Bootstrapped ranking via boot_ttest() is available if univariate ranking is unstable. The filtered matrix is then passed to glmnet lasso fitting, with outer-fold test predictions concatenated for ROC AUC evaluation via pROC; on pure gaussian noise data this correctly reports AUC ≈ 0.50, whereas whole-dataset filtering inflates hold-out AUC (Fig. 2A, red line).

## Related tools

- **nestedcv** (Provides ttest_filter and ensures the filter is nested within the outer CV loop to avoid information leakage from performance test sets) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the lasso/elastic-net model on the t-test-filtered predictor matrix within each inner training fold) — https://CRAN.R-project.org/package=nestedcv
- **pROC** (Computes ROC AUC on concatenated outer test-fold predictions to check for filtering-induced bias)
- **R** (Implementation language for the nestedcv filter functions and the leakage simulation) — https://CRAN.R-project.org/package=nestedcv

## Examples

```
res <- nestcv.glmnet(y, x, family = "binomial", filterFUN = ttest_filter, filter_options = list(nfilter = 100), cv.cores = cores)
```

## Evaluation signals

- On pure gaussian-noise predictors with random balanced outcome and P >> n, fully nested CV with ttest_filter inside the outer loop reports ROC AUC close to 0.50; a substantially higher value indicates leakage
- Filter-call audit: ttest_filter must be invoked once per outer (and inner) training fold, never once on the whole dataset, and the outer test fold must not influence variable ranking
- Filtered matrix dimensionality matches nfilter (article example uses nfilter = 100) and the retained variables differ across outer folds, confirming fold-specific selection
- Comparison table (noise_simulation_auc.csv) shows Scheme A (whole-dataset filtering, hold-out test) AUC >> Scheme C (fully nested CV) AUC, reproducing the Fig. 2A red vs nested pattern
- summary() of the nestcv.glmnet result reports per-outer-fold tuned lambda/alpha, confirming the filter ran upstream of hyperparameter tuning in each fold

## Limitations

- Whole-dataset application of the filter inflates apparent model accuracy through information leakage — the exact failure mode the nested design prevents
- Bootstrapped univariate filters (e.g. boot_ttest) exist because single-split univariate rankings can be unstable; plain ttest_filter rankings may vary fold to fold
- The t-test filter targets binary comparisons; continuous or multi-class outcomes need correl_filter or anova_filter
- Biomedical datasets with P >> n are often too small for a conventional 2/3-1/3 split, so a single-filter + single-split design both leaks and wastes samples

## Evidence

- [abstract] Fast filter functions for feature selection are provided and the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets: "Fast filter functions for feature selection are provided and the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] A number of feature selection filter functions (t-test, Wilcoxon test, ANOVA, Pearson/Spearman correlation, random forest, ReliefF) for feature selection are provided: "A number of feature
selection filter functions (t-test, Wilcoxon test, ANOVA, Pearson/Spearman
correlation, random forest, ReliefF) for feature selection are provided"
- [intro] filt <- ttest_filter(y, x, nfilter = 100): "filt <- ttest_filter(y, x, nfilter = 100)"
- [intro] it has been demonstrated that filtering on the whole dataset creates a bias when determining accuracy of models (Vabalas et al, 2019): "it has been
demonstrated that filtering on the whole dataset creates a bias when determining
accuracy of models (Vabalas et al, 2019)"
- [readme] Filters can be used by setting the filterFUN argument. Options for the filter function are passed as a list through filter_options.: "Filters can be used by setting the `filterFUN` argument. Options for the filter 
function are passed as a list through `filter_options`."
