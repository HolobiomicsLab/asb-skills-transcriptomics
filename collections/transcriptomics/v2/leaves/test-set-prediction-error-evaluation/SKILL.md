---
name: test-set-prediction-error-evaluation
description: Use when comparing penalized or latent-factor regression methods in a
  p >> n setting and you need unbiased out-of-sample generalization error (test MSE)
  plus feature-recovery quality on a synthetic or held-out benchmark.
when_to_use_negative:
- The goal is only in-sample fit or training-set residual diagnostics — this skill
  requires an independent test set disjoint from both training and validation data.
- The outcome is unsupervised (no response y), so prediction error cannot be computed.
- There is no ground-truth feature support, making precision/recall and feature-selection
  ROC computation impossible; restrict to test MSE only.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3577
tools:
- name: suffpcr
  role: Primary method under evaluation; fits row-sparse PCA followed by OLS/logistic
    regression via suffPCR(X, y, d=1:3)
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment for generating the synthetic benchmark, fitting all models,
    and computing test MSE, precision/recall, and ROC curves
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub before running the benchmark
  repo: https://github.com/dajmcdon/suffpcr
- name: glmnet
  role: Fits the Lasso, Ridge, and Elastic Net competitors over their regularization
    paths for test-set comparison
- name: fps
  role: Dependency providing the FPS-PCR competitor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:c1fc92eef42044d027fb956ac5f2b3dc6de811ccbc4e406e528c7cc1c787cea8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - fps
  - glmnet
  - remotes
  - suffpcr
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# test-set-prediction-error-evaluation

## Summary

Evaluate high-dimensional regression methods (SuffPCR vs. Lasso, Elastic Net, Ridge, SPC, AIMER, ISPCA, FPS-PCR, Oracle) on an independent test set by recording test MSE alongside feature-selection precision/recall and ROC curves, mirroring the paper's synthetic benchmark (Algorithm 3 data, n=100, p=1000).

## When to use

Use when comparing penalized or latent-factor regression methods in a p >> n setting and you need unbiased out-of-sample generalization error (test MSE) plus feature-recovery quality on a synthetic or held-out benchmark.

## When NOT to use

- The goal is only in-sample fit or training-set residual diagnostics — this skill requires an independent test set disjoint from both training and validation data.
- The outcome is unsupervised (no response y), so prediction error cannot be computed.
- There is no ground-truth feature support, making precision/recall and feature-selection ROC computation impossible; restrict to test MSE only.

## Inputs

- Training/validation/test split of the Algorithm 3 synthetic design: n=100 samples, p=1000 features, rank-1 latent factor structure X = 5 * tcrossprod(U, V) + 0.1 * matrix(rnorm(n*p), n), unit loading V with 5 nonzero entries
- Continuous response y = U + noise at SNR 5/5, with 15 true features of which only 10 carry marginal correlation
- Fitted model objects from suffPCR(X, y, d=1:3) and each competitor, tuned on the held-out validation set

## Outputs

- simulation_results.csv with per-method test-set MSE and feature-selection precision/recall
- roc_curves.png comparing feature-selection ROC curves for SuffPCR, Lasso, Elastic Net, SPC, and AIMER over the regularization paths
- Saved dataset sim_data.rds for reproducibility

## How to apply

Split the data so that every penalized method is tuned on a held-out validation set and final models are scored only once on an independent test set, which prevents tuning leakage. Fit each competitor with its native engine (glmnet for Lasso/Ridge/Elastic Net, suffpcr for SuffPCR, plus SPC, AIMER, ISPCA, FPS-PCR) over its regularization path on the training data. Record test-set MSE for every method and, for sparse methods, the set of selected features compared with the ground-truth nonzero coefficients to compute precision/recall. Sweep the regularization paths to trace feature-selection ROC curves, which reveals methods that cannot identify features with zero marginal correlation but nonzero coefficients (Lasso, Elastic Net, SPC fail here while SuffPCR and AIMER achieve a perfect ROC). Write aggregated metrics (e.g. simulation_results.csv) and ROC plots (e.g. roc_curves.png); the benchmark is complete when every method has a test MSE and precision/recall entry.

## Related tools

- **suffpcr** (Primary method under evaluation; fits row-sparse PCA followed by OLS/logistic regression via suffPCR(X, y, d=1:3)) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment for generating the synthetic benchmark, fitting all models, and computing test MSE, precision/recall, and ROC curves) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub before running the benchmark) — https://github.com/dajmcdon/suffpcr
- **glmnet** (Fits the Lasso, Ridge, and Elastic Net competitors over their regularization paths for test-set comparison)
- **fps** (Dependency providing the FPS-PCR competitor)

## Examples

```
remotes::install_github("dajmcdon/suffpcr"); library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- Every evaluated method (SuffPCR, Oracle, Lasso, Ridge, Elastic Net, SPC, AIMER, ISPCA, FPS-PCR) has a non-empty test-set MSE entry in simulation_results.csv.
- Every sparse method has a precision/recall entry for feature selection against the 15 true features.
- SuffPCR matches or improves the reported simulation performance: smaller test MSE than all alternatives including the Oracle, while generally selecting the correct features.
- In the adversarial setting (5 features with zero marginal correlation but nonzero coefficients), Lasso, Elastic Net, and SPC fail to identify five of the features, while SuffPCR and AIMER achieve a perfect ROC curve.
- Test-set predictions are computed on data never used for fitting or validation tuning, so test MSE is not optimistically biased relative to validation error.

## Limitations

- The benchmark setting is adversarial by construction (rank-1 latent factors, features with zero marginal correlation but nonzero coefficients), so absolute error levels may not transfer to other data-generating regimes.
- Results depend on the simulation seed and the rank choices (d=1:3) supplied to suffPCR; different latent ranks or SNR may change the ranking of methods.
- ROC curves are only computable for methods with a regularization path yielding graded feature inclusion; methods without sparsity patterns cannot be placed on the ROC comparison.

## Evidence

- [results] Adversarial setting advantage of SuffPCR over all alternatives: [evidence span withheld — anchor status: not_found]
- [results] ROC failure of marginal-correlation screeners: [evidence span withheld — anchor status: not_found]
- [readme] Method definition grounding the fitting step: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [readme] Synthetic-data generation and fitting pattern from the README example: "X <- 5 * tcrossprod(U, V) + 0.1 * matrix(rnorm(n * p), n)
y <- U + rnorm(n)
out <- suffPCR(X, y, d=1:3)"
- [methods] Installation path required before running the benchmark: "remotes::install_github("dajmcdon/suffpcr")"
