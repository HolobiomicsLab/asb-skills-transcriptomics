---
name: lasso-logistic-regression-fitting
description: Use when fitting lasso or elastic-net logistic regression to binary outcome
  data with predictors far outnumbering samples (P ≫ n, e.g. 150 samples × 20,000
  predictors), and hyperparameter tuning must be nested within CV to avoid leakage.
when_to_use_negative:
- The predictors matrix does not exist yet or the task is raw data pre-processing
  (this skill assumes an already-assembled x/y or data.frame ready for model fitting)
- The analysis requires non-glmnet machine learning models that fit better through
  nestcv.train/caret rather than the glmnet regularized linear model framework
- Univariate feature-selection filters (t-test, Wilcoxon, ANOVA, ReliefF) are not
  needed and only a plain glmnet::cv.glmnet fit without nested outer-loop performance
  estimation is required
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: nestedcv
  role: Implements the fully nested k × l-fold CV wrapper around glmnet (nestcv.glmnet)
    including balance argument for fold-internal oversampling
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the lasso/elastic-net regularized logistic regression models whose lambda
    and alpha are cross-validated within the nested loop
- name: pROC
  role: Computes ROC AUC from concatenated outer-fold test predictions to quantify
    performance (and leakage-induced upward bias)
- name: R
  role: Language in which the whole nested CV workflow (nestcv.glmnet, pROC, randomsample)
    is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:774d3286bfe67ef9132eb6e54dc241c486cbeefbdffc66de210a1b569e2bc058
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  - pROC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lasso-logistic-regression-fitting

## Summary

Fits lasso (and elastic-net) logistic regression models on high-dimensional binary outcomes using the nestedcv package, with lambda (and optionally alpha) tuned via inner CV while outer CV folds provide unbiased performance estimates. This prevents the optimistic bias that arises when feature selection or data balancing (e.g. randomsample oversampling) is applied outside the CV loops.

## When to use

Use when fitting lasso or elastic-net logistic regression to binary outcome data with predictors far outnumbering samples (P ≫ n, e.g. 150 samples × 20,000 predictors), and hyperparameter tuning must be nested within CV to avoid leakage.

## When NOT to use

- The predictors matrix does not exist yet or the task is raw data pre-processing (this skill assumes an already-assembled x/y or data.frame ready for model fitting)
- The analysis requires non-glmnet machine learning models that fit better through nestcv.train/caret rather than the glmnet regularized linear model framework
- Univariate feature-selection filters (t-test, Wilcoxon, ANOVA, ReliefF) are not needed and only a plain glmnet::cv.glmnet fit without nested outer-loop performance estimation is required

## Inputs

- Binary outcome vector y (e.g. imbalanced two-class labels from a simulated nestedcv vignette dataset)
- Numeric predictor matrix x (e.g. 150 samples × 20,000 predictors with weak signal in the first 30 predictors)
- Optional outer/inner fold indices (nestedcv defaults to 10 x 10 caret createFolds() stratified folds)
- Optional data.frame with factor/character predictors for one_hot() encoding

## Outputs

- nestcv.glmnet result object (inspectable with summary() and coef() for final fitted coefficients)
- Concatenated outer-fold test predictions for unbiased accuracy metrics
- ROC AUC values (e.g. computed with pROC) tabulated side by side, such as imbalance_auc_comparison.csv
- Tuning diagnostics: alpha/lambda plots via plot_alphas(res) and plot_lambdas(res)

## How to apply

Build a predictor matrix (one-hot encoding factor/character columns with one_hot() where needed) and a binary outcome vector, then call nestcv.glmnet with family='binomial'; nestedcv partitions the data into 10 x 10 stratified outer/inner folds (stratification uses caret's createFolds(), custom fold indices may be supplied). Inner CV tunes lambda (and elastic-net alpha); the model fitted on each inner fold is tested on the left-out outer fold, and outer test-fold predictions are concatenated for unbiased accuracy measures such as ROC AUC (computed with pROC). If classes are imbalanced, apply randomsample oversampling only to training data inside the outer CV folds via the balance argument — applying it to the whole dataset before nestcv.glmnet leaks test information and inflates apparent AUC. Parallelise the outer loop with cv.cores (mclapply on unix/mac, parLapply on Windows, or the future framework), inspect tuning with plot_alphas()/plot_lambdas(), and after CV run a final round on the whole dataset to fit the final model for external prediction.

## Related tools

- **nestedcv** (Implements the fully nested k × l-fold CV wrapper around glmnet (nestcv.glmnet) including balance argument for fold-internal oversampling) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the lasso/elastic-net regularized logistic regression models whose lambda and alpha are cross-validated within the nested loop)
- **pROC** (Computes ROC AUC from concatenated outer-fold test predictions to quantify performance (and leakage-induced upward bias))
- **R** (Language in which the whole nested CV workflow (nestcv.glmnet, pROC, randomsample) is executed)

## Examples

```
library(nestedcv); res <- nestcv.glmnet(y, x, family = "binomial", balance = TRUE, cv.cores = cores); summary(res)
```

## Evaluation signals

- Nested CV AUC computed from concatenated outer-fold predictions should be lower than the AUC from a run where oversampling/SMOTE was applied outside the CV loop (the unnested value is the upwardly biased one)
- summary(res) reports per-outer-fold results and converged tuning; coef(res) returns a sparse coefficient vector consistent with lasso sparsity
- plot_alphas(res) and plot_lambdas(res) show how deviance varies across alpha/lambda, confirming inner-CV tuning actually occurred for each outer fold
- Fold construction is stratified on the outcome by default (caret createFolds()); verify class balance in each outer test fold for imbalanced data

## Limitations

- If class balancing (randomsample oversampling or SMOTE) is applied to the whole dataset before nestcv.glmnet, data leakage causes upward bias in apparent performance; balancing must be done inside the outer folds via the balance argument
- Biomedical datasets with P ≫ n are typically too small for a conventional 2/3-1/3 split, so per-fold test-set estimates are noisy — nested CV over all folds is used instead of a single split
- No changelog is available in the synthesized source document, and the discussion section text is absent from the source material

## Evidence

- [abstract] abstract quote on nested k×l-fold implementation: "The nestedcv R package implements fully nested k × l-fold CV for lasso and elastic-net regularized linear models"
- [intro] intro quote on glmnet alpha cross-validation: "Nested cross-validation (CV) for the glmnet and caret packages. With glmnet this
includes cross-validation of elastic net alpha parameter."
- [intro] intro quote on outer-fold prediction concatenation: "the unseen test predictions from the outer folds are compared against the true results for the outer test folds and the results concatenated"
- [readme] readme example invocation of nestcv.glmnet: "res <- nestcv.glmnet(y, x, family = "multinomial", cv.cores = cores)
summary(res)"
- [readme] readme quote on tuning diagnostics plots: "Output from the nested CV with glmnet can be plotted to show how deviance is 
affected by alpha and lambda."
