---
name: cv-optimism-bias-quantification
description: Use when you have fitted a model with embedded univariate feature filtering
  (e.g. t-test filter, glmnet) on transcriptomic data and need to demonstrate that
  the inner-CV AUC is inflated relative to the unbiased outer-fold AUC.
when_to_use_negative:
- Filtering has already been performed on the whole dataset before CV — the article
  states this creates bias when determining model accuracy, so the outer/inner AUC
  contrast is no longer interpretable
- The dataset is large enough for a conventional 2/3-1/3 hold-out split with a large
  test set; nested CV is motivated by small biomedical samples where P ≫ n
- The outcome is multinomial/regression rather than binary, since the AUC-based outer-vs-inner
  comparison is defined for binary ROC analysis
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: nestedcv
  role: R package providing nestcv.glmnet for fully nested k × l-fold CV with filters
    nested inside the outer loop; exposes outer-fold and inner-CV AUC metrics
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the elastic-net regularized logistic model whose lambda (and alpha) are
    tuned on inner folds within each outer fold
- name: pROC
  role: ROC/AUC computation for the concatenated outer-fold test predictions and inner-CV
    fold predictions
- name: R
  role: Runtime environment for the nested CV workflow and writing the AUC comparison
    CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:4439125c185ee6b7b94eac38c954e5e73aac08def75ec9020468ebbeb5f884c4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  - pROC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cv-optimism-bias-quantification

## Summary

Quantifies the optimism bias of inner-CV performance estimates in high-dimensional (P ≫ n) transcriptomic modelling by comparing the unbiased outer-fold ROC AUC from nested CV against the left-out inner-CV AUC, e.g. 0.783 vs 0.863 in the R4RA rituximab glmnet example.

## When to use

Use when you have fitted a model with embedded univariate feature filtering (e.g. t-test filter, glmnet) on transcriptomic data and need to demonstrate that the inner-CV AUC is inflated relative to the unbiased outer-fold AUC.

## When NOT to use

- Filtering has already been performed on the whole dataset before CV — the article states this creates bias when determining model accuracy, so the outer/inner AUC contrast is no longer interpretable
- The dataset is large enough for a conventional 2/3-1/3 hold-out split with a large test set; nested CV is motivated by small biomedical samples where P ≫ n
- The outcome is multinomial/regression rather than binary, since the AUC-based outer-vs-inner comparison is defined for binary ROC analysis

## Inputs

- RNA-seq expression matrix (e.g. R4RA rituximab cohort, E-MTAB-11611 from ArrayExpress)
- Binary response vector of drug (rituximab) response
- Filter specification (filterFUN = ttest_filter, nfilter = 300) and CV fold settings (10x10 stratified)

## Outputs

- Outer-fold (unbiased) ROC AUC from the nestcv.glmnet summary metrics
- Inner-CV (left-out inner fold) ROC AUC (innercv_roc)
- Comparison verdict (outer AUC lower than inner AUC) written to e.g. rituximab_nestedcv_auc.csv

## How to apply

Fit a fully nested k × l-fold CV (default 10 × 10 stratified folds via caret's createFolds) using nestcv.glmnet from the nestedcv R package on an expression matrix with a binary response, setting filterFUN = ttest_filter with nfilter = 300 so the filter is nested within the outer loop and cannot leak into test sets. Extract the outer-fold ROC AUC (computed on concatenated left-out outer-fold predictions) and the innercv_roc AUC from the same fitted object. Report both side by side: the outer estimate is the unbiased model-accuracy measure, while the inner estimate is optimistically biased because hyperparameter tuning and filtering were selected on it. Write both AUC values and the comparison verdict (outer < inner) to an output CSV. If applying to new data, the direction of the gap should hold; a gap near zero or inverted suggests leakage or mis-specified folds.

## Related tools

- **nestedcv** (R package providing nestcv.glmnet for fully nested k × l-fold CV with filters nested inside the outer loop; exposes outer-fold and inner-CV AUC metrics) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the elastic-net regularized logistic model whose lambda (and alpha) are tuned on inner folds within each outer fold)
- **pROC** (ROC/AUC computation for the concatenated outer-fold test predictions and inner-CV fold predictions)
- **R** (Runtime environment for the nested CV workflow and writing the AUC comparison CSV)

## Examples

```
res <- nestcv.glmnet(y, x, family = "binomial", filterFUN = ttest_filter, filter_options = list(nfilter = 300), cv.cores = cores); summary(res)
```

## Evaluation signals

- Outer-fold AUC must be lower than the inner-CV AUC (R4RA reference: 0.783 outer vs 0.863 inner, a ~0.08 optimism gap)
- Filter must run inside the outer loop: filterFUN = ttest_filter with nfilter = 300, not applied to the full dataset beforehand
- Folds should be 10 × 10 and stratified on the outcome by default (caret createFolds)
- Both AUC values present in the fitted object (outer summary metrics and innercv_roc) and written to the output CSV with the comparison verdict
- Outer test-fold predictions are concatenated across folds before AUC computation, as in the nestedcv design

## Limitations

- The inner-CV AUC is optimistically biased by construction (tuning and filtering performed on inner data); it must not be reported as model performance
- A final round of CV on the whole dataset is needed to fit the final model for external prediction — the outer-fold AUC applies to the CV procedure, not a single fitted model
- Default 10 × 10 folds can be computationally expensive; the article notes parallelisation of the outer loop via cv.cores (mclapply on unix/mac, parLapply on Windows) or the future framework

## Evidence

- [abstract] Inner CV is used to tune models and outer CV is used to determine model performance without bias: "Inner CV is used to tune models and outer CV is used to determine model performance without bias"
- [abstract] the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets: "the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] nestedcv partitions the dataset into outer and inner folds (default 10 x 10 folds): "`nestedcv` partitions the dataset into outer and inner folds (default 10 x 10
folds)."
- [intro] filtering on the whole dataset creates a bias when determining accuracy: "it has been demonstrated that filtering on the whole dataset creates a bias when determining accuracy of models (Vabalas et al, 2019)"
- [other] Raw RNA-Seq data for the R4RA example at ArrayExpress E-MTAB-11611: "# Raw RNA-Seq data for this example is located at:
# https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-11611/"
- [readme] README: filters passed via filterFUN with options through filter_options: "Filters can be used by setting the `filterFUN` argument. Options for the filter 
function are passed as a list through `filter_options`."
