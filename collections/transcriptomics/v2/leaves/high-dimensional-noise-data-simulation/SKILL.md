---
name: high-dimensional-noise-data-simulation
description: 'Use when validating a cross-validation workflow for leakage bias: you
  need a null dataset (no true signal, P >> n) where any AUC above ~0.5 from filtered
  models demonstrates information leakage.'
when_to_use_negative:
- Your goal is to fit a final predictive model on real data rather than to benchmark
  CV design for leakage bias — the noise simulation exists only as a null control
- Your data are low-dimensional (P << n) where whole-dataset filtering bias is not
  the concern and a conventional train/test split suffices
- Your dataset already contains real signal structure — any elevated AUC then reflects
  signal, not leakage, defeating the null-dataset rationale
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: nestedcv
  role: Runs the fully nested 10 x 10-fold CV via nestcv.glmnet with filterFUN embedded
    in the outer loop, and provides the simulated imbalanced vignette dataset
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the lasso logistic model inside each nested CV scheme on the simulated
    noise data
  repo: https://CRAN.R-project.org/package=nestedcv
- name: pROC
  role: Computes ROC AUC on hold-out and outer test-fold predictions from the noise
    simulation
- name: R
  role: Environment in which the gaussian-noise predictor matrix and random balanced
    binary outcome are generated and all schemes are executed
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:778f42ee8d312bf7009351c7e948ea715b530b15c402795afe1b6ce41c7f4c40
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  - pROC
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# high-dimensional-noise-data-simulation

## Summary

Generates pure gaussian-noise predictor matrices with P >> n and random balanced binary outcomes to benchmark whether whole-dataset feature filtering inflates apparent ROC AUC versus fully nested CV with filters embedded in the outer loop.

## When to use

Use when validating a cross-validation workflow for leakage bias: you need a null dataset (no true signal, P >> n) where any AUC above ~0.5 from filtered models demonstrates information leakage.

## When NOT to use

- Your goal is to fit a final predictive model on real data rather than to benchmark CV design for leakage bias — the noise simulation exists only as a null control
- Your data are low-dimensional (P << n) where whole-dataset filtering bias is not the concern and a conventional train/test split suffices
- Your dataset already contains real signal structure — any elevated AUC then reflects signal, not leakage, defeating the null-dataset rationale

## Inputs

- Nothing external: simulated pure gaussian-noise predictor matrix in R (n samples x P predictors, P >> n) with a random balanced binary outcome vector
- Optional vignette variant: imbalanced simulated dataset with 150 samples and 20,000 predictors where only the first 30 predictors carry weak signal

## Outputs

- ROC AUC estimates for biased (whole-dataset filtering, or unnested oversampling) and unbiased (fully nested CV) schemes, computed on concatenated outer-fold test predictions
- Side-by-side comparison table written to CSV (e.g. noise_simulation_auc.csv, imbalance_auc_comparison.csv)

## How to apply

Simulate a pure gaussian-noise predictor matrix with many more predictors than samples (P >> n) and assign a random balanced binary outcome, so the true AUC is 0.50 by construction. Run Scheme A by ranking all predictors with whole-dataset univariate filtering (ttest_filter), fitting a glmnet lasso on a 2:1 train/test split, and scoring the hold-out with pROC — this reuses test samples during filtering and inflates AUC (Fig. 2A red line). Run Scheme C with nestcv.glmnet (default 10 x 10 folds) embedding filterFUN inside the outer loop so filters see only training folds, then compute AUC on concatenated outer test-fold predictions, which should stay near 0.50. A variant simulation (nestedcv vignette: 150 samples, 20,000 predictors, only the first 30 predictors carrying weak signal, imbalanced classes) benchmarks unnested vs nested random oversampling balancing. Tabulate the AUC estimates side by side (e.g. noise_simulation_auc.csv) to quantify the bias.

## Related tools

- **nestedcv** (Runs the fully nested 10 x 10-fold CV via nestcv.glmnet with filterFUN embedded in the outer loop, and provides the simulated imbalanced vignette dataset) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the lasso logistic model inside each nested CV scheme on the simulated noise data) — https://CRAN.R-project.org/package=nestedcv
- **pROC** (Computes ROC AUC on hold-out and outer test-fold predictions from the noise simulation)
- **R** (Environment in which the gaussian-noise predictor matrix and random balanced binary outcome are generated and all schemes are executed)

## Examples

```
res <- nestcv.glmnet(y, x, family = "binomial", filterFUN = ttest_filter, filter_options = list(nfilter = 100))
```

## Evaluation signals

- Unbiased fully nested CV (Scheme C) reports ROC AUC close to 0.50 on pure gaussian-noise data
- Whole-dataset t-test filtering scheme (Scheme A) yields substantially overoptimistic AUC above 0.50, demonstrating the leakage bias
- Simulated classes are balanced (or imbalanced as in the vignette variant: 150 samples, 20,000 predictors, first 30 weakly informative)
- AUC values from all schemes are tabulated side by side and written to the comparison CSV
- In the nested run, the filter never sees outer test folds — verified by inspecting filter behaviour inside nestcv.glmnet's outer loop

## Limitations

- Conclusions are specific to the pure gaussian-noise / P >> n regime; datasets too small for a 2/3-1/3 split behave differently and need nested CV instead
- Unnested oversampling or SMOTE applied outside the CV loop also causes leakage and upward bias — balancing must be nested within outer folds
- Whole-dataset filtering bias is a documented phenomenon (Vabalas et al, 2019) but the simulation only quantifies it for univariate filtering plus glmnet lasso

## Evidence

- [abstract] Nested CV with embedded feature selection is especially suited to biomedical data where the sample size is frequently limited, but the number of predictors may be significantly larger (P ≫ n): "Nested CV with embedded feature selection is especially suited to biomedical data where the sample size is frequently limited, but the number of predictors may be significantly larger (P ≫ n)"
- [intro] whole-dataset filtering bias documented: "it has been demonstrated that filtering on the whole dataset creates a bias when determining accuracy of models (Vabalas et al, 2019)"
- [abstract] filters nested to avoid leakage: "the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [readme] filterFUN argument in README: "Filters can be used by setting the `filterFUN` argument. Options for the filter 
function are passed as a list through `filter_options`."
