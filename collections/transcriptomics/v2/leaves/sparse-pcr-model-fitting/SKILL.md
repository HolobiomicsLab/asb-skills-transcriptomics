---
name: sparse-pcr-model-fitting
description: Use when fitting regression or classification models on high-dimensional,
  correlated omics data (e.g. transcriptomic microarrays with n << p) where sparse
  feature selection and strong prediction MSE are both required.
when_to_use_negative:
- The response-generating signal is not sparse in the features or does not lie near
  a low-dimensional principal subspace, since SuffPCR assumes a sparse subspace model
  and performs nearly optimally only when model assumptions are satisfied
- You need a one-off exploratory PCA of the expression matrix without a regression
  target — row-sparse PCA here is a means to sufficient PCR, not general dimension
  reduction
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: suffpcr
  role: R package providing the suffPCR() function that implements row-sparse PCA
    followed by OLS or logistic regression; also ships the five microarray datasets
    (aml, dlbcl, miller, nsclc, vantveer) used for benchmarking
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Used to install the suffpcr package from GitHub
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment in which suffPCR is fitted and benchmarks (10 random 40/30/30
    train/validation/test splits, lambda selection on validation) are run
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:89cae16c370917518b4d79a3db88e0ca41ecc345905e02c686134ef1ff0ab64d
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# sparse-pcr-model-fitting

## Summary

Fits sufficient principal component regression (SuffPCR) for high-dimensional omics regression/classification by first estimating row-sparse principal components and then fitting OLS or logistic regression on the recovered subspace, so predictions depend on only a small subset of genes.

## When to use

Use when fitting regression or classification models on high-dimensional, correlated omics data (e.g. transcriptomic microarrays with n << p) where sparse feature selection and strong prediction MSE are both required.

## When NOT to use

- The response-generating signal is not sparse in the features or does not lie near a low-dimensional principal subspace, since SuffPCR assumes a sparse subspace model and performs nearly optimally only when model assumptions are satisfied
- You need a one-off exploratory PCA of the expression matrix without a regression target — row-sparse PCA here is a means to sufficient PCR, not general dimension reduction

## Inputs

- n x p gene expression matrix X (e.g. microarray datasets aml, dlbcl, miller, nsclc, vantveer shipped with the suffpcr package)
- Response vector y (continuous for linear regression; binary for logistic classification)
- Grid of candidate subspace dimensions d (e.g. 1:3) and candidate penalty (lambda) values selected on a validation fold

## Outputs

- Fitted SuffPCR model object with estimated sparse principal components, selected features, and predictions
- Out-of-sample test MSE and number of selected features per fit (e.g. Table 2 style benchmark summaries)
- Selected-feature subsets whose predictions depend on only a small subset of genes

## How to apply

Install the suffpcr package from GitHub with remotes::install_github('dajmcdon/suffpcr') and load it in R. Prepare an n x p expression matrix X and response y (continuous for regression; the method also supports classification via logistic regression). Call suffPCR(X, y, d=1:3) to fit over a grid of subspace dimensions d (the README example uses d=1:3 with n=100, p=50, a signal loading V supported on only 5 of 50 features, and X = 5*tcrossprod(U, V) + 0.1 noise), selecting the dimension and penalty (e.g. over log-spaced lambda values) on a held-out validation fold. Internally the method implements row-sparse PCA followed by ordinary least squares or logistic regression on the estimated sparse subspace, so the fitted model yields both predictions and an interpretable set of selected genes. Benchmark against feature-selecting alternatives (SPC, Lasso, Elastic Net, SPCA, DSPCA) under an identical train/validation/test splitting and lambda-selection protocol when evaluating performance.

## Related tools

- **suffpcr** (R package providing the suffPCR() function that implements row-sparse PCA followed by OLS or logistic regression; also ships the five microarray datasets (aml, dlbcl, miller, nsclc, vantveer) used for benchmarking) — https://github.com/dajmcdon/suffpcr
- **remotes** (Used to install the suffpcr package from GitHub) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment in which suffPCR is fitted and benchmarks (10 random 40/30/30 train/validation/test splits, lambda selection on validation) are run)

## Examples

```
library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- Reproduced benchmark matches the reported ranking pattern in which SuffPCR attains the lowest average test MSE on 4 of the 5 real microarray datasets
- Selected feature count remains comparatively small relative to feature-selecting alternatives (SPC, Lasso, Elastic Net, SPCA, DSPCA)
- Validation-fold lambda/dimension selection yields consistent test MSE across 10 random 40/30/30 splits
- In the semi-synthetic NSCLC setting, recovered sparse loadings overlap the planted top-20 sparse signal direction (feature-selection overlap), matching the Figure 3 pattern
- Estimated subspace is row-sparse: predictions depend on only a small subset of genes, not the full feature set

## Limitations

- Near-optimal performance holds when model assumptions are satisfied; deviations from the sparse-subspace assumption can degrade predictions
- Existing sparse linear approaches (and by comparison SuffPCR's alternatives) can fail to select the correct features or predict poorly relative to non-sparse alternatives — benchmarking protocol must match splitting and lambda selection to make comparisons fair
- In the semi-synthetic NSCLC analysis, competing methods such as FPS can achieve similar MSE with poor feature selection, so MSE alone is insufficient for judging correctness

## Evidence

- [abstract] Method implements row-sparse PCA then linear model: "SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace."
- [abstract] Predictions depend on few genes: "Because the estimated subspace is sparse in the features, the resulting predictions will depend on only a small subset of genes."
- [intro] Goal of suffpcr package: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [methods] Package installation command: "remotes::install_github("dajmcdon/suffpcr")"
- [readme] README example invocation with d=1:3 grid: "out <- suffPCR(X, y, d=1:3)"
- [abstract] Performance claim on transcriptomic data: "SuffPCR works well on a variety of simulated and experimental transcriptomic data, performing nearly optimally when the model assumptions are satisfied."
