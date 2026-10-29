---
name: rna-seq-expression-matrix-processing
description: Use when the input is raw RNA-Seq data from a high-dimensional transcriptomics
  experiment (P >> n, e.g. ArrayExpress E-MTAB-11611) that must be converted into
  a predictor matrix x and outcome vector y for glmnet/nestedcv modelling.
when_to_use_negative:
- Input is already a processed expression matrix paired with a numeric/factor outcome
  vector ready for nestcv.glmnet
- The goal is unsupervised analysis or the data lack outcome labels, since fold stratification
  on the outcome (caret createFolds) and filter functions like ttest_filter require
  y
- The dataset is small and low-dimensional enough for a conventional 2/3-1/3 train/test
  split, where nested CV's whole-dataset testing is unnecessary
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: nestedcv
  role: Consumes the processed expression matrix and outcome vector; partitions data
    into stratified outer/inner folds and runs nestcv.glmnet with embedded feature-selection
    filters
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the elastic-net regularized models on the processed matrix, including
    cross-validation of the alpha parameter; coefficients of the final fit are extracted
    after processing
- name: R
  role: Environment in which the RNA-Seq data are loaded, subset to the rituximab
    arm, matrix-encoded, and passed to nestedcv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:328fda8043221d05c10f526b1858b09acef5fe60c2b4f0563f3967662ad3c2f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# rna-seq-expression-matrix-processing

## Summary

Prepare raw transcriptomic count data (e.g. RNA-Seq from ArrayExpress E-MTAB-11611 R4RA rheumatoid arthritis samples) into a sample-by-gene expression matrix with aligned outcome labels, so it can be used for nested cross-validated elastic-net modelling with the nestedcv R package.

## When to use

Use when the input is raw RNA-Seq data from a high-dimensional transcriptomics experiment (P >> n, e.g. ArrayExpress E-MTAB-11611) that must be converted into a predictor matrix x and outcome vector y for glmnet/nestedcv modelling.

## When NOT to use

- Input is already a processed expression matrix paired with a numeric/factor outcome vector ready for nestcv.glmnet
- The goal is unsupervised analysis or the data lack outcome labels, since fold stratification on the outcome (caret createFolds) and filter functions like ttest_filter require y
- The dataset is small and low-dimensional enough for a conventional 2/3-1/3 train/test split, where nested CV's whole-dataset testing is unnecessary

## Inputs

- Raw RNA-Seq data from ArrayExpress experiment E-MTAB-11611 (R4RA rheumatoid arthritis RNA-Seq)
- Response/outcome labels for the treatment subset (e.g. rituximab-treated responder status)
- Optional: data.frame of predictors containing factors or character variables requiring one-hot encoding via one_hot()

## Outputs

- Numeric sample-by-gene expression matrix x compatible with glmnet/nestcv.glmnet
- Aligned outcome vector y (stratifiable for CV fold construction)
- Downstream: final_model_coefficients.csv with nonzero glmnet coefficients ranked by value (e.g. 21 genes plus intercept, top-ranked AC016582.3, PCBP3, TMEM170B, EIF4E3, SEC14L6, CEP85, APLF ... DIRAS3)

## How to apply

Load the raw RNA-Seq data from the ArrayExpress experiment (e.g. E-MTAB-11611, the R4RA rheumatoid arthritis cohort) and subset to the treatment arm of interest, such as the rituximab-treated samples. Construct a samples-as-rows, genes-as-columns matrix of expression values and attach the binary response labels (e.g. responder status after rituximab), ensuring row/column alignment between x and y. Because transcriptomic data are P >> n (tens of thousands of genes vs limited samples), convert any data.frame of predictors to a numeric matrix — using one_hot() if factors/characters are present — since nestcv.glmnet requires a matrix input. Stratify fold construction on the outcome variable (the default, via caret's createFolds()) so class balance is preserved in outer and inner CV folds. The processed matrix and label vector then feed directly into nestcv.glmnet(y, x, ...) for 10 x 10-fold nested CV with embedded feature-selection filters, followed by a final round of CV on the whole dataset to fit the final model.

## Related tools

- **nestedcv** (Consumes the processed expression matrix and outcome vector; partitions data into stratified outer/inner folds and runs nestcv.glmnet with embedded feature-selection filters) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the elastic-net regularized models on the processed matrix, including cross-validation of the alpha parameter; coefficients of the final fit are extracted after processing)
- **R** (Environment in which the RNA-Seq data are loaded, subset to the rituximab arm, matrix-encoded, and passed to nestedcv)

## Examples

```
library(nestedcv); x <- as.matrix(rnaseq_expr[, -1]); y <- factor(response_labels); res <- nestcv.glmnet(y, x, family = "binomial", cv.cores = cores); coef(res)
```

## Evaluation signals

- The processed x matrix and y vector have identical sample ordering, and y is a factor/binary outcome suitable for stratified fold creation
- The predictor matrix contains no factor/character columns (one_hot() applied) so nestcv.glmnet runs without type errors
- Downstream nested CV on the rituximab subset reproduces the published AUC of 0.7648 for the no-filter glmnet model
- The final refitted glmnet model retains 21 genes plus intercept, and the ranked coefficients in final_model_coefficients.csv exactly match the vignette order (AC016582.3, PCBP3, TMEM170B, EIF4E3, SEC14L6, CEP85, APLF ... DIRAS3)
- Filters are demonstrably nested within the outer CV loop (no filtering on the whole dataset before CV), avoiding information leakage

## Limitations

- Filtering predictors on the whole dataset before CV creates bias when determining model accuracy (Vabalas et al, 2019); feature selection must occur only within training folds
- Results depend on stochastic fold assignments; nestedcv default 10 x 10 stratified folds (caret createFolds) may not exactly reproduce published gene lists without matching fold indices
- The source document lacks changelog and full discussion text for the GitHub mirror, so version-specific behaviour changes are not documented

## Evidence

- [intro] Raw RNA-Seq data for this example is located at:
# https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-11611/: "Raw RNA-Seq data for this example is located at:
# https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-11611/"
- [intro] Dataframes of predictors with factors or character variables can be converted to a matrix using one-hot encoding: "Dataframes of predictors with factors or character variables can be converted to a matrix using one-hot encoding"
- [intro] By default `nestedcv` stratifies outer & inner CV folds based on the outcome variable using caret's `createFolds()` function.: "By default `nestedcv` stratifies outer & inner CV folds based on the outcome variable using caret's `createFolds()` function."
- [readme] Use `summary()` to see full information from the nested model fitting. `coef()` can be used to show the coefficients of the final fitted model.: "Use `summary()` to see full information from the nested model fitting. `coef()` can be used to show the coefficients of the final fitted model."
