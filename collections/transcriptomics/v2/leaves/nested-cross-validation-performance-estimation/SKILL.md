---
name: nested-cross-validation-performance-estimation
description: Use when a small-n, high-predictor-count (P >> n) dataset needs unbiased
  model accuracy (e.g. ROC AUC) while feature selection or class balancing must be
  embedded inside the CV loop to avoid information leakage.
when_to_use_negative:
- The dataset is large enough for a conventional 2/3-1/3 train/test split without
  severe small-test-set inaccuracy — nested CV is motivated by biomedical datasets
  too small for this split
- The goal is feature ranking or biological interpretation rather than unbiased accuracy
  estimation of a final fitted model
- Predictor count is modest relative to sample size and no embedded feature selection
  or hyperparameter tuning is needed, making simple k-fold CV sufficient
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2269
tools:
- name: nestedcv
  role: Implements fully nested k x l-fold CV (nestcv.glmnet, nestcv.train) with embedded
    feature-selection filters inside the outer loop
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits lasso/elastic-net regularised models with inner-loop cross-validation
    of lambda and alpha
  repo: https://CRAN.R-project.org/package=nestedcv
- name: pROC
  role: Computes ROC AUC on concatenated outer-fold left-out test predictions
- name: caret
  role: Alternative framework used by nestcv.train to tune other models (e.g. ranger
    random forest) within nested CV
  repo: https://CRAN.R-project.org/package=nestedcv
- name: CORElearn
  role: Provides ReliefF for relieff_filter embedded within nested CV
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:abf478ebd83bd596985a89c8eadd07253546d494fca8c82e2f8905f6ba7b7ddb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CORElearn
  - R
  - caret
  - glmnet
  - nestedcv
  - pROC
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# nested-cross-validation-performance-estimation

## Summary

Unbiased estimation of predictive model performance in high-dimensional (P >> n) settings using fully nested k x l-fold cross-validation via the nestedcv R package, with hyperparameter tuning on inner folds and unbiased performance measured on concatenated left-out outer-fold test predictions. Prevents the overoptimistic ROC AUC bias caused by whole-dataset feature filtering or balancing done outside the CV loop.

## When to use

Use when a small-n, high-predictor-count (P >> n) dataset needs unbiased model accuracy (e.g. ROC AUC) while feature selection or class balancing must be embedded inside the CV loop to avoid information leakage.

## When NOT to use

- The dataset is large enough for a conventional 2/3-1/3 train/test split without severe small-test-set inaccuracy — nested CV is motivated by biomedical datasets too small for this split
- The goal is feature ranking or biological interpretation rather than unbiased accuracy estimation of a final fitted model
- Predictor count is modest relative to sample size and no embedded feature selection or hyperparameter tuning is needed, making simple k-fold CV sufficient

## Inputs

- Predictor matrix x (rows = samples, columns = predictors; dataframes with factors can be one-hot encoded via one_hot())
- Outcome vector y (binary, or multinomial as in the iris 3-class example)
- Optional imbalanced binary dataset (e.g. 150 samples, 20,000 predictors, only first 30 predictors with weak signal) from the nestedcv vignette

## Outputs

- Concatenated outer-fold test predictions and unbiased ROC AUC (pROC) from nested CV
- Biased comparison AUC from whole-dataset filtering or unnested oversampling schemes
- Side-by-side comparison table (e.g. noise_simulation_auc.csv, imbalance_auc_comparison.csv)
- Fitted nestedcv object summarised via summary(res); final model coefficients via coef()

## How to apply

