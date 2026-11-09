---
name: cross-validated-regularization-tuning
description: Use when fitting suffPCR (row-sparse PCA + OLS/logistic) on high-dimensional,
  correlated omics data like the DLBCL expression set and the penalty lambda / subspace
  dimension d must be chosen by 5-fold cross-validation before extracting sparse loadings.
when_to_use_negative:
- Features are low-dimensional and only weakly correlated, so plain OLS or logistic
  regression without a sparse PCA step is adequate — SuffPCR's advantage targets omics
  data with correlated features
- You need only class labels or predictions from an already-fitted model and have
  no need to tune lambda or select features
- The response cannot be plausibly modeled as a linear (or logistic) function of a
  low-dimensional sparse principal component subspace — the paper notes near-optimality
  holds only 'when the model assumptions are satisfied'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: suffpcr
  role: R package providing suffPCR(), the lambda/d cross-validation machinery, and
    coef.suffpcr() for extracting nonzero feature-sparse loadings
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment in which suffpcr is installed and CV fitting is executed
- name: remotes
  role: Used to install suffpcr from GitHub before tuning can be run
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:8fa9efcb7e00023e9538ce132836b62bb0864d4b1ea2238aad653076aedd1331
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - remotes
  - suffpcr
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cross-validated-regularization-tuning

## Summary

Selects the sparsity penalty (lambda) and subspace dimension d for sufficient principal component regression (SuffPCR) by running 5-fold cross-validation over the lambda grid (lambda_max, lambda_min, lambda_seq), then refitting the model on all samples at the best lambda so that downstream feature selection is reproducible.

## When to use

Use when fitting suffPCR (row-sparse PCA + OLS/logistic) on high-dimensional, correlated omics data like the DLBCL expression set and the penalty lambda / subspace dimension d must be chosen by 5-fold cross-validation before extracting sparse loadings.

## When NOT to use

- Features are low-dimensional and only weakly correlated, so plain OLS or logistic regression without a sparse PCA step is adequate — SuffPCR's advantage targets omics data with correlated features
- You need only class labels or predictions from an already-fitted model and have no need to tune lambda or select features
- The response cannot be plausibly modeled as a linear (or logistic) function of a low-dimensional sparse principal component subspace — the paper notes near-optimality holds only 'when the model assumptions are satisfied'

## Inputs

- High-dimensional gene expression dataset bundled with suffpcr (e.g. dlbcl, aml, miller, nsclc, vantveer) or an equivalent n x p expression matrix with a continuous or binary response y
- suffPCR tuning grid parameters: lambda_max, lambda_min, lambda_seq
- Candidate subspace dimensions d (e.g. 1:3)

## Outputs

- Cross-validation error profile over the lambda grid with the best-performing lambda
- Refitted suffPCR model on all samples at the selected lambda and dimension d
- Nonzero feature-sparse loadings from coef.suffpcr(object, i, d, ...) — the selected feature set (e.g. 87 features / 32 genes / 2 ESTs for DLBCL)
- Structured gene-list table (e.g. dlbcl_selected_features.csv)

## How to apply

Install the suffpcr package (remotes::install_github("dajmcdon/suffpcr")) and load the target expression dataset, e.g. library(suffpcr); data(dlbcl). Run 5-fold cross-validation over the suffPCR lambda grid defined by lambda_max, lambda_min, and lambda_seq to obtain out-of-fold error for each penalty level, and pick the best-performing lambda. Simultaneously consider a small set of subspace dimensions (the README example sweeps d=1:3) since both lambda and d control the sparse principal component subspace. Refit suffPCR on all samples at the selected lambda using the chosen d, which performs row-sparse PCA followed by ordinary least squares or logistic regression on the recovered subspace. Extract the nonzero, feature-sparse loadings with coef.suffpcr(object, i, d, ...) to obtain the selected features (in the DLBCL case this yields 87 features from 32 unique genes and 2 ESTs, including 17 ribosomal-protein and 9 MHCII genes).

## Related tools

- **suffpcr** (R package providing suffPCR(), the lambda/d cross-validation machinery, and coef.suffpcr() for extracting nonzero feature-sparse loadings) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment in which suffpcr is installed and CV fitting is executed)
- **remotes** (Used to install suffpcr from GitHub before tuning can be run) — https://github.com/dajmcdon/suffpcr

## Examples

```
library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- Out-of-fold CV error at the selected lambda is the minimum over the lambda_seq grid
- The refit on all samples at the selected lambda produces nonzero feature-sparse loadings (coef.suffpcr) whose support matches the CV-selected model, not a denser default
- The selected feature count is small relative to p — e.g. 87 features collapsing to 32 unique genes and 2 ESTs (Hs.22635/ZBTB44, Hs.343870) on the DLBCL dataset
- Gene-group structure is interpretable: 17 ribosomal-protein genes and 9 MHCII genes among the DLBCL selections, with individually flagged genes CORO1A, FEZ1, RAG1, RYK, CXCL5
- Prediction performance of the tuned model improves on non-sparse or untuned alternatives, consistent with SuffPCR's claimed improved predictions in high-dimensional regression/classification

## Limitations

- Near-optimal performance is guaranteed only 'when the model assumptions are satisfied' — a single sparse principal component subspace with a linear (or logistic) relationship to y
- Untuned or incorrectly gridded lambda/d choices can mis-select features: existing sparse linear approaches 'can fail to select the correct features, predict poorly relative to non-sparse alternatives or ignore any unknown grouping structures for the features'
- The subspace dimension d must be supplied explicitly by the analyst (e.g. d=1:3 in the README example); there is no fully automatic default documented

## Evidence

- [abstract] SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace.: "SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace."
- [abstract] Because the estimated subspace is sparse in the features, the resulting predictions will depend on only a small subset of genes.: "Because the estimated subspace is sparse in the features, the resulting predictions will depend on only a small subset of genes."
- [readme] The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression.: "The goal of suffpcr is to estimate sufficient principal component regression
by implementing row-sparse PCA followed by ordinary least squares or logistic
regression."
- [readme] You can install the package with remotes::install_github("dajmcdon/suffpcr").: "You can install the package with

``` r
remotes::install_github("dajmcdon/suffpcr")"
- [readme] out <- suffPCR(X, y, d=1:3): "out <- suffPCR(X, y, d=1:3)"
