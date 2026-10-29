---
name: high-dimensional-gene-expression-feature-selection
description: Use when the task is selecting a small, sparse set of predictive genes
  from high-dimensional, correlated omics/expression data (p >> n) for regression
  or classification, and unknown feature groupings (e.g. ribosomal or MHCII genes)
  should be respected.
when_to_use_negative:
- The input is already a small feature table or the features are not high-dimensional/correlated
  — sparse PCA offers no benefit and other linear models suffice
- The prediction task is not linear regression or classification on a continuous/binary
  response — suffPCR implements row-sparse PCA followed by OLS or logistic regression
  only
- A previously selected gene signature already exists and the goal is validation rather
  than de novo feature discovery
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: suffpcr
  role: R package implementing sufficient principal component regression (row-sparse
    PCA + OLS/logistic regression); provides suffPCR(), coef.suffpcr(), and the dlbcl
    expression dataset used for feature selection
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment in which suffpcr is installed and the cross-validation,
    refitting, and coefficient extraction are executed
- name: remotes
  role: Used to install the suffpcr package from GitHub during setup
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:df06ae885c064ccf2ccbbd580fb498537011cb6d2a4da87a594be363b70a21e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - remotes
  - suffpcr
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# high-dimensional-gene-expression-feature-selection

## Summary

Feature selection for high-dimensional transcriptomic regression/classification via SuffPCR: row-sparse PCA followed by OLS or logistic regression on the recovered sparse subspace, so predictions depend on only a small subset of genes (e.g. 87 features / 32 genes / 2 ESTs selected from DLBCL expression data).

## When to use

Use when the task is selecting a small, sparse set of predictive genes from high-dimensional, correlated omics/expression data (p >> n) for regression or classification, and unknown feature groupings (e.g. ribosomal or MHCII genes) should be respected.

## When NOT to use

- The input is already a small feature table or the features are not high-dimensional/correlated — sparse PCA offers no benefit and other linear models suffice
- The prediction task is not linear regression or classification on a continuous/binary response — suffPCR implements row-sparse PCA followed by OLS or logistic regression only
- A previously selected gene signature already exists and the goal is validation rather than de novo feature discovery

## Inputs

- High-dimensional gene-expression dataset shipped with suffpcr (dlbcl, aml, miller, nsclc, vantveer) or an equivalent samples x genes expression matrix X with response y
- Subspace dimension candidates d (e.g. 1:3)
- suffPCR lambda grid parameters (lambda_max, lambda_min, lambda_seq)

## Outputs

- Fitted suffPCR model with feature-sparse loadings
- Vector of selected features via coef.suffpcr(object, i, d, ...) (e.g. 87 nonzero features on DLBCL)
- Annotated gene-list table (e.g. dlbcl_selected_features.csv) tallying unique genes, ESTs, and functional groups (ribosomal-protein genes, MHCII genes)
- Cross-validated lambda choice and predictive performance

## How to apply

Install the suffpcr package from GitHub with remotes::install_github("dajmcdon/suffpcr") and load the dataset of interest (e.g. dlbcl, aml, miller, nsclc, vantveer) with library(suffpcr). Run k-fold cross-validation (e.g. 5-fold on DLBCL) over the suffPCR lambda grid (lambda_max, lambda_min, lambda_seq) on the full data to select the best-performing lambda, then refit suffPCR on all samples at the selected lambda using the chosen subspace dimension d (e.g. out <- suffPCR(X, y, d=1:3)). The method first estimates sparse principal components (row-sparse PCA) and then estimates a linear model (OLS or logistic regression) on the recovered subspace, so the estimated subspace is sparse in features. Extract the nonzero, feature-sparse loadings with coef.suffpcr(object, i, d, ...) to obtain the selected features, then annotate them with gene identifiers (on DLBCL this yields 87 features = 32 unique genes + 2 ESTs, including 17 ribosomal-protein genes and 9 MHCII genes) and write the structured gene list to CSV (e.g. dlbcl_selected_features.csv).

## Related tools

- **suffpcr** (R package implementing sufficient principal component regression (row-sparse PCA + OLS/logistic regression); provides suffPCR(), coef.suffpcr(), and the dlbcl expression dataset used for feature selection) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment in which suffpcr is installed and the cross-validation, refitting, and coefficient extraction are executed)
- **remotes** (Used to install the suffpcr package from GitHub during setup)

## Examples

```
remotes::install_github("dajmcdon/suffpcr"); library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- Cross-validation over the lambda grid selects a lambda that minimizes prediction error on held-out folds (e.g. 5-fold CV on the full DLBCL data)
- coef.suffpcr(object, i, d, ...) returns nonzero loadings for only a small subset of features (87 features on DLBCL), confirming feature sparsity of the estimated subspace
- Selected features annotate to a biologically coherent set: 32 unique genes + 2 ESTs (Hs.22635/ZBTB44, Hs.343870), with 17 ribosomal-protein genes and 9 MHCII genes
- The fitted model's predictions depend only on the small selected gene subset, consistent with the row-sparse PCA subspace
- Structured output (dlbcl_selected_features.csv) contains one row per selected feature with gene identifiers and group flags (ribosomal, MHCII, individually discussed genes CORO1A, FEZ1, RAG1, RYK, CXCL5)

## Limitations

- Near-optimal performance requires the model assumptions to be satisfied (linear single-index structure with sparse loadings); when assumptions fail, other sparse linear approaches may also predict poorly
- SuffPCR addresses settings where feature groupings are unknown; existing sparse linear methods can fail to select correct features or ignore grouping structures — results should be interpreted in light of these alternatives

## Evidence

- [abstract] SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace.: "SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace."
- [abstract] Because the estimated subspace is sparse in the features, the resulting predictions will depend on only a small subset of genes.: "Because the estimated subspace is sparse in the features, the resulting predictions will depend on only a small subset of genes."
- [methods] remotes::install_github("dajmcdon/suffpcr"): "remotes::install_github("dajmcdon/suffpcr")
library(suffpcr)"
- [methods] out <- suffPCR(X, y, d=1:3): "out <- suffPCR(X, y, d=1:3)"
- [intro] The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression.: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [readme] dajmcdon/suffpcr README: suffPCR example call on simulated data with d=1:3: "out <- suffPCR(X, y, d=1:3)"
