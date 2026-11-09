---
name: binary-classification-metric-evaluation
description: Use when you need unbiased performance estimates (AUC, accuracy, balanced
  accuracy) for a binary classifier fitted on high-dimensional transcriptomic data
  via nested cross-validation with nestedcv, e.g. to compare against vignette-reported
  values.
when_to_use_negative:
- The dataset is large enough for a conventional 2/3-1/3 split — the article motivates
  nested CV specifically for small biomedical datasets where P >> n.
- Feature filtering has already been performed on the whole dataset — filtering on
  the whole dataset creates a bias when determining accuracy of models.
- The outcome is multinomial or regression — use family-appropriate metrics (e.g.
  the iris multinomial glmnet example or BostonHousing2 regression) rather than binary
  AUC/balanced accuracy.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: nestedcv
  role: Implements the nested k x l-fold CV loop, produces outer-fold test predictions
    and performance summary for the fitted glmnet model
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the elastic-net regularized logistic model whose lambda and alpha are
    tuned by inner CV inside nestcv.glmnet
- name: caret
  role: Provides createFolds() used to stratify outer and inner CV folds on the outcome
    variable
- name: R
  role: Execution environment for the nestedcv pipeline and writing cv_performance_metrics.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:b750e926ffcc2ae4bcf4e3212b7a78e6806d3d6c03df0d5102d8ad959cc01fb6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - caret
  - glmnet
  - nestedcv
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# binary-classification-metric-evaluation

## Summary

Extracts unbiased outer-CV binary classification performance metrics (AUC, accuracy, balanced accuracy) from nestedcv nested cross-validation runs, computed on concatenated left-out outer-fold test predictions. Used to benchmark nested glmnet models on high-dimensional transcriptomic data such as the R4RA rituximab RNA-seq subset.

## When to use

Use when you need unbiased performance estimates (AUC, accuracy, balanced accuracy) for a binary classifier fitted on high-dimensional transcriptomic data via nested cross-validation with nestedcv, e.g. to compare against vignette-reported values.

## When NOT to use

- The dataset is large enough for a conventional 2/3-1/3 split — the article motivates nested CV specifically for small biomedical datasets where P >> n.
- Feature filtering has already been performed on the whole dataset — filtering on the whole dataset creates a bias when determining accuracy of models.
- The outcome is multinomial or regression — use family-appropriate metrics (e.g. the iris multinomial glmnet example or BostonHousing2 regression) rather than binary AUC/balanced accuracy.

## Inputs

- Outcome vector y (binary factor) for the rituximab treatment arm of E-MTAB-11611 (n=68)
- Predictor matrix x of raw RNA-seq gene expression after median-expression cut-off (>16,000 genes)
- Fitted nestcv.glmnet result object (outer_result folds and left-out test predictions)

## Outputs

- cv_performance_metrics.csv containing outer-CV AUC, accuracy and balanced accuracy
- summary(res) report of the nested model fitting with outer-fold performance measures

## How to apply

Fit the model with nestcv.glmnet (default 10 x 10 stratified outer/inner folds; folds are stratified on the outcome variable via caret's createFolds), tuning glmnet elastic-net parameters alpha and lambda on inner folds. The inner fold CV is used to tune optimal hyperparameters, then the model is fitted on the whole inner fold and tested on the left-out data from the outer fold. Performance is not read from a single hold-out split: the unseen test predictions from the outer folds are compared against the true results and concatenated, and AUC, accuracy and balanced accuracy are computed on those concatenated predictions to avoid the bias from filtering or tuning on the whole dataset. For the R4RA rituximab example (E-MTAB-11611 subset, n=68, >16,000 genes after median-expression cut-off, no filterFUN), the expected outer-CV values are AUC 0.7648, accuracy 0.7059, balanced accuracy 0.6773; write the metrics to cv_performance_metrics.csv and compare against the nestedcv vignette values.

## Related tools

- **nestedcv** (Implements the nested k x l-fold CV loop, produces outer-fold test predictions and performance summary for the fitted glmnet model) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the elastic-net regularized logistic model whose lambda and alpha are tuned by inner CV inside nestcv.glmnet)
- **caret** (Provides createFolds() used to stratify outer and inner CV folds on the outcome variable)
- **R** (Execution environment for the nestedcv pipeline and writing cv_performance_metrics.csv)

## Examples

```
res <- nestcv.glmnet(y, x, family = "binomial", cv.cores = cores); summary(res)
```

## Evaluation signals

- On the rituximab subset (n=68, no filter), outer-CV metrics should reproduce AUC 0.7648, accuracy 0.7059, balanced accuracy 0.6773 within rounding against the nestedcv vignette.
- AUC, accuracy and balanced accuracy must be computed on concatenated left-out outer-fold test predictions, not on inner-CV or whole-dataset fitted values.
- summary(res) reports nested model fitting results with outer-CV measures; coef() gives the final fitted model coefficients.
- Folds are stratified on the outcome variable by default (10 x 10 outer/inner), so class proportions should be preserved across folds.
- No filterFUN should be set for the no-filter benchmark; inner CV tunes both lambda and alpha before each outer-fold test.

## Limitations

- Outer-fold test sets are small on limited-n datasets (here 68 samples across 10 outer folds), so metric estimates carry sampling variability.
- Filtering predictors on the whole dataset creates bias when determining accuracy; only filters nested within the outer CV loop avoid information leakage from performance test sets.
- Metrics are meaningful only if a final round of CV on the whole dataset is used to fit the final model for external prediction, separate from the performance estimate.

## Evidence

- [abstract] Inner CV is used to tune models and outer CV is used to determine model performance without bias: "Inner CV is used to tune models and outer CV is used to determine model performance without bias"
- [intro] the unseen test predictions from the outer folds are compared against the true results for the outer test folds and the results concatenated: "the unseen test predictions from the outer folds are compared against the true results for the outer test folds and the results concatenated"
- [intro] By default nestedcv stratifies outer & inner CV folds based on the outcome variable using caret's createFolds() function.: "By default `nestedcv` stratifies outer & inner CV folds based on the outcome
variable using caret's `createFolds()` function."
- [results] Running nestcv.glmnet on the rituximab subset with no filter yielded outer-CV performance of AUC 0.7648, accuracy 0.7059, and balanced accuracy 0.6773.: [evidence span withheld — anchor status: not_found]
- [readme] Use summary() to see full information from the nested model fitting. coef() can be used to show the coefficients of the final fitted model.: "Use `summary()` to see full information from the nested model fitting. `coef()`
can be used to show the coefficients of the final fitted model."
