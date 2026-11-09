---
name: feature-selection-leakage-bias-assessment
description: Use when validating a high-dimensional (P >> n) classification pipeline
  where a feature-selection filter precedes model fitting, to check whether reported
  accuracy is inflated by information leakage from the test set into the filter.
when_to_use_negative:
- The dataset has real, validated signal and only a train/test split without embedded
  filtering is used — the bias demonstrated here arises specifically from whole-dataset
  filtering before CV
- The number of predictors is not much larger than the sample size (P >> n), since
  leakage-driven overfitting of this magnitude is characteristic of high-dimensional
  biomedical data
- The goal is unbiased hyperparameter tuning rather than assessing whether filter
  placement biases performance estimation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: nestedcv
  role: Runs the fully nested 10 x 10-fold CV with ttest_filter embedded in the outer
    loop via nestcv.glmnet (Scheme C)
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the lasso model under each scheme, including inside the nested-CV outer
    folds
- name: pROC
  role: Computes ROC AUC on hold-out and concatenated outer test-fold predictions
- name: R
  role: Runtime environment for the simulation and all three schemes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:ca01088386ec53a61ba2c5914efcdecee4ce182e162a6a42a53ccb2d652bcdac
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
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# feature-selection-leakage-bias-assessment

## Summary

Quantifies the overoptimistic performance bias caused by performing univariate feature filtering (e.g. t-test) on the whole dataset before model fitting, by comparing whole-dataset filtering, inner-CV-only predictions, and fully nested CV with filters embedded in the outer loop on pure-gaussian-noise data (P >> n).

## When to use

Use when validating a high-dimensional (P >> n) classification pipeline where a feature-selection filter precedes model fitting, to check whether reported accuracy is inflated by information leakage from the test set into the filter.

## When NOT to use

- The dataset has real, validated signal and only a train/test split without embedded filtering is used — the bias demonstrated here arises specifically from whole-dataset filtering before CV
- The number of predictors is not much larger than the sample size (P >> n), since leakage-driven overfitting of this magnitude is characteristic of high-dimensional biomedical data
- The goal is unbiased hyperparameter tuning rather than assessing whether filter placement biases performance estimation

## Inputs

- Simulated pure gaussian-noise predictor matrix (X) with P >> n
- Random balanced binary outcome vector (y)
- R environment with nestedcv, glmnet and pROC installed

## Outputs

- Three ROC AUC estimates (whole-dataset filtering, inner-CV predictions, fully nested CV) tabulated side by side
- Comparison table written to noise_simulation_auc.csv
- Reference expectation: nested-CV AUC ≈ 0.50 vs inflated whole-dataset-filtering AUC

## How to apply

Generate a pure gaussian-noise predictor matrix with P >> n and a random balanced binary outcome, so any apparent signal is by definition leakage. In R with nestedcv/glmnet/pROC, run three schemes side by side: (A) rank all predictors by whole-dataset t-test filtering (ttest_filter), fit a lasso via glmnet on a 2:1 train/test split and compute hold-out ROC AUC with pROC — this yields an overoptimistic AUC (Fig. 2A red line); (B) fit nested CV but compute AUC from left-out inner CV folds, which still share training data with the filter and remain biased; (C) run fully nested CV with nestcv.glmnet (default 10 x 10 folds), passing ttest_filter via filterFUN so the filter sees only inner training folds, and compute ROC AUC on concatenated outer test-fold predictions — this should report AUC close to 0.50. Tabulate the three AUCs and write the comparison to noise_simulation_auc.csv.

## Related tools

- **nestedcv** (Runs the fully nested 10 x 10-fold CV with ttest_filter embedded in the outer loop via nestcv.glmnet (Scheme C)) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the lasso model under each scheme, including inside the nested-CV outer folds)
- **pROC** (Computes ROC AUC on hold-out and concatenated outer test-fold predictions)
- **R** (Runtime environment for the simulation and all three schemes)

## Examples

```
res <- nestcv.glmnet(y, x, family = "binomial", filterFUN = ttest_filter, filter_options = list(nfilter = 100), cv.cores = cores); roc(res$y, as.numeric(res$pred))
```

## Evaluation signals

- Scheme C (fully nested CV with ttest_filter inside the outer loop) reports ROC AUC close to 0.50 on pure-noise data
- Scheme A (whole-dataset t-test filtering, hold-out test) reports a substantially inflated AUC relative to 0.50
- Scheme B (AUC from left-out inner CV folds) also exceeds 0.50, confirming inner-fold predictions are not unbiased
- The three AUCs are tabulated side by side and written to noise_simulation_auc.csv
- Filtering in Scheme C is verified to be nested within the outer CV loop so the filter sees only training folds

## Limitations

- Bias from whole-dataset filtering is demonstrated on pure gaussian-noise data with P >> n; magnitude in real datasets will vary with signal structure
- nestedcv defaults to 10 x 10 folds, which can be computationally heavy; outer loop parallelisation (cv.cores) may be needed on many-core machines
- The finding follows prior work (Vabalas et al, 2019) showing whole-dataset filtering biases accuracy estimation; it does not address leakage sources beyond filter placement

## Evidence

- [intro] whole-dataset filtering bias: "it has been demonstrated that filtering on the whole dataset creates a bias when determining accuracy of models (Vabalas et al, 2019)"
- [abstract] nested filter placement prevents leakage: "the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [abstract] inner vs outer fold roles: "Inner CV is used to tune models and outer CV is used to determine model performance without bias"
- [abstract] P >> n context: [evidence span withheld — anchor status: not_found]
- [readme] filter usage argument: "Filters can be used by setting the `filterFUN` argument. Options for the filter function are passed as a list through `filter_options`."
