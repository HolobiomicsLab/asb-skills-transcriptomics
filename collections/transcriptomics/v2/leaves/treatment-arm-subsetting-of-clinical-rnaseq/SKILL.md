---
name: treatment-arm-subsetting-of-clinical-rnaseq
description: Use when you need the >16,000-gene rituximab-arm RNA-seq subset (n=68)
  from E-MTAB-11611 as a binary-classification input matrix for nested CV or feature-selection
  benchmarks.
when_to_use_negative:
- Input is already a feature-reduced or pre-filtered expression matrix for a single
  arm — the median-expression cut-off and arm subsetting would be applied twice
- Analysis targets multi-class or regression outcomes — this skill produces a binary
  rituximab-arm classification setup
- Sample size permits a conventional 2/3-1/3 train/test split — nested CV on the full
  subset is designed for the P >> n regime where such splits are unreliable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: nestedcv
  role: Provides nestcv.glmnet for 10 x 10 stratified nested CV with inner-fold elastic-net
    tuning on the subset matrix
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the elastic-net regularised logistic model whose alpha and lambda are
    tuned on inner CV folds
- name: R
  role: Runtime environment in which data download, subsetting and nested CV are scripted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:3ce2d20c695ad8d9e77382e20bf43bc83938033aca56261275e99009679cda28
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# treatment-arm-subsetting-of-clinical-rnaseq

## Summary

Prepares the R4RA rheumatoid arthritis RNA-seq benchmark by downloading raw data for E-MTAB-11611 from ArrayExpress, applying a median-expression gene cut-off, and subsetting samples to the rituximab treatment arm (n=68) with an outcome vector and predictor matrix for nested CV modelling.

## When to use

Use when you need the >16,000-gene rituximab-arm RNA-seq subset (n=68) from E-MTAB-11611 as a binary-classification input matrix for nested CV or feature-selection benchmarks.

## When NOT to use

- Input is already a feature-reduced or pre-filtered expression matrix for a single arm — the median-expression cut-off and arm subsetting would be applied twice
- Analysis targets multi-class or regression outcomes — this skill produces a binary rituximab-arm classification setup
- Sample size permits a conventional 2/3-1/3 train/test split — nested CV on the full subset is designed for the P >> n regime where such splits are unreliable

## Inputs

- Raw RNA-Seq data for ArrayExpress experiment E-MTAB-11611 (R4RA rheumatoid arthritis)
- Sample metadata mapping samples to treatment arms (rituximab vs other) and response outcome
- Median-expression gene filter cut-off applied to the raw count/expression matrix

## Outputs

- Rituximab-arm subset: outcome vector y (n=68) and predictor matrix x (>16,000 genes after median-expression filtering)
- cv_performance_metrics.csv with outer-CV AUC, accuracy and balanced accuracy

## How to apply

Obtain the raw RNA-Seq data for experiment E-MTAB-11611 from ArrayExpress (https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-11611/). Process per the nestedcv vignette: apply the median-expression cut-off to genes to reduce the predictor set, then subset samples to the rituximab treatment arm (n=68). Form the binary outcome vector (treatment response) and the predictor matrix from the retained samples and filtered genes. Feed the resulting y/x pair into nestcv.glmnet with default 10 x 10 stratified outer/inner folds (folds stratified on the outcome via caret's createFolds()); the no-filter variant passes no filterFUN so glmnet elastic-net tuning of alpha and lambda on inner folds is the only embedded selection. Extract concatenated left-out outer-fold test predictions to compute unbiased AUC, accuracy and balanced accuracy (expected no-filter values: AUC 0.7648, accuracy 0.7059, balanced accuracy 0.6773), writing them to cv_performance_metrics.csv for comparison against the vignette.

## Related tools

- **nestedcv** (Provides nestcv.glmnet for 10 x 10 stratified nested CV with inner-fold elastic-net tuning on the subset matrix) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the elastic-net regularised logistic model whose alpha and lambda are tuned on inner CV folds)
- **R** (Runtime environment in which data download, subsetting and nested CV are scripted)

## Examples

```
res <- nestcv.glmnet(y, x, family = "binomial", cv.cores = cores); summary(res)
```

## Evaluation signals

- Outer-CV metrics from the no-filter run match the vignette values: AUC 0.7648, accuracy 0.7059, balanced accuracy 0.6773
- Subset dimensionality checks: exactly n=68 samples in the rituximab arm and >16,000 genes after the median-expression cut-off
- cv_performance_metrics.csv exists and its columns report AUC, accuracy and balanced accuracy computed on concatenated outer-fold test predictions
- Folds are stratified on the outcome (default caret createFolds()) — verify fold composition is balanced across classes
- No filterFUN is passed in the no-filter variant, so no filter-step information enters model fitting beyond glmnet's own regularisation

## Limitations

- Filtering predictors on the whole dataset creates bias when determining model accuracy; any feature selection must be nested within the outer CV loop (Vabalas et al, 2019)
- The subset is small (n=68) with P >> n, so accuracy estimated on a single left-out test set is unstable; outer-CV concatenation is required for unbiased measures
- Results depend on the default 10 x 10 fold structure and stratification; custom fold indices change reproducibility of the reported metrics

## Evidence

- [intro] Raw RNA-Seq data source for E-MTAB-11611: "# Raw RNA-Seq data for this example is located at:
# https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-11611/"
- [abstract] Outer CV performance is unbiased: "Inner CV is used to tune models and outer CV is used to determine model performance without bias"
- [intro] Default 10 x 10 stratified folds: "`nestedcv` partitions the dataset into outer and inner folds (default 10 x 10
folds)."
- [intro] Whole-dataset filtering causes bias: "it has been
demonstrated that filtering on the whole dataset creates a bias when determining
accuracy of models (Vabalas et al, 2019)"
- [intro] Outer fold predictions are concatenated for accuracy measures: "the unseen test predictions from the outer folds are compared against the true results for the outer test folds and the results concatenated"
