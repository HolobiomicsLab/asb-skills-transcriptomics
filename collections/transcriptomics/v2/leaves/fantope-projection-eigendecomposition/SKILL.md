---
name: fantope-projection-eigendecomposition
description: Use when replicating or auditing how suffPCR turns a sparse PCA (FPS)
  solution into a row-sparse loading matrix V̂d — i.e., Fantope projection via truncated
  eigendecomposition, ADMM soft-thresholding, and Algorithm 2 elbow threshold selection.
when_to_use_negative:
- When the analysis goal is standard (dense) PCA rather than row-sparse PCA with a
  downstream regression — the hard-thresholding of small-ℓ2-norm rows is specific
  to SuffPCR's goal of feature-sparse subspaces
- When features are known in advance and no sparse subspace recovery is needed, e.g.,
  if the phenotype already depends on an explicit, verified gene list rather than
  an unknown feature grouping
tools:
- name: suffpcr
  role: Reference R implementation of sufficient principal component regression (row-sparse
    PCA via FPS + OLS/logistic regression) against which the reimplementation is compared
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr reference package from GitHub for the validation comparison
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:628229507fa9e3c55fdc0d50defd70584d3b2bd1bd1b54df900f7674129d6be5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - remotes
  - suffpcr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# fantope-projection-eigendecomposition

## Summary

Reconstructs the SuffPCR estimation pipeline: an ADMM solver whose Fantope projection step is approximated by a truncated eigendecomposition with elementwise soft-thresholding (Algorithm 1), followed by elbow-based hard-thresholding of row norms (Algorithm 2) to obtain row-sparse principal components before regressing the phenotype on the recovered subspace.

## When to use

Use when replicating or auditing how suffPCR turns a sparse PCA (FPS) solution into a row-sparse loading matrix V̂d — i.e., Fantope projection via truncated eigendecomposition, ADMM soft-thresholding, and Algorithm 2 elbow threshold selection.

## When NOT to use

- When the analysis goal is standard (dense) PCA rather than row-sparse PCA with a downstream regression — the hard-thresholding of small-ℓ2-norm rows is specific to SuffPCR's goal of feature-sparse subspaces
- When features are known in advance and no sparse subspace recovery is needed, e.g., if the phenotype already depends on an explicit, verified gene list rather than an unknown feature grouping

## Inputs

- Synthetic or real numeric feature matrix X (e.g., n=100 x p=50 transcriptomic expression matrix)
- Phenotype vector y (continuous for OLS regression, binary for logistic regression)
- Target subspace dimension d (e.g., d = 1:3)
- Installed suffpcr R package (reference implementation for comparison)

## Outputs

- Row-sparse loading matrix V̂d for each dimension d in 1:3
- Downstream coefficient estimate β̂ from the linear model fit on the recovered subspace
- reimplementation_agreement.csv containing per-d max absolute differences and zero/nonzero row-pattern agreement versus the package output

## How to apply

Generate the synthetic benchmark exactly as in the README example (n=100, p=50; U ~ N(0,1); unit-norm V with 5 nonzero entries; X = 5*tcrossprod(U,V) + 0.1*Gaussian noise; y = U + noise). Independently of the package's internal code paths, implement the ADMM iteration in which the Fantope projection is approximated by a truncated eigendecomposition and elementwise soft-thresholding enforces the sparsity penalty. Then apply Algorithm 2's elbow criterion: compute the empirical derivative of the observation-weighted variances of row norms on each side of a candidate threshold and maximize their difference to separate signal rows from noise rows, yielding the hard threshold applied in Steps 9–11 of Algorithm 1. Fit the downstream linear model (OLS or logistic) on the recovered subspace for each dimension d = 1:3, and compare V̂d and β̂ against the reference suffPCR() fit (coef.suffpcr) on identical data.

## Related tools

- **suffpcr** (Reference R implementation of sufficient principal component regression (row-sparse PCA via FPS + OLS/logistic regression) against which the reimplementation is compared) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr reference package from GitHub for the validation comparison) — https://github.com/dajmcdon/suffpcr

## Examples

```
library(suffpcr); n <- 100; p <- 50; U <- rnorm(n); V <- c(rnorm(5), rep(0, p - 5)); V <- V / sqrt(sum(V^2)); X <- 5 * tcrossprod(U, V) + 0.1 * matrix(rnorm(n * p), n); y <- U + rnorm(n); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- For each d in 1:3, V̂d from the reimplementation matches the package coefficients (coef.suffpcr) to within floating-point noise (max absolute difference ≈ 0) on the synthetic dataset
- β̂ from the downstream linear model matches the package output to within numerical tolerance for each dimension
- The estimated subspace is row-sparse, and the zero/nonzero row pattern of V̂d agrees exactly with the package output
- Algorithm 2's elbow criterion isolates the 5 signal rows (the nonzero entries of the unit-norm V) from the 45 noise rows in the synthetic benchmark
- reimplementation_agreement.csv is produced with per-d max absolute differences and row-pattern agreement entries

## Limitations

- Near-optimal performance requires the model's assumptions to be satisfied; deviations (e.g., strongly uncorrelated or non-sparse structure) can degrade behavior as with existing sparse linear approaches
- The hard threshold is data-dependent: the elbow criterion (Algorithm 2) relies on a clear separation of observation-weighted variance derivatives between signal and noise row norms and may be ambiguous when that separation is weak
- The ADMM Fantope step is only approximated by a truncated eigendecomposition, so agreement with the reference is valid only within numerical tolerance, not bit-identically

## Evidence

- [full_text] The algorithm makes the FPS solution row-sparse by hard-thresholding rows of V̂d whose ℓ2 norm is small (Steps 9–11 of Algorithm 1): [evidence span withheld — anchor status: not_found]
- [full_text] Algorithm 2 threshold selection: [evidence span withheld — anchor status: not_found]
- [abstract] SuffPCR two-stage approach: "SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace."
- [readme] README example data generation and fit: "X <- 5 * tcrossprod(U, V) + 0.1 * matrix(rnorm(n * p), n); y <- U + rnorm(n); out <- suffPCR(X, y, d=1:3)"
- [methods] Package installation: "remotes::install_github("dajmcdon/suffpcr")"
