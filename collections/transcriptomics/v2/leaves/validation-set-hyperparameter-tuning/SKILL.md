---
name: validation-set-hyperparameter-tuning
description: Use when comparing multiple penalized high-dimensional regression methods
  (e.g. suffpcr vs glmnet competitors) so each is tuned identically on held-out validation
  data before test-set evaluation.
when_to_use_negative:
- When only a single method is being fit and no cross-method comparison is intended
  — a single train/test split without a dedicated validation set avoids wasting samples
- When the response or feature structure differs from the linear/logistic high-dimensional
  regression setting SuffPCR targets (row-sparse PCA followed by OLS or logistic regression)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3524
tools:
- name: suffpcr
  role: Provides suffPCR() whose penalty and latent-dimension d=1:3 are tuned on the
    validation set alongside competitors
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment for generating synthetic data, fitting models, and computing
    validation/test metrics
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub before tuning runs
  repo: https://github.com/dajmcdon/suffpcr
- name: fps
  role: Dependency for the FPS-PCR competitor included in the tuned comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:008931061f37cc5937701c64bd97db0abeb4a42d40277c5ee02f92ec1b57b7e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - fps
  - remotes
  - suffpcr
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# Validation-set hyperparameter tuning

## Summary

Tunes every penalized regression method (SuffPCR, Lasso, Ridge, Elastic Net, SPC, AIMER, ISPCA, FPS-PCR) on a held-out validation set before scoring each on an independent test set, enabling fair prediction-error (test MSE) and feature-selection (precision/recall, ROC) comparisons across regularization paths.

## When to use

Use when comparing multiple penalized high-dimensional regression methods (e.g. suffpcr vs glmnet competitors) so each is tuned identically on held-out validation data before test-set evaluation.

## When NOT to use

- When only a single method is being fit and no cross-method comparison is intended — a single train/test split without a dedicated validation set avoids wasting samples
- When the response or feature structure differs from the linear/logistic high-dimensional regression setting SuffPCR targets (row-sparse PCA followed by OLS or logistic regression)

## Inputs

- Simulated design matrix X (e.g. Algorithm 3 synthetic data with n=100, p=1000, rank-1 latent factor structure)
- Response vector y = U + noise at SNR 5/5 with known true feature set (15 features, 10 marginally correlated)
- Fitted model objects for each penalized method along their regularization paths

## Outputs

- Tuned model per method (hyperparameters selected on validation set)
- Test-set MSE per method
- Selected-feature lists with precision/recall of feature recovery
- Feature-selection ROC curves over regularization paths (roc_curves.png)
- Aggregated metrics table (simulation_results.csv)

## How to apply

Split the simulated data (e.g. the Algorithm 3 synthetic benchmark with n=100, p=1000, rank-1 latent factor X = 5 * tcrossprod(U, V) + 0.1 noise, SNR 5/5) into training, validation, and independent test partitions. For each penalized method — SuffPCR fitted via suffPCR(X, y, d=1:3), plus Lasso, Ridge, and Elastic Net from glmnet, SPC, AIMER, ISPCA, and FPS-PCR — sweep its regularization path and select the hyperparameters that minimize held-out validation error. Apply the chosen model untouched to the test set and record test-set MSE, the selected features, and precision/recall of feature recovery. For ROC analysis, compute feature-selection ROC curves for SuffPCR, Lasso, Elastic Net, SPC, and AIMER across the full regularization paths rather than at a single tuned point. Aggregate all metrics (simulation_results.csv, roc_curves.png) so every method has a complete MSE and precision/recall entry.

## Related tools

- **suffpcr** (Provides suffPCR() whose penalty and latent-dimension d=1:3 are tuned on the validation set alongside competitors) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment for generating synthetic data, fitting models, and computing validation/test metrics) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub before tuning runs) — https://github.com/dajmcdon/suffpcr
- **fps** (Dependency for the FPS-PCR competitor included in the tuned comparison)

## Examples

```
out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- Every method in the comparison table has both a test-set MSE and a precision/recall entry for feature selection
- SuffPCR matches or improves the reported simulation performance: nearly optimal prediction with correct feature recovery relative to sparse competitors
- In the adversarial zero-marginal-correlation setting, SuffPCR should outperform all alternatives including the Oracle estimator on test MSE
- SuffPCR and AIMER should achieve a perfect ROC curve while Lasso, Elastic Net, and SPC fail to identify the five zero-marginal-correlation features
- Tuned hyperparameters are chosen purely from validation-set error, never from the test set used for final scoring

## Limitations

- Competitors like Lasso, Elastic Net, and SPC rely on marginal-correlation screening and cannot identify features with zero marginal correlation but non-zero coefficients in the Algorithm 3 design
- Tuning on a held-out validation set consumes data that would otherwise be available for training, which matters at small n (e.g. n=100)

## Evidence

- [methods] tuning every penalized method on a held-out validation set: [evidence span withheld — anchor status: not_found]
- [methods] Evaluate all fitted models on an independent test set: [evidence span withheld — anchor status: not_found]
- [methods] ROC curves computed over regularization paths: [evidence span withheld — anchor status: not_found]
- [abstract] SuffPCR performs nearly optimally when assumptions hold: "SuffPCR works well on a variety of simulated and experimental transcriptomic data, performing nearly optimally when the model assumptions are satisfied."
- [readme] README example invocation pattern: "out <- suffPCR(X, y, d=1:3)"
