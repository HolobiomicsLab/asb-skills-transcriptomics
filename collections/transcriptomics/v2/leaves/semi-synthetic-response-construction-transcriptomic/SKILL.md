---
name: semi-synthetic-response-construction-transcriptomic
description: Use when benchmarking high-dimensional regression methods (e.g. SuffPCR
  vs sparse linear models vs non-sparse PCR) on real omics data where no true response
  exists, and ground-truth feature selection is needed.
when_to_use_negative:
- The response y is already observed (real phenotype/outcome data) — no synthetic
  response needs to be constructed and feature-selection ground truth comes from other
  sources.
- The analysis is fully simulated (both X and y synthetic, as in the README example
  with n=100, p=50) rather than semi-synthetic over real genomic data.
- The goal is classification on a binary response rather than constructing a linear
  phenotype for regression benchmarking.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: suffpcr
  role: R package providing the nsclc dataset, row-sparse PCA followed by OLS/logistic
    regression (suffPCR(X, y, d=1:3)), and the fitted method being benchmarked on
    the semi-synthetic response
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub prior to running the semi-synthetic
    analysis
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment for the semi-synthetic response construction, suffPCR
    fitting, and Figure 3 rendering
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:b34ac49bf8128d8a7002abc515ba6f3b35caa12f9ae494202d524dc84b6b8da2
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# semi-synthetic-response-construction-transcriptomic

## Summary

Constructs semi-synthetic phenotypes for transcriptomic benchmarking by keeping real genomic data (X) but generating the response (y) from a known sparse direction — here the top-norm rows of the PCA loading matrix V — so that prediction MSE and feature-selection overlap can be evaluated against ground truth, as in Figure 3 of the SuffPCR paper.

## When to use

Use when benchmarking high-dimensional regression methods (e.g. SuffPCR vs sparse linear models vs non-sparse PCR) on real omics data where no true response exists, and ground-truth feature selection is needed.

## When NOT to use

- The response y is already observed (real phenotype/outcome data) — no synthetic response needs to be constructed and feature-selection ground truth comes from other sources.
- The analysis is fully simulated (both X and y synthetic, as in the README example with n=100, p=50) rather than semi-synthetic over real genomic data.
- The goal is classification on a binary response rather than constructing a linear phenotype for regression benchmarking.

## Inputs

- nsclc expression dataset (123 observations × 939 genes) shipped with the suffpcr R package
- Empirical PCA loading matrix V from the expression matrix
- Gaussian noise draws for signal-plus-noise response construction
- Target signal-to-noise ratio for phenotype calibration

## Outputs

- Semi-synthetic response vector y paired with the real nsclc expression matrix X
- Out-of-sample MSE per method (SuffPCR, sparse linear models, non-sparse PCR)
- Feature-selection overlap of each method with the true top-20 loading rows of V
- Figure 3 comparison plot of MSE and selected features (figure3_mse_comparison.png)

## How to apply

Load a real expression matrix shipped with the package — for the Figure 3 analysis, the nsclc dataset (123 observations, 939 genes) from the suffpcr package. PCA-decompose the expression matrix and treat the top 20 rows of the empirical loading matrix V, ranked by norm, as the sparse signal direction. Reconstruct the projected signal from this direction, add Gaussian noise, and calibrate a linear phenotype y to a target signal-to-noise ratio so only the response is synthetic while predictors remain real experimental genomic data. Then fit suffPCR across a candidate dimension grid (d=1:3, following the package example pattern out <- suffPCR(X, y, d=1:3)) alongside sparse linear models and non-sparse principal component regression on the identical semi-synthetic data. Compare methods by out-of-sample MSE and overlap of selected features with the true sparse loading direction; the rationale is that sparse linear approaches can fail to select correct features even when MSE is competitive.

## Related tools

- **suffpcr** (R package providing the nsclc dataset, row-sparse PCA followed by OLS/logistic regression (suffPCR(X, y, d=1:3)), and the fitted method being benchmarked on the semi-synthetic response) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub prior to running the semi-synthetic analysis) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment for the semi-synthetic response construction, suffPCR fitting, and Figure 3 rendering) — https://github.com/dajmcdon/suffpcr

## Examples

```
library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- The reproduced Figure 3 matches the pattern reported in the paper: SuffPCR achieves competitive MSE and correct feature selection relative to alternatives in the semi-synthetic NSCLC analysis.
- The semi-synthetic response y must be calibrated to the target signal-to-noise ratio before model fitting.
- Feature-selection overlap is computed against the exact top 20 rows of the empirical loading matrix V ranked by norm — a method selecting unrelated genes (e.g. FPS achieving similar MSE with poor feature selection) demonstrates the evaluation discriminates selection quality.
- Dimensions for suffPCR are fit across the candidate grid d=1:3, matching the package example invocation pattern.
- All comparison methods (sparse linear models and non-sparse principal component regression) are fit on the identical semi-synthetic X, y.

## Limitations

- FPS can achieve similar MSE to SuffPCR while selecting the wrong features, so MSE alone is insufficient for judging method quality in this design.
- SuffPCR performs nearly optimally only when model assumptions are satisfied; the semi-synthetic construction is designed to meet them, and real observed responses may not.
- Results depend on the calibration signal-to-noise ratio and the choice of top-20 loading rows; other sparsity levels or SNRs are not validated by this task.

## Evidence

- [readme] Goal of the method being benchmarked: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [abstract] Two-stage method structure motivating the semi-synthetic ground truth: "SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace."
- [abstract] Failure mode the semi-synthetic design exposes in alternatives: "These methods can fail to select the correct features, predict poorly relative to non-sparse alternatives or ignore any unknown grouping structures for the features."
- [other] Semi-synthetic construction procedure for the NSCLC Figure 3 analysis: [evidence span withheld — anchor status: not_found]
- [other] Rationale for including feature-selection overlap alongside MSE: [evidence span withheld — anchor status: not_found]
