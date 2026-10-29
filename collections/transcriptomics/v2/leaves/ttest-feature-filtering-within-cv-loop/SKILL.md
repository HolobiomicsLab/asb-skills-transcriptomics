---
name: ttest-feature-filtering-within-cv-loop
description: Use when high-dimensional transcriptomic data (P >> n, e.g. RNA-seq expression
  matrices) must be filtered to a small top-variable set before fitting glmnet/elastic-net
  models, and you need an unbiased performance estimate rather than the optimistically
  biased inner-CV metric.
when_to_use_negative:
- When the predictor set is already a small curated feature table — univariate t-test
  filtering (nfilter = 300) is meant for P >> n high-dimensional data
- When you intend to filter features once on the full dataset before CV — this creates
  the performance-estimation bias the skill exists to avoid
- When the outcome is not a two-class response compatible with a t-test filter (e.g.
  multinomial or regression targets need other filters like anova_filter or correl_filter)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: nestedcv
  role: Provides nestcv.glmnet and ttest_filter to embed univariate t-test feature
    selection within the outer CV loop and report outer vs inner AUC
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the elastic-net regularized logistic model whose lambda/alpha are tuned
    on inner folds after the filter is applied
- name: pROC
  role: Used to compute the ROC AUC for left-out outer and inner CV folds
- name: R
  role: Runtime environment for the nestedcv workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:935941299ac7469a0dc9566478c8b756287ef02ac189766e460308004c95fb32
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  - pROC
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ttest-feature-filtering-within-cv-loop

## Summary

Applies the nestedcv R package's ttest_filter (univariate t-test ranking, e.g. nfilter = 300) strictly inside the outer loop of nested cross-validation so feature selection is re-run on each outer training fold and the outer-fold AUC is an unbiased estimate, unlike filtering on the whole dataset which inflates performance.

## When to use

Use when high-dimensional transcriptomic data (P >> n, e.g. RNA-seq expression matrices) must be filtered to a small top-variable set before fitting glmnet/elastic-net models, and you need an unbiased performance estimate rather than the optimistically biased inner-CV metric.

## When NOT to use

- When the predictor set is already a small curated feature table — univariate t-test filtering (nfilter = 300) is meant for P >> n high-dimensional data
- When you intend to filter features once on the full dataset before CV — this creates the performance-estimation bias the skill exists to avoid
- When the outcome is not a two-class response compatible with a t-test filter (e.g. multinomial or regression targets need other filters like anova_filter or correl_filter)

## Inputs

- High-dimensional RNA-seq expression matrix (predictors as matrix; P >> n, e.g. E-MTAB-11611 R4RA rituximab cohort from ArrayExpress)
- Binary outcome vector (e.g. rituximab response status) matching the sample rows
- Filter configuration: filterFUN = ttest_filter with filter_options such as nfilter = 300
- Fold specification (default 10x10 stratified outer/inner folds via caret's createFolds(); custom fold indices optional)

## Outputs

- Fitted nestcv.glmnet object with outer_result, outer-fold summary metrics and innercv_roc AUC
- Outer-fold (unbiased) ROC AUC, e.g. 0.783 on the R4RA rituximab cohort
- Inner-CV (optimistically biased) ROC AUC, e.g. 0.863 on the same cohort
- Comparison table/CSV (e.g. rituximab_nestedcv_auc.csv) recording both AUCs and the outer-vs-inner verdict

## How to apply

Load the expression matrix and binary response (e.g. the R4RA rituximab cohort, E-MTAB-11611 from ArrayExpress) and call nestcv.glmnet with default 10x10 stratified outer/inner folds, passing filterFUN = ttest_filter and filter_options = list(nfilter = 300) so the t-test filter is nested within the outer CV loop. Inner folds tune glmnet lambda (and alpha) hyperparameters; the model fitted on the inner training data is tested on the left-out outer fold, and outer test-fold predictions are concatenated for the final accuracy measure. Critically, the filter must run per outer fold on training data only — the article notes that 'filtering on the whole dataset creates a bias when determining accuracy'. Extract both the outer-fold ROC AUC and the innercv_roc AUC from the summary of the fitted object; the outer estimate should be lower (unbiased) than the inner estimate (e.g. 0.783 vs 0.863 on the R4RA rituximab cohort). Write both metrics and the comparison verdict to a results file such as rituximab_nestedcv_auc.csv.

## Related tools

- **nestedcv** (Provides nestcv.glmnet and ttest_filter to embed univariate t-test feature selection within the outer CV loop and report outer vs inner AUC) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the elastic-net regularized logistic model whose lambda/alpha are tuned on inner folds after the filter is applied)
- **pROC** (Used to compute the ROC AUC for left-out outer and inner CV folds)
- **R** (Runtime environment for the nestedcv workflow)

## Examples

```
res <- nestcv.glmnet(y, x, filterFUN = ttest_filter, filter_options = list(nfilter = 300), cv.cores = cores); summary(res)
```

## Evaluation signals

- Outer-fold AUC must be lower than the left-out inner-CV AUC (unbiased vs biased), matching the R4RA benchmark of 0.783 (outer) vs 0.863 (inner)
- Confirm filterFUN is evaluated per outer fold: the selected nfilter = 300 features may differ between outer folds, with no leakage from outer test sets
- summary() of the nestcv.glmnet object must expose both outer-fold summary metrics and innercv_roc AUC
- Fold setup should be the default 10x10 stratified folds on the outcome variable (caret createFolds()); verify stratification on the rituximab response
- Output CSV must contain both AUC values and an explicit outer-vs-inner comparison verdict

## Limitations

- Filtering predictors on the whole dataset creates bias when determining model accuracy; the filter must only see training data within CV loops
- Biomedical datasets may be too small for a conventional 2/3-1/3 split, which is why nested CV is used rather than a single held-out test set
- The GitHub discussion section lacks a changelog and full metadata, so version pinning of nestedcv cannot be verified from the synthesized sources

## Evidence

- [abstract] the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets: "Fast filter functions for feature selection are provided and the package ensures that filters are nested within the outer CV loop to avoid information leakage"
- [intro] it has been demonstrated that filtering on the whole dataset creates a bias when determining accuracy of models: "it has been
demonstrated that filtering on the whole dataset creates a bias when determining
accuracy of models (Vabalas et al, 2019)"
- [results] Nested CV with a univariate filter and glmnet on the rituximab cohort reaches an outer-fold AUC of 0.783: [evidence span withheld — anchor status: not_found]
- [readme] Filters can be used by setting the filterFUN argument. Options for the filter function are passed as a list through filter_options.: "Filters can be used by setting the `filterFUN` argument. Options for the filter 
function are passed as a list through `filter_options`."
- [intro] nestedcv partitions the dataset into outer and inner folds (default 10 x 10 folds): "`nestedcv` partitions the dataset into outer and inner folds (default 10 x 10
folds)."
