---
name: feature-selection-precision-recall-analysis
description: Use when comparing feature-selection quality of penalized or latent-factor
  regression methods on simulated data where the true nonzero coefficients are known
  and you need precision/recall or ROC curves.
when_to_use_negative:
- Real experimental data (e.g., the aml, dlbcl, miller, nsclc, vantveer transcriptomic
  datasets) where the true feature support is unknown — precision/recall cannot be
  computed without ground truth
- When the analysis goal is only predictive accuracy (test MSE) and feature identity
  is not of interest
- Methods that do not produce a feature-selection or coefficient path (e.g., plain
  Ridge or the Oracle) cannot yield ROC curves over a regularization path
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3518
tools:
- name: suffpcr
  role: Fits SuffPCR via row-sparse PCA followed by OLS or logistic regression; its
    feature selection is evaluated with precision/recall and ROC against competitors
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment for generating simulation data, fitting all methods, and
    computing selection metrics
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub before running the benchmark
  repo: https://github.com/dajmcdon/suffpcr
- name: fps
  role: Dependency providing the FPS-PCR competitor in the feature-selection comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:e664e613cd275a8733c944f89adf1c66840761464ab652f82d5fa750de76b0ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - fps
  - remotes
  - suffpcr
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# feature-selection-precision-recall-analysis

## Summary

Evaluates how well high-dimensional regression methods (SuffPCR, Lasso, Elastic Net, SPC, AIMER) recover true features by recording selected features, computing precision/recall, and tracing ROC curves over regularization paths on synthetic data with known support.

## When to use

Use when comparing feature-selection quality of penalized or latent-factor regression methods on simulated data where the true nonzero coefficients are known and you need precision/recall or ROC curves.

## When NOT to use

- Real experimental data (e.g., the aml, dlbcl, miller, nsclc, vantveer transcriptomic datasets) where the true feature support is unknown — precision/recall cannot be computed without ground truth
- When the analysis goal is only predictive accuracy (test MSE) and feature identity is not of interest
- Methods that do not produce a feature-selection or coefficient path (e.g., plain Ridge or the Oracle) cannot yield ROC curves over a regularization path

## Inputs

- Simulated design matrix X with known latent factor structure and true coefficient support (e.g., n=100, p=1000, rank-1 factor, 15 true features)
- Response vector y generated as U + noise at a specified SNR (e.g., 5/5)
- Fitted models from suffPCR, glmnet (Lasso, Ridge, Elastic Net), SPC, AIMER, ISPCA, FPS-PCR, and an Oracle estimator with their coefficient/selection paths

## Outputs

- Per-method precision and recall of feature selection on an independent test set
- Feature-selection ROC curves over regularization paths (roc_curves.png)
- Aggregated metrics table including test-set MSE and selection precision/recall per method (simulation_results.csv)
- Saved simulation dataset (sim_data.rds) for reproducibility

## How to apply

Generate synthetic data with a known coefficient support (e.g., Algorithm 3 structure: n=100, p=1000, rank-1 latent factor X = 5*tcrossprod(U,V) + 0.1*noise, unit loading V with r=5 nonzero entries, y = U + noise, 15 true features of which only 10 have marginal correlation) and save it (e.g., sim_data.rds). Fit each method — suffPCR(X, y, d=1:3) plus Oracle, Lasso, Ridge, Elastic Net (glmnet), SPC, AIMER, ISPCA, FPS-PCR — tuning penalized methods on a held-out validation set. For each fitted model, extract the set of selected features and compute precision (fraction selected that are true) and recall (fraction true features recovered) on an independent test set. Additionally compute feature-selection ROC curves by sweeping each method's regularization path and thresholding coefficient magnitudes, plotting ROC for SuffPCR, Lasso, Elastic Net, SPC, and AIMER. Aggregate metrics into a results table (simulation_results.csv) and export the ROC comparison (e.g., roc_curves.png); results are complete when every method has test MSE and precision/recall entries and SuffPCR matches or improves the reported performance of nearly optimal prediction with correct feature recovery.

## Related tools

- **suffpcr** (Fits SuffPCR via row-sparse PCA followed by OLS or logistic regression; its feature selection is evaluated with precision/recall and ROC against competitors) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment for generating simulation data, fitting all methods, and computing selection metrics) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub before running the benchmark) — https://github.com/dajmcdon/suffpcr
- **fps** (Dependency providing the FPS-PCR competitor in the feature-selection comparison)

## Examples

```
library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- Every method in the comparison table has both a test-set MSE and a precision/recall entry — no missing cells
- SuffPCR matches or improves the reported simulation behavior: nearly optimal prediction while generally selecting the correct features
- In the adversarial (zero-marginal-correlation) setting, SuffPCR achieves smaller test MSE than the Oracle and all sparse competitors
- SuffPCR and AIMER attain a (near) perfect ROC curve while Lasso, Elastic Net, and SPC fail to identify the five features with zero marginal correlation but nonzero coefficients
- Selection precision and recall are computed against the known 15-feature ground-truth support on an independent test set, not the training data

## Limitations

- Lasso, Elastic Net, and SPC cannot identify features with zero marginal correlation but nonzero coefficients — ROC-based comparison is only meaningful for methods with a selection path (SuffPCR, AIMER, and the penalized methods)
- Feature-recovery metrics require simulated data with known support; they cannot be computed for the experimental transcriptomic datasets
- Penalized competitors must be tuned on a held-out validation set for the comparison to be fair; untuned paths bias precision/recall

## Evidence

- [results] In this adversarial setting, SuffPCR outperforms all alternatives including the Oracle estimator (smaller test MSE while generally selecting the correct features): [evidence span withheld — anchor status: not_found]
- [results] SuffPCR and AIMER achieve a perfect ROC curve while Lasso, Elastic Net, and SPC cannot identify five features: [evidence span withheld — anchor status: not_found]
- [readme] The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression.: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic
regression."
- [methods] out <- suffPCR(X, y, d=1:3): [evidence span withheld — anchor status: not_found]