Load nestedcv and generate or load a predictor matrix X with outcome y (nestedcv stratifies outer and inner folds on y by default using caret's createFolds(), default 10 x 10 folds). For the biased comparison, rank all predictors by whole-dataset univariate filtering (e.g. ttest_filter with nfilter = 100) or apply balancing (e.g. randomsample oversampling) outside the CV loop, fit glmnet lasso, and compute ROC AUC with pROC on held-out data — this inflates apparent performance. For the unbiased scheme, call nestcv.glmnet with filterFUN embedded (e.g. ttest_filter) so the filter sees only training folds within the outer loop; inner CV (default 10-fold) tunes lambda and the elastic net alpha parameter, then the model fitted on the whole inner fold is tested on the left-out outer fold. Concatenate the unseen outer-fold test predictions against true outcomes and compute ROC AUC with pROC; tabulate schemes side by side (e.g. noise_simulation_auc.csv) to demonstrate that the nested scheme reports AUC near 0.50 on pure gaussian noise. Optionally parallelise the outer loop with cv.cores (mclapply on unix/mac, parLapply on Windows) and inspect tuning with plot_alphas(res) and plot_lambdas(res).

## Related tools

- **nestedcv** (Implements fully nested k x l-fold CV (nestcv.glmnet, nestcv.train) with embedded feature-selection filters inside the outer loop) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits lasso/elastic-net regularised models with inner-loop cross-validation of lambda and alpha) — https://CRAN.R-project.org/package=nestedcv
- **pROC** (Computes ROC AUC on concatenated outer-fold left-out test predictions)
- **caret** (Alternative framework used by nestcv.train to tune other models (e.g. ranger random forest) within nested CV) — https://CRAN.R-project.org/package=nestedcv
- **CORElearn** (Provides ReliefF for relieff_filter embedded within nested CV)

## Examples

```
res <- nestcv.glmnet(y, x, filterFUN = ttest_filter, filter_options = list(nfilter = 100), cv.cores = cores); summary(res)
```

## Evaluation signals

- On pure gaussian-noise data with P >> n, the fully nested scheme (filter restricted to training folds) reports ROC AUC close to 0.50 (Fig. 2A baseline), while whole-dataset t-test filtering yields substantially overoptimistic AUC
- Unnested randomsample oversampling applied outside the CV loop shows upward-biased apparent AUC relative to balance applied inside outer folds (imbalance_auc_comparison.csv)
- All filters (ttest_filter, wilcoxon_filter, anova_filter, correl_filter, rf_filter, relieff_filter) are demonstrably nested within the outer CV loop — no leakage from outer performance test sets
- summary(res) returns complete nested model-fitting information; outer-fold tuned lambda/alpha can be plotted per fold via plot(res$outer_result[[1]]$cvafit), plot_alphas(res), plot_lambdas(res)
- Outer and inner folds are stratified on the outcome y by default (caret createFolds), so class proportions are preserved across folds

## Limitations

- Biomedical datasets may be too small for a conventional 2/3-1/3 split, which is why nested CV concatenating outer-fold predictions is preferred over a single hold-out test set
- A final round of CV on the whole dataset is needed afterwards to determine hyperparameters for fitting the final model to external data — the nested result itself is a performance estimate, not a deployable model
- Parallelisation of the outer CV loop is platform-dependent: mclapply on unix/mac but parLapply on Windows (or via the future framework)
- Bootstrap-stability variants (boot_ttest) exist because univariate filter rankings can be unstable

## Evidence

- [abstract] Inner CV is used to tune models and outer CV is used to determine model performance without bias: "Inner CV is used to tune models and outer CV is used to determine model performance without bias"
- [abstract] Filters nested to avoid leakage: "the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] Whole-dataset filtering bias: "it has been
demonstrated that filtering on the whole dataset creates a bias when determining
accuracy of models (Vabalas et al, 2019)"
- [intro] Fold structure and prediction concatenation: "The inner fold CV, (default is 10-fold), is used to tune optimal
hyperparameters for models. Then the model is fitted on the whole inner fold and
tested on the left-out data from the outer fold."
- [readme] README installation/example: "In this example using iris dataset (multinomial, 3 classes), we fit a glmnet
model, tuning both lambda and alpha with 10 x 10-fold nested CV."
- [readme] README filter options: "Filters can be used by setting the `filterFUN` argument. Options for the filter 
function are passed as a list through `filter_options`."
