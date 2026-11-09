---
name: rna-seq-treatment-response-classification
description: Use when classifying binary treatment response (e.g. rituximab responders
  vs non-responders in the R4RA E-MTAB-11611 cohort) from transcriptomic matrices
  where predictors vastly outnumber samples (P >> n) and an honest out-of-sample AUC
  is required.
when_to_use_negative:
- The dataset is large enough for a conventional 2/3-1/3 train/test split, or predictors
  are few relative to samples — the article motivates nested CV specifically for small-n,
  P >> n biomedical data
- You only need hyperparameter tuning, not an unbiased performance estimate — plain
  cv.glmnet inner CV suffices and is faster
- Predictors include factors/character variables not yet encoded — one_hot() conversion
  is needed before passing a matrix to glmnet
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: nestedcv
  role: R package providing nestcv.glmnet for fully nested k x l-fold CV with filters
    embedded in the outer loop
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the lasso/elastic-net regularised logistic model whose lambda and alpha
    are tuned within inner CV folds
- name: caret
  role: Provides createFolds() used by nestedcv to stratify outer and inner CV folds
    on the outcome
- name: pROC
  role: ROC AUC computation for left-out outer- and inner-fold predictions in binary
    comparisons
- name: R
  role: Runtime environment for the nested CV workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:39eaac843ed79c85ba870d81430bdb295bd905627a955f7bd00f83c13acc18ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - caret
  - glmnet
  - nestedcv
  - pROC
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# rna-seq-treatment-response-classification

## Summary

Builds an unbiased classifier of treatment response from high-dimensional RNA-seq expression data using nested cross-validation (nestcv.glmnet) with an embedded univariate t-test feature filter, so that filtering and hyperparameter tuning never leak into the outer test folds.

## When to use

Use when classifying binary treatment response (e.g. rituximab responders vs non-responders in the R4RA E-MTAB-11611 cohort) from transcriptomic matrices where predictors vastly outnumber samples (P >> n) and an honest out-of-sample AUC is required.

## When NOT to use

- The dataset is large enough for a conventional 2/3-1/3 train/test split, or predictors are few relative to samples — the article motivates nested CV specifically for small-n, P >> n biomedical data
- You only need hyperparameter tuning, not an unbiased performance estimate — plain cv.glmnet inner CV suffices and is faster
- Predictors include factors/character variables not yet encoded — one_hot() conversion is needed before passing a matrix to glmnet

## Inputs

- RNA-seq expression matrix (genes x samples), e.g. R4RA rituximab cohort from ArrayExpress accession E-MTAB-11611
- Binary treatment-response outcome vector (responder/non-responder) aligned to sample names
- Optional: number of physical cores for parallelising the outer CV loop (cv.cores)

## Outputs

- Fitted nestcv.glmnet object with outer_result per outer fold and tuned alpha/lambda
- Outer-fold (unbiased) ROC AUC from the nested model summary
- Inner-CV (innercv) ROC AUC, expected to be optimistically biased relative to the outer AUC
- CSV table of both AUC values and the comparison verdict (e.g. rituximab_nestedcv_auc.csv)

## How to apply

Load the RNA-seq expression matrix (e.g. E-MTAB-11611 from ArrayExpress) and the binary response outcome, then call nestcv.glmnet from the nestedcv R package with default 10 x 10 stratified outer/inner folds (stratified on the outcome via caret's createFolds). Set filterFUN = ttest_filter with filter_options = list(nfilter = 300) so the univariate t-test filter runs inside the outer CV loop on training data only, avoiding the whole-dataset filtering bias described by Vabalas et al. The inner 10-fold CV tunes glmnet hyperparameters (lambda and elastic-net alpha), the model is fitted on the inner training data and tested on the left-out outer fold, and outer-fold predictions are concatenated for accuracy measures. Extract the outer-fold ROC AUC (via summary of the nestcv.glmnet object) as the unbiased performance estimate, and separately report the innercv ROC AUC, which is expected to be optimistically higher (e.g. 0.783 outer vs 0.863 inner in the R4RA rituximab example). Optionally run a final round of CV on the whole dataset to fit the final model for prediction on external data, and write both AUC values plus the comparison verdict to an output CSV.

## Related tools

- **nestedcv** (R package providing nestcv.glmnet for fully nested k x l-fold CV with filters embedded in the outer loop) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the lasso/elastic-net regularised logistic model whose lambda and alpha are tuned within inner CV folds)
- **caret** (Provides createFolds() used by nestedcv to stratify outer and inner CV folds on the outcome)
- **pROC** (ROC AUC computation for left-out outer- and inner-fold predictions in binary comparisons)
- **R** (Runtime environment for the nested CV workflow)

## Examples

```
res <- nestcv.glmnet(y, x, family = "binomial", filterFUN = ttest_filter, filter_options = list(nfilter = 300), cv.cores = cores); summary(res)
```

## Evaluation signals

- Outer-fold AUC must be lower than the inner-fold (innercv_roc) AUC — e.g. 0.783 vs 0.863 in the R4RA rituximab benchmark — confirming the outer estimate is the unbiased one
- The t-test filter must be nested inside the outer CV loop (passed as filterFUN to nestcv.glmnet), not run on the whole dataset before CV
- Outer folds default to 10 x 10 and are stratified on the outcome via caret's createFolds(); verify fold structure in the returned object
- summary(res) must report both outer ROC AUC and innercv ROC AUC; plot_lambdas(res)/plot_alphas(res) should show per-outer-fold tuning trajectories
- Output CSV contains exactly the two AUC values and the comparison verdict that outer < inner

## Limitations

- Filtering predictors on the whole dataset before CV creates optimistic bias when determining model accuracy (Vabalas et al, 2019) — the filter must stay within the outer loop
- Inner-fold AUC is optimistically biased and should not be reported as model performance
- Biomedical test sets may be too small for reliable accuracy estimation with a simple split, which is why nested CV is used instead
- Outer-loop parallelisation via mclapply works on unix/mac; parLapply is required on Windows

## Evidence

- [abstract] Inner CV is used to tune models and outer CV is used to determine model performance without bias: "Inner CV is used to tune models and outer CV is used to determine model performance without bias"
- [abstract] Filters nested within outer CV loop to avoid leakage: "the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] Default 10 x 10 stratified folds: "`nestedcv` partitions the dataset into outer and inner folds (default 10 x 10
folds)."
- [intro] Outer test predictions concatenated for accuracy: "the unseen test predictions from the outer folds are compared against the true results for the outer test folds and the results concatenated"
- [intro] ttest_filter example call: "filt <- ttest_filter(y, x, nfilter = 100)"
- [intro] Whole-dataset filtering causes bias: "it has been
demonstrated that filtering on the whole dataset creates a bias when determining
accuracy of models (Vabalas et al, 2019)"
- [readme] README: nested CV for glmnet with alpha tuning: "Nested cross-validation (CV) for the glmnet and caret packages. With glmnet this
includes cross-validation of elastic net alpha parameter."
- [readme] README: filterFUN argument usage: "Filters can be used by setting the `filterFUN` argument. Options for the filter 
function are passed as a list through `filter_options`."
