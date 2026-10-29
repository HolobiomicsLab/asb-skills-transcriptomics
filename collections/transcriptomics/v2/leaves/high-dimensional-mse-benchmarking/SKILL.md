---
name: high-dimensional-mse-benchmarking
description: Use when comparing prediction methods (e.g. SuffPCR vs sparse linear
  models vs non-sparse PCR) on high-dimensional omics data where both out-of-sample
  MSE and correct feature selection must be evaluated.
when_to_use_negative:
- When the response is real rather than semi-synthetic and no ground-truth sparse
  signal direction exists, feature-selection overlap cannot be computed and only MSE
  is meaningful.
- When the goal is low-dimensional regression with few, weakly correlated features
  — the method targets high-dimensional omics with correlated features.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: suffpcr
  role: Fits sufficient principal component regression via row-sparse PCA followed
    by OLS or logistic regression across d=1:3; provides the nsclc dataset
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment for the semi-synthetic benchmark analysis and figure rendering
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub before benchmarking
  repo: https://github.com/dajmcdon/suffpcr
- name: stats
  role: Base R tools for PCA decomposition and Gaussian noise generation in building
    the semi-synthetic response
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:d1a08272cc29aaeb4d14f3883e6697aa0b7884fc8a3cd1f7465c85f2726fd4af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - remotes
  - stats
  - suffpcr
  - tibble
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# high-dimensional-mse-benchmarking

## Summary

Benchmark SuffPCR against sparse linear models and non-sparse principal component regression on high-dimensional transcriptomic data by comparing out-of-sample MSE and feature-selection overlap (the paper's Figure 3 semi-synthetic NSCLC evaluation).

## When to use

Use when comparing prediction methods (e.g. SuffPCR vs sparse linear models vs non-sparse PCR) on high-dimensional omics data where both out-of-sample MSE and correct feature selection must be evaluated.

## When NOT to use

- When the response is real rather than semi-synthetic and no ground-truth sparse signal direction exists, feature-selection overlap cannot be computed and only MSE is meaningful.
- When the goal is low-dimensional regression with few, weakly correlated features — the method targets high-dimensional omics with correlated features.

## Inputs

- nsclc expression dataset from the suffpcr R package (123 observations x 939 genes)
- semi-synthetic linear phenotype y calibrated to a target signal-to-noise ratio
- candidate subspace dimension grid (d = 1:3)

## Outputs

- out-of-sample MSE per method
- feature-selection overlap per method
- Figure 3-style MSE and selected-features comparison plot (figure3_mse_comparison.png)

## How to apply

Load the nsclc dataset (123 observations, 939 genes) shipped with the suffpcr R package, then build a semi-synthetic response by PCA-decomposing the expression matrix, keeping the top 20 rows of the empirical loading matrix V ranked by norm as the sparse signal direction, recombining the projected signal with Gaussian noise, and calibrating a linear phenotype to a target signal-to-noise ratio. Fit suffPCR across a candidate dimension grid (d=1:3) following the package example pattern out <- suffPCR(X, y, d=1:3), then fit sparse linear model and non-sparse PCR alternatives on the identical semi-synthetic data. Compute out-of-sample MSE and feature-selection overlap for each method and render the Figure 3 comparison (saved as figure3_mse_comparison.png). The rationale is that sparse linear approaches 'can fail to select the correct features, predict poorly relative to non-sparse alternatives', so both metrics are needed; a method like FPS may achieve similar MSE with poor feature selection.

## Related tools

- **suffpcr** (Fits sufficient principal component regression via row-sparse PCA followed by OLS or logistic regression across d=1:3; provides the nsclc dataset) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment for the semi-synthetic benchmark analysis and figure rendering) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub before benchmarking) — https://github.com/dajmcdon/suffpcr
- **stats** (Base R tools for PCA decomposition and Gaussian noise generation in building the semi-synthetic response)

## Examples

```
out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- Reproduced figure matches the Figure 3 pattern reported in the paper: SuffPCR achieves competitive MSE and correct feature selection relative to alternatives.
- Semi-synthetic response is calibrated to the target signal-to-noise ratio after recombining projected signal with Gaussian noise.
- Feature-selection overlap recovers the sparse signal direction defined by the top 20 rows of the empirical loading matrix V ranked by norm.
- All methods are fit on the identical semi-synthetic data, so MSE differences are attributable to the methods, not data splits.
- suffPCR call spans the full candidate dimension grid d=1:3, following the package example pattern.

## Limitations

- Sparse linear approaches (e.g. lasso-type) can fail to select correct features or predict poorly relative to non-sparse alternatives; FPS in the paper achieved similar MSE but poor feature selection.
- Near-optimal performance of SuffPCR holds when the model assumptions are satisfied; deviations may degrade results.
- Dataset is small (123 observations, 939 genes), so MSE estimates may be noisy.
