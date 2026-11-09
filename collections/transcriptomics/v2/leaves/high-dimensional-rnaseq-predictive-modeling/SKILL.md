---
name: high-dimensional-rnaseq-predictive-modeling
description: Use when the dataset has far more predictors than samples (P >> n, e.g.
  RNA-Seq like E-MTAB-11611), an unbiased estimate of model accuracy is required,
  and feature selection must be nested within CV to avoid leakage bias.
when_to_use_negative:
- When predictors are fewer than or comparable to samples and a conventional 2/3-1/3
  train/test split gives an adequately sized test set
- When feature selection has already been performed on the whole dataset and you only
  need to evaluate a fixed model — nested re-filtering is unnecessary, and prior whole-dataset
  filtering would still bias results
- When the target model is not available through glmnet or caret's prediction methods
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3318
tools:
- name: nestedcv
  role: Core package implementing nested k x l-fold CV with embedded filters; provides
    nestcv.glmnet and nestcv.train
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits lasso/elastic-net regularised models within nestcv.glmnet, with cross-validation
    of the alpha parameter
- name: caret
  role: Framework through which nestcv.train accesses many prediction methods, e.g.
    ranger random forest
- name: randomForest
  role: Backend for rf_filter variable-importance feature selection embedded in the
    outer CV loop
- name: ranger
  role: Random forest engine for nestcv.train (method = "ranger") and ranger_filter
    variable importance
- name: CORElearn
  role: Provides ReliefF via relieff_filter for embedded feature selection
- name: R
  role: Runtime environment for the nestedcv workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:63e4a2fdda9a6d5276c215a7b21b58b37b20f7e1f84bd8b88833ca38c03326f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CORElearn
  - R
  - caret
  - glmnet
  - nestedcv
  - randomForest
  - ranger
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# high-dimensional-rnaseq-predictive-modeling

## Summary

Build predictive models on high-dimensional transcriptomic data (P >> n) using the nestedcv R package, which performs fully nested k x l-fold cross-validation (default 10 x 10 folds) with feature-selection filters embedded inside the outer loop and inner-loop hyperparameter tuning via glmnet or caret.

## When to use

Use when the dataset has far more predictors than samples (P >> n, e.g. RNA-Seq like E-MTAB-11611), an unbiased estimate of model accuracy is required, and feature selection must be nested within CV to avoid leakage bias.

## When NOT to use

- When predictors are fewer than or comparable to samples and a conventional 2/3-1/3 train/test split gives an adequately sized test set
- When feature selection has already been performed on the whole dataset and you only need to evaluate a fixed model — nested re-filtering is unnecessary, and prior whole-dataset filtering would still bias results
- When the target model is not available through glmnet or caret's prediction methods

## Inputs

- R expression matrix of predictors (genes x samples) from a high-dimensional dataset such as E-MTAB-11611 R4RA rheumatoid arthritis RNA-Seq
- Outcome vector y (class labels or continuous response)
- Optional filter choice and options (filterFUN, filter_options), model family, and cv.cores setting

## Outputs

- Fitted nested CV model object (from nestcv.glmnet or nestcv.train) with outer_result per fold
- Concatenated left-out outer-fold predictions and unbiased performance metrics via summary()
- Final model coefficients via coef() and tuning diagnostics via plot_alphas()/plot_lambdas()

## How to apply

Load the expression matrix and outcome into R with nestedcv, then call nestcv.glmnet (tuning both lambda and elastic net alpha via inner CV) or nestcv.train for caret-backed engines such as random forest via ranger. One-hot encode factor predictors with one_hot() before glmnet, and optionally set filterFUN to an embedded filter (ttest_filter, wilcoxon_filter, anova_filter, correl_filter, rf_filter/ranger_filter, relieff_filter, glmnet_filter, stat_filter) so feature selection happens only on training data within the outer loop — filtering on the whole dataset biases accuracy estimates (Vabalas et al, 2019). nestedcv stratifies outer and inner folds on the outcome by default using caret's createFolds(); inner folds (default 10-fold) tune hyperparameters, the model is fitted on the inner training data and tested on the left-out outer fold, and outer test predictions are concatenated for accuracy measures. For expensive models such as random forest, parallelise the outer CV loop with cv.cores (mclapply on unix/mac, parLapply on Windows), sized near the physical core count. Finally call summary() for performance details, coef() for final-model coefficients, and plot_alphas()/plot_lambdas() to inspect glmnet tuning.

