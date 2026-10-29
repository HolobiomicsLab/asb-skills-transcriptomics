---
name: feature-selection-accuracy-comparison
description: Use when comparing high-dimensional regression methods on omics data
  where both predictive accuracy (out-of-sample MSE) and correctness of the selected
  gene subset matter, e.g. reproducing Figure 3-style semi-synthetic benchmarks.
when_to_use_negative:
- The response is a real, unmodified phenotype rather than a semi-synthetic one —
  there is no ground-truth feature set, so feature-selection overlap cannot be computed
- The data are low-dimensional or features are not correlated; the paper's motivation
  is specifically high-dimensional omics with correlated features where sparse PCA
  recovers a feature-sparse subspace
- The goal is classification needing logistic regression on an unknown latent subspace
  without a feature-accuracy comparison component
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: suffpcr
  role: 'Primary method: estimates sufficient principal component regression by row-sparse
    PCA followed by ordinary least squares, with sparse estimated PCs enabling feature-selection
    evaluation'
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub prior to running the benchmark
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment for fitting all competing methods and computing MSE/overlap
    metrics
  repo: https://github.com/dajmcdon/suffpcr
- name: stats
  role: Provides PCA decomposition for building the semi-synthetic response and OLS-based
    comparison regression
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:0e7ff13a90f9d3c4e0d1ad2796a8708051d1b5a58fc55ce66db223445c20a759
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# feature-selection-accuracy-comparison

## Summary

Benchmark SuffPCR against sparse linear models and non-sparse principal component regression on semi-synthetic transcriptomic data, jointly evaluating out-of-sample prediction MSE and feature-selection overlap. This skill addresses the paper's finding that sparse methods can predict well yet select the wrong features.

## When to use

Use when comparing high-dimensional regression methods on omics data where both predictive accuracy (out-of-sample MSE) and correctness of the selected gene subset matter, e.g. reproducing Figure 3-style semi-synthetic benchmarks.

## When NOT to use

- The response is a real, unmodified phenotype rather than a semi-synthetic one — there is no ground-truth feature set, so feature-selection overlap cannot be computed
- The data are low-dimensional or features are not correlated; the paper's motivation is specifically high-dimensional omics with correlated features where sparse PCA recovers a feature-sparse subspace
- The goal is classification needing logistic regression on an unknown latent subspace without a feature-accuracy comparison component

## Inputs

- suffpcr package datasets in R: nsclc (123 observations, 939 genes), aml, dlbcl, miller, vantveer
- High-dimensional expression matrix X (n x p) with correlated genomic features
- Semi-synthetic response vector y calibrated to a target signal-to-noise ratio

## Outputs

- Out-of-sample MSE per method (SuffPCR vs sparse linear models vs non-sparse PCR)
- Feature-selection overlap (agreement between each method's selected genes and the known sparse signal direction)
- Comparison figure of MSE and selected features (e.g. figure3_mse_comparison.png)

## How to apply

Load a shipped expression dataset (e.g. the nsclc dataset with 123 observations and 939 genes from the suffpcr package in R). Construct a semi-synthetic response by PCA-decomposing the expression matrix, keeping the top 20 rows of the empirical loading matrix V ranked by norm as the sparse signal direction, recombining the projected signal with Gaussian noise, and calibrating a linear phenotype to a target signal-to-noise ratio. Fit suffPCR across a candidate dimension grid (e.g. d=1:3 via out <- suffPCR(X, y, d=1:3)), then fit sparse linear models and non-sparse principal component regression on the same data. Compute out-of-sample MSE and feature-selection overlap for each method and render a side-by-side comparison; judge success when the reproduced figure matches the paper's pattern — SuffPCR achieving competitive MSE while also selecting the correct sparse features, whereas alternatives like FPS may match MSE with poor feature selection.

## Related tools

- **suffpcr** (Primary method: estimates sufficient principal component regression by row-sparse PCA followed by ordinary least squares, with sparse estimated PCs enabling feature-selection evaluation) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub prior to running the benchmark) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment for fitting all competing methods and computing MSE/overlap metrics) — https://github.com/dajmcdon/suffpcr
- **stats** (Provides PCA decomposition for building the semi-synthetic response and OLS-based comparison regression)

## Examples

```
library(suffpcr); data(nsclc); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- The reproduced comparison figure matches the Figure 3 pattern reported in the paper: SuffPCR competitive on MSE with correct feature selection
- SuffPCR's selected genes overlap the top-20-norm rows of the empirical loading matrix V used to build the semi-synthetic signal
- Alternatives like FPS are expected to achieve similar MSE but show poor feature-selection overlap — a divergence that validates the metric design
- suffPCR runs across the candidate dimension grid d=1:3 and the returned object contains fitted models over that grid
- Dataset dimensions match the package documentation (e.g. nsclc: 123 observations x 939 genes)

## Limitations

- Near-optimal performance of SuffPCR holds only when the model assumptions are satisfied; existing sparse approaches can fail to select correct features or predict poorly relative to non-sparse alternatives
- In the semi-synthetic setting only the phenotypes are generated from real data, so feature-selection accuracy depends on the constructed sparse ground truth (top 20 rows of V by norm)
- FPS can achieve similar MSE to SuffPCR while selecting poor features, so MSE alone is not a sufficient evaluation signal

## Evidence

- [abstract] joint MSE/feature-selection evaluation: "These methods can fail to select the correct features, predict poorly relative to non-sparse alternatives or ignore any unknown grouping structures for the features."
- [results] semi-synthetic benchmark result: [evidence span withheld — anchor status: not_found]
- [readme] core algorithm: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [readme] example invocation: "out <- suffPCR(X, y, d=1:3)"
