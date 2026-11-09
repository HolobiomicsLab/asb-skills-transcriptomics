---
name: rnaseq-expression-median-cutoff-filtering
description: Use when starting from a raw RNA-seq expression matrix (e.g. >16,000
  genes from ArrayExpress E-MTAB-11611) and low-abundance genes must be dropped by
  a median-expression cut-off before fitting models such as nestcv.glmnet.
when_to_use_negative:
- Supervised feature selection is intended — filtering on the whole dataset creates
  bias when determining model accuracy (Vabalas et al, 2019); use nestedcv filter
  functions inside the outer loop instead.
- The input is already a reduced feature table or the model is not high-dimensional
  (P >> n), where a median-abundance cutoff is unnecessary.
- The data are not RNA-seq expression measurements (e.g. Sonar, BostonHousing2, iris
  predictors), where abundance-based gene filtering does not apply.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: nestedcv
  role: Provides the nestcv.glmnet workflow that consumes the median-filtered matrix,
    with 10 x 10 stratified nested CV and inner-CV tuning of alpha and lambda
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the elastic-net regularized logistic model inside nestcv.glmnet after
    the expression cut-off is applied
  repo: https://CRAN.R-project.org/package=nestedcv
- name: R
  role: Execution environment for data download, median-cutoff filtering, and running
    nestedcv
  repo: https://CRAN.R-project.org/package=nestedcv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:13e1f8c621cbde7c811886efbabd86067216153833aae2ee1bcacc0ea9e85c1b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rnaseq-expression-median-cutoff-filtering

## Summary

Pre-processing skill that removes low-abundance genes from an RNA-seq expression matrix by applying a median-expression cut-off across samples, as done in the nestedcv vignette on the R4RA rheumatoid arthritis RNA-seq dataset (E-MTAB-11611) before nested cross-validation.

## When to use

Use when starting from a raw RNA-seq expression matrix (e.g. >16,000 genes from ArrayExpress E-MTAB-11611) and low-abundance genes must be dropped by a median-expression cut-off before fitting models such as nestcv.glmnet.

## When NOT to use

- Supervised feature selection is intended — filtering on the whole dataset creates bias when determining model accuracy (Vabalas et al, 2019); use nestedcv filter functions inside the outer loop instead.
- The input is already a reduced feature table or the model is not high-dimensional (P >> n), where a median-abundance cutoff is unnecessary.
- The data are not RNA-seq expression measurements (e.g. Sonar, BostonHousing2, iris predictors), where abundance-based gene filtering does not apply.

## Inputs

- Raw RNA-seq gene expression matrix/counts (e.g. from ArrayExpress experiment E-MTAB-11611, >16,000 genes)
- Sample metadata annotating treatment arm (rituximab subset, n=68) and outcome labels

## Outputs

- Median-expression-filtered predictor matrix (low-abundance genes removed)
- Outcome vector y for the selected treatment arm (n=68 rituximab samples)
- cv_performance_metrics.csv with outer-CV AUC, accuracy and balanced accuracy from nestcv.glmnet

## How to apply

Obtain the raw RNA-seq data from the ArrayExpress accession (E-MTAB-11611 in the vignette), then compute the per-gene median expression across all samples and retain only genes whose median expression passes the cut-off. This is an unsupervised, sample-blind reduction, so it is applied before CV splitting and does not leak outcome information, unlike supervised filters (t-test, Wilcoxon, ANOVA, ReliefF) which nestedcv insists be nested inside the outer CV loop. After filtering, subset samples to the analysis arm (rituximab arm, n=68) and form the outcome vector y and predictor matrix x for nestcv.glmnet. Run nestcv.glmnet with default 10 x 10 stratified outer/inner folds (stratified via caret's createFolds) and no filterFUN, tuning elastic-net alpha and lambda on inner folds. Judge success by unbiased outer-CV metrics computed on concatenated left-out fold predictions, against the vignette reference values of AUC 0.7648, accuracy 0.7059, balanced accuracy 0.6773.

## Related tools

- **nestedcv** (Provides the nestcv.glmnet workflow that consumes the median-filtered matrix, with 10 x 10 stratified nested CV and inner-CV tuning of alpha and lambda) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the elastic-net regularized logistic model inside nestcv.glmnet after the expression cut-off is applied) — https://CRAN.R-project.org/package=nestedcv
- **R** (Execution environment for data download, median-cutoff filtering, and running nestedcv) — https://CRAN.R-project.org/package=nestedcv

## Examples

```
keep <- apply(x, 1, median) > 0; x <- x[keep, ]; res <- nestcv.glmnet(y, x, cv.cores = cores)
```

## Evaluation signals

- The filtered predictor matrix contains only genes whose median expression across samples passes the cut-off; low-abundance genes are absent.
- Sample count matches the rituximab treatment arm (n=68) with a correctly formed binary outcome vector.
- nestcv.glmnet runs with filterFUN unset (no supervised filter) and default 10 x 10 stratified outer/inner folds.
- Outer-CV metrics on concatenated left-out fold predictions approximate the vignette reference: AUC 0.7648, accuracy 0.7059, balanced accuracy 0.6773 (write to cv_performance_metrics.csv).
- No supervised filter (t-test/Wilcoxon/ANOVA/ReliefF) was applied outside the CV loops, confirming no information leakage from performance test sets.

## Limitations

- The median-expression cut-off is dataset-specific to the vignette's RNA-seq preprocessing; the exact threshold is not stated in the provided context and must be taken from the nestedcv vignette.
- Unbiased performance (outer CV) is only valid for this n=68 subset and no-filter configuration; adding filters or changing folds changes results.
- Biomedical datasets with P >> n are too small for a conventional 2/3-1/3 split, so metrics rely entirely on nested CV rather than a held-out test set.

## Evidence

- [other] Raw RNA-Seq data for this example is located at ArrayExpress E-MTAB-11611; vignette applies the median-expression cut-off to genes and subsets to the rituximab arm (n=68).: [evidence span withheld — anchor status: not_found]
- [abstract] Article motivates nested CV for high-dimensional expression data where predictors far outnumber samples.: "Nested CV with embedded feature selection is especially suited to biomedical data where the sample size is frequently limited, but the number of predictors may be significantly larger (P ≫ n)"
- [intro] Filtering on the whole dataset creates accuracy bias; feature selection must stay inside CV loops.: "it has been
demonstrated that filtering on the whole dataset creates a bias when determining
accuracy of models (Vabalas et al, 2019)"
- [intro] Default fold structure used by nestedcv after the expression matrix is prepared.: "`nestedcv` partitions the dataset into outer and inner folds (default 10 x 10
folds)."
- [readme] README confirms the R/nestcv.glmnet invocation pattern used with the filtered matrix.: "res <- nestcv.glmnet(y, x, family = "multinomial", cv.cores = cores)"