## Related tools

- **nestedcv** (Core package implementing nested k x l-fold CV with embedded filters; provides nestcv.glmnet and nestcv.train) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits lasso/elastic-net regularised models within nestcv.glmnet, with cross-validation of the alpha parameter)
- **caret** (Framework through which nestcv.train accesses many prediction methods, e.g. ranger random forest)
- **randomForest** (Backend for rf_filter variable-importance feature selection embedded in the outer CV loop)
- **ranger** (Random forest engine for nestcv.train (method = "ranger") and ranger_filter variable importance)
- **CORElearn** (Provides ReliefF via relieff_filter for embedded feature selection)
- **R** (Runtime environment for the nestedcv workflow)

## Examples

```
library(nestedcv); cores <- parallel::detectCores(logical = FALSE); res <- nestcv.glmnet(y, x, family = "multinomial", cv.cores = cores); summary(res)
```

## Evaluation signals

- summary() reports concatenated outer-fold test predictions against true labels, giving an unbiased accuracy measure that should be reported rather than inner-CV performance
- The filter must run inside the outer CV loop — check the fit specification so no feature selection touches the outer test folds (avoids leakage bias)
- glmnet fits should show cross-validated elastic net alpha as well as lambda per outer fold; plot(res$outer_result[[1]]$cvafit) inspects fold 1 tuning
- If cv.cores is used, wall-clock speedup should be positive and plateau near the number of physical cores (parallel::detectCores(logical = FALSE))
- With cv.cores = 8, expect benchmark speedups consistent with the reported maxima of up to 5.0x for nestcv.train random forest and up to 2.5x for nestcv.glmnet

## Limitations

- Parallelisation of the outer CV loop via mclapply works on unix/mac; Windows falls back to parLapply
- Biomedical datasets with P >> n may be too small for a conventional 2/3-1/3 split, which is why the whole dataset is used across nested folds rather than a single split
- Filtering predictors on the whole dataset before CV creates bias when determining model accuracy (Vabalas et al, 2019)
- README-reported speedups are upper bounds: up to 5.0x (nestcv.train random forest) and up to 2.5x (nestcv.glmnet); gains depend on the model and core count

## Evidence

- [abstract] Nested CV with embedded feature selection is suited to biomedical data where sample size is limited: "Nested CV with embedded feature selection is especially suited to biomedical data where the sample size is frequently limited, but the number of predictors may be significantly larger (P ≫ n)"
- [abstract] Filters nested within outer CV loop to avoid leakage: "the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] Default 10 x 10 folds and stratification: "`nestedcv` partitions the dataset into outer and inner folds (default 10 x 10
folds)."
- [intro] Filtering on whole dataset creates bias: "it has been
demonstrated that filtering on the whole dataset creates a bias when determining
accuracy of models (Vabalas et al, 2019)"
- [intro] Parallelisation of outer CV loop via cv.cores: "all allow parallelisation of the outer CV loop using `mclapply` from the
`parallel` package on unix/mac and `parLapply`"
- [readme] README: detect physical cores and fit nestcv.glmnet: "cores <- parallel::detectCores(logical = FALSE)  # detect physical cores

res <- nestcv.glmnet(y, x, family = "multinomial", cv.cores = cores)"
- [readme] README: nestcv.train with ranger: [evidence span withheld — anchor status: external_doc]
- [readme] README: filters passed via filterFUN and filter_options: "Filters can be used by setting the `filterFUN` argument. Options for the filter 
function are passed as a list through `filter_options`."
