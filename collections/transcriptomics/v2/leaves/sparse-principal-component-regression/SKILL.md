---
name: sparse-principal-component-regression
description: Use when regressing (or classifying) a phenotype on high-dimensional,
  correlated transcriptomic features where marginal-correlation screening is unreliable
  and you need both accurate prediction and feature-sparse, gene-level interpretability.
when_to_use_negative:
- Low-dimensional or independent-feature settings where standard sparse linear models
  (Lasso, Elastic Net, SPC) already satisfy the marginal-correlation screening assumption
- When the full dense principal component subspace (non-sparse PCR) is desired and
  no feature-level interpretability is needed
- When a pre-existing feature/signature table is already given and no subspace estimation
  is required
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: suffpcr
  role: Core R package implementing row-sparse PCA followed by OLS or logistic regression;
    provides suffPCR(), coef.suffpcr(), and bundled datasets (aml, dlbcl, miller,
    nsclc, vantveer)
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: 'Installs the suffpcr package from GitHub: remotes::install_github(''dajmcdon/suffpcr'')'
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment for fitting suffPCR and computing test MSE, precision/recall,
    and ROC curves
- name: fps
  role: Dependency providing the Fantope projection and selection (FPS) sparse-PCA
    solver underlying the row-sparse subspace estimation and the FPS-PCR competitor
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:83cc7ec39891ba2b23b2950fc0a498ae8df897f0fc336cf90bd9aa178c20e00f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - fps
  - remotes
  - stats
  - suffpcr
  - tibble
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sparse-principal-component-regression

## Summary

Fit SuffPCR: a two-stage estimator that first recovers a row-sparse principal component subspace via sparse PCA (Fantope projection with ADMM and hard-threshold row selection), then fits OLS or logistic regression on the recovered subspace, so predictions depend on only a small subset of genes.

## When to use

Use when regressing (or classifying) a phenotype on high-dimensional, correlated transcriptomic features where marginal-correlation screening is unreliable and you need both accurate prediction and feature-sparse, gene-level interpretability.

## When NOT to use

- Low-dimensional or independent-feature settings where standard sparse linear models (Lasso, Elastic Net, SPC) already satisfy the marginal-correlation screening assumption
- When the full dense principal component subspace (non-sparse PCR) is desired and no feature-level interpretability is needed
- When a pre-existing feature/signature table is already given and no subspace estimation is required

## Inputs

- High-dimensional feature matrix X (n samples x p features, e.g. gene expression; package ships aml, dlbcl, miller, nsclc, vantveer transcriptomic datasets; nsclc is 123 observations x 939 genes)
- Response vector y (continuous for linear regression or binary for logistic regression)
- Candidate subspace dimension grid (e.g. d=1:3)
- Optional: penalized competitor fits (glmnet, SPC, AIMER, ISPCA, FPS) and an independent test set for benchmarking

## Outputs

- Row-sparse loading matrix V̂d for each subspace dimension d (zero/nonzero row pattern defines selected features)
- Subspace regression coefficients β̂ via coef.suffpcr(object, i, d, ...)
- Predictions on held-out test data with test-set MSE
- Feature-selection precision/recall and ROC curves over the regularization path (e.g. simulation_results.csv, roc_curves.png, figure3_mse_comparison.png)

## How to apply

Install the R package from GitHub with remotes::install_github('dajmcdon/suffpcr') and call suffPCR(X, y, d=1:3) over a grid of subspace dimensions d. Stage 1 implements row-sparse PCA (FPS with an ℓ2,0-type AIRLB penalty solved by ADMM with truncated eigendecomposition and elementwise soft-thresholding); rows of the loading matrix V̂d with small ℓ2 norm are hard-thresholded, with the threshold t chosen automatically by the elbow criterion (Algorithm 2), which maximizes the difference in empirical derivatives of observation-weighted row-norm variances on either side of a candidate threshold. Stage 2 fits a linear model (OLS for regression, logistic for classification) on the recovered subspace to obtain β̂. Tune penalized competitors (Lasso, Ridge, Elastic Net via glmnet, SPC, AIMER, ISPCA, FPS-PCR) on a held-out validation set and compare all methods on test-set MSE, selected features, precision/recall, and feature-selection ROC curves over regularization paths. The method is designed so that correlated features violating marginal-correlation screening (e.g. zero marginal correlation but non-zero coefficients) are still recovered.

## Related tools

- **suffpcr** (Core R package implementing row-sparse PCA followed by OLS or logistic regression; provides suffPCR(), coef.suffpcr(), and bundled datasets (aml, dlbcl, miller, nsclc, vantveer)) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub: remotes::install_github('dajmcdon/suffpcr')) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment for fitting suffPCR and computing test MSE, precision/recall, and ROC curves)
- **fps** (Dependency providing the Fantope projection and selection (FPS) sparse-PCA solver underlying the row-sparse subspace estimation and the FPS-PCR competitor)

## Examples

```
library(suffpcr); out <- suffPCR(X, y, d=1:3); coef.suffpcr(out, i, d)
```

## Evaluation signals

- For each d in 1:3, the estimated subspace V̂d is row-sparse and the reimplemented V̂d and β̂ match package coef.suffpcr output within floating-point tolerance (per-d max absolute difference and zero/nonzero row-pattern agreement)
- On the Algorithm 3 adversarial synthetic benchmark (n=100, p=1000, rank-1 latent factor with spike 5 / noise 0.1, SNR 5/5, 15 true features of which 10 carry marginal correlation), SuffPCR attains smaller test MSE than all alternatives including the Oracle, and SuffPCR and AIMER achieve a perfect ROC curve while Lasso, Elastic Net, and SPC cannot identify five features
- Every benchmarked method has a complete test-set MSE and precision/recall entry in simulation_results.csv
- On DLBCL, the model at the cross-validated lambda selects 87 features corresponding to 32 unique genes and 2 ESTs (Hs.22635/ZBTB44, Hs.343870), including 17 ribosomal-protein genes and 9 MHCII genes
- Semi-synthetic NSCLC (Figure 3) pattern reproduced: SuffPCR competitive MSE with correct feature selection, though FPS achieves similar MSE with poor feature selection

## Limitations

- Near-optimal performance is guaranteed only when the model assumptions are satisfied; alternative sparse linear approaches can fail to select correct features or predict poorly relative to non-sparse alternatives
- Competing penalized methods require tuning on a held-out validation set for fair benchmarking
- No changelog is available for the repository, so version-specific behavior changes cannot be tracked

## Evidence

- [readme] Two-stage definition: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [abstract] Prediction depends on few genes: "Because the estimated subspace is sparse in the features, the resulting predictions will depend on only a small subset of genes."
- [abstract] Near-optimal performance claim: "SuffPCR works well on a variety of simulated and experimental transcriptomic data, performing nearly optimally when the model assumptions are satisfied."
- [readme] Canonical invocation: "X <- 5 * tcrossprod(U, V) + 0.1 * matrix(rnorm(n * p), n)
y <- U + rnorm(n)
out <- suffPCR(X, y, d=1:3)"
- [methods] Installation path: "remotes::install_github("dajmcdon/suffpcr")"
