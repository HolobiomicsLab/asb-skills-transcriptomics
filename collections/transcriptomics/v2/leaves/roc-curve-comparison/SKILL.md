---
name: roc-curve-comparison
description: Use when you need to verify that a sparse regression method selects the
  correct features, by comparing feature-selection ROC curves over regularization
  paths against competitors on simulated data with known ground truth.
when_to_use_negative:
- The goal is purely predictive performance (test MSE) rather than feature recovery
  — ROC over selection paths is unnecessary
- No ground-truth set of true nonzero features is known (e.g. real transcriptomic
  data without simulated coefficients), so TPR/FPR cannot be defined
- Comparing only methods without a feature-selection path (Oracle, Ridge, ISPCA, FPS-PCR),
  for which a selection ROC curve is not defined
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: suffpcr
  role: Provides suffPCR() whose row-sparse PCA regularization path supplies the primary
    ROC curve and feature selection under comparison
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub before running the comparison
  repo: https://github.com/dajmcdon/suffpcr
- name: fps
  role: Dependency for the FPS-PCR competitor included in the benchmark's MSE/precision-recall
    table
- name: R
  role: Runtime environment for the suffpcr package, glmnet competitors, and ROC computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:6e7e4d692244fd65123637c0a0207d76eacbe2c89d8dd8d6ce05eb0e4716bf5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - fps
  - remotes
  - suffpcr
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# roc-curve-comparison

## Summary

Builds and compares feature-selection ROC curves for sparse high-dimensional regression methods (SuffPCR, Lasso, Elastic Net, SPC, AIMER) computed over their regularization paths, to test whether each method recovers the true nonzero features in a synthetic design where only some true features carry marginal correlation.

## When to use

Use when you need to verify that a sparse regression method selects the correct features, by comparing feature-selection ROC curves over regularization paths against competitors on simulated data with known ground truth.

## When NOT to use

- The goal is purely predictive performance (test MSE) rather than feature recovery — ROC over selection paths is unnecessary
- No ground-truth set of true nonzero features is known (e.g. real transcriptomic data without simulated coefficients), so TPR/FPR cannot be defined
- Comparing only methods without a feature-selection path (Oracle, Ridge, ISPCA, FPS-PCR), for which a selection ROC curve is not defined

## Inputs

- Synthetic design matrix X from Algorithm 3: n=100, p=1000, rank-1 latent factor (spike 5, noise 0.1), saved as sim_data.rds
- Response vector y = U + noise with SNR 5/5
- Ground-truth feature set: 15 true features, 10 of which carry marginal correlation
- Regularization paths (selected-feature sequences) from SuffPCR, Lasso, Elastic Net, SPC, and AIMER

## Outputs

- roc_curves.png — overlaid feature-selection ROC curves for SuffPCR, Lasso, Elastic Net, SPC, and AIMER
- simulation_results.csv — per-method test-set MSE, selected features, and precision/recall of feature selection

## How to apply

Generate the Algorithm 3 adversarial synthetic design (n=100, p=1000, rank-1 latent factor X = 5*tcrossprod(U,V) + 0.1*noise, unit loading V with 5 nonzero entries, y = U + noise, SNR 5/5, 15 true features of which only 10 carry marginal correlation) and save it as sim_data.rds. Fit suffPCR(X, y, d=1:3) plus the competitors Oracle, Lasso, Ridge, Elastic Net (glmnet), SPC, AIMER, ISPCA and FPS-PCR, tuning every penalized method on a held-out validation set. Sweep each penalized method's regularization path, and at every path point score the selected features against the known 15 true features to compute TPR/FPR pairs for the ROC. Restrict the ROC comparison to methods with a selection path (SuffPCR, Lasso, Elastic Net, SPC, AIMER); record test-set MSE, selected features, and precision/recall in simulation_results.csv and the ROC overlay in roc_curves.png. The adversarial design is deliberate: features with zero marginal correlation but nonzero coefficients break SPC's marginal-correlation screening, so correct feature recovery here is the discriminating criterion.

## Related tools

- **suffpcr** (Provides suffPCR() whose row-sparse PCA regularization path supplies the primary ROC curve and feature selection under comparison) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub before running the comparison) — https://github.com/dajmcdon/suffpcr
- **fps** (Dependency for the FPS-PCR competitor included in the benchmark's MSE/precision-recall table)
- **R** (Runtime environment for the suffpcr package, glmnet competitors, and ROC computation)

## Examples

```
library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- In the adversarial zero-marginal-correlation design, SuffPCR and AIMER should attain a perfect ROC curve (TPR=1 at FPR=0)
- Lasso, Elastic Net, and SPC should fail to identify the five features lacking marginal correlation, visibly degrading their ROC curves
- simulation_results.csv is complete only when every method has a test-set MSE and a precision/recall entry
- SuffPCR's test MSE should match or improve the reported simulation performance (outperforming even the Oracle estimator)
- Selected-feature sets from the path should coincide with the 15 true features for SuffPCR (precision and recall near 1)

## Limitations

- The ROC comparison requires simulated data with known true coefficients; it cannot be computed on the real transcriptomic datasets (aml, dlbcl, miller, nsclc, vantveer) shipped with the package
- The synthetic setting is explicitly adversarial to marginal-correlation screening (SPC), so absolute ROC levels do not transfer to settings where marginal correlation is informative
- Only methods with a regularization/selection path (SuffPCR, Lasso, Elastic Net, SPC, AIMER) enter the ROC comparison; Oracle, Ridge, ISPCA, and FPS-PCR appear only in the MSE/precision-recall table
- No changelog found for the repository, so version drift in ROC-reproducing code cannot be tracked

## Evidence

- [full_text] SuffPCR and AIMER achieve a perfect ROC curve while Lasso, Elastic Net, and SPC cannot identify five features: [evidence span withheld — anchor status: not_found]
- [abstract] SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace: "SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace."
- [methods] run suffPCR on simulated data: [evidence span withheld — anchor status: not_found]
- [readme] README example showing the synthetic rank-1 design and suffPCR call: "X <- 5 * tcrossprod(U, V) + 0.1 * matrix(rnorm(n * p), n)
y <- U + rnorm(n)
out <- suffPCR(X, y, d=1:3)"
- [intro] row-sparse PCA followed by ordinary least squares or logistic regression: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
