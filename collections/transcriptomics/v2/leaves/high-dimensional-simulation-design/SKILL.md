---
name: high-dimensional-simulation-design
description: Use when evaluating whether a sparse supervised dimension-reduction method
  still predicts well and recovers features when the marginal-correlation screening
  assumption is deliberately violated (zero-marginal-correlation, non-zero-coefficient
  features).
when_to_use_negative:
- The data are real experimental transcriptomic data (use the suffpcr aml/dlbcl/miller/nsclc/vantveer
  data applications instead of the synthetic benchmark).
- The goal is to benchmark screening methods under their favorable assumption, i.e.,
  features with strong marginal correlation to the response — this construction is
  designed to violate marginal-correlation screening.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3517
tools:
- name: suffpcr
  role: R package providing suffPCR() for row-sparse PCA followed by OLS or logistic
    regression on the recovered subspace
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub via remotes::install_github
  repo: https://github.com/dajmcdon/suffpcr
- name: fps
  role: Dependency providing FPS-PCR competitor for the benchmark comparison
- name: R
  role: Runtime environment in which the simulation, competitor fits, and evaluation
    are executed
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:f98a5816377f537e063e6c1a6b6c36604b714821423d4eb9575cedc0f09da901
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
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# high-dimensional-simulation-design

## Summary

Design and run an adversarial high-dimensional synthetic benchmark (n=100, p=1000, rank-1 latent factor structure) in which features have zero marginal correlation but non-zero coefficients, then compare SuffPCR against penalized and sparse-PCA competitors on test MSE, feature selection precision/recall, and regularization-path ROC curves.

## When to use

Use when evaluating whether a sparse supervised dimension-reduction method still predicts well and recovers features when the marginal-correlation screening assumption is deliberately violated (zero-marginal-correlation, non-zero-coefficient features).

## When NOT to use

- The data are real experimental transcriptomic data (use the suffpcr aml/dlbcl/miller/nsclc/vantveer data applications instead of the synthetic benchmark).
- The goal is to benchmark screening methods under their favorable assumption, i.e., features with strong marginal correlation to the response — this construction is designed to violate marginal-correlation screening.

## Inputs

- Simulated design matrix X (n=100, p=1000) with rank-1 latent factor structure (spike 5, noise 0.1)
- Unit loading vector V with r=5 nonzero entries
- Simulated response y = U + noise at SNR 5/5
- Independent test set generated from the same latent factor model

## Outputs

- simulation_results.csv with per-method test-set MSE and precision/recall of feature selection
- roc_curves.png comparing feature-selection ROC over regularization paths for SuffPCR, Lasso, Elastic Net, SPC, and AIMER
- sim_data.rds holding the saved synthetic dataset

## How to apply

Generate data from the Algorithm 3 construction: a rank-1 latent factor X = 5 * tcrossprod(U, V) + 0.1 * matrix(rnorm(n*p), n) with n=100, p=1000, a unit loading vector V with r=5 non-zero entries, response y = U + noise at SNR 5/5, and 15 true features of which only 10 carry marginal correlation. Fit suffPCR(X, y, d=1:3) alongside competitors Oracle, Lasso, Ridge, Elastic Net (glmnet), SPC, AIMER, ISPCA, and FPS-PCR (fps dependency), tuning every penalized method on a held-out validation set. Evaluate on an independent test set recording test MSE, selected features, and precision/recall of feature selection. Compute feature-selection ROC curves over the regularization paths for SuffPCR, Lasso, Elastic Net, SPC, and AIMER. Aggregate metrics to simulation_results.csv and ROC comparison to roc_curves.png.

## Related tools

- **suffpcr** (R package providing suffPCR() for row-sparse PCA followed by OLS or logistic regression on the recovered subspace) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub via remotes::install_github) — https://github.com/dajmcdon/suffpcr
- **fps** (Dependency providing FPS-PCR competitor for the benchmark comparison)
- **R** (Runtime environment in which the simulation, competitor fits, and evaluation are executed) — https://github.com/dajmcdon/suffpcr

## Examples

```
library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- Every method in the results table has a test-set MSE and a precision/recall entry for feature selection.
- SuffPCR matches or improves the reported simulation performance: nearly optimal prediction with correct feature recovery relative to sparse competitors.
- In the adversarial setting, SuffPCR outperforms all alternatives including the Oracle estimator (smaller test MSE while generally selecting the correct features).
- SuffPCR and AIMER achieve a perfect ROC curve while Lasso, Elastic Net, and SPC fail to identify the five zero-marginal-correlation features.
- The generated design matches the construction: rank-1 factor with spike 5 and noise 0.1, V unit-normalized with 5 non-zero entries, and only 10 of 15 true features carrying marginal correlation.

## Limitations

- The construction is adversarial by design: it targets methods that rely on marginal-correlation screening (Lasso, Elastic Net, SPC), so those methods cannot identify five features in this setting.
- Near-optimal SuffPCR performance holds when model assumptions are satisfied; the article's guarantees apply under the latent factor model assumptions.
- The benchmark requires a holdout validation set for tuning every penalized method and an independent test set, so sample size per split is limited (n=100 total in the reported design).

## Evidence

- [intro] The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression.: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [readme] README example construction: "X <- 5 * tcrossprod(U, V) + 0.1 * matrix(rnorm(n * p), n); y <- U + rnorm(n); out <- suffPCR(X, y, d=1:3)"
- [methods] Install from GitHub: "remotes::install_github("dajmcdon/suffpcr")"
- [abstract] Performance claim: "SuffPCR works well on a variety of simulated and experimental transcriptomic data, performing nearly optimally when the model assumptions are satisfied."
- [abstract] Competitor failure mode: "These methods can fail to select the correct features, predict poorly relative to non-sparse alternatives or ignore any unknown grouping structures for the features."
