---
name: admm-optimization-implementation
description: Use when reconstructing or independently verifying the row-sparse FPS
  estimation path of SuffPCR (Algorithm 1 with AIRLB plus Algorithm 2 elbow thresholding),
  or when implementing an ADMM solver for sparse PCA loading matrices whose sparsity
  must match the suffpcr package.
when_to_use_negative:
- The goal is unsupervised dimension reduction only — SuffPCR is specifically 'row-sparse
  PCA followed by ordinary least squares or logistic regression', so without a phenotype
  y to regress there is no sufficient-PC objective to solve.
- The target subspace need not be row-sparse in features; existing sparse approaches
  are noted to sometimes 'fail to select the correct features' or 'predict poorly
  relative to non-sparse alternatives', so non-sparse PCR alternatives are preferable
  when sparsity assumptions do not hold.
- The analysis only needs to run the package as-is (out <- suffPCR(...)) rather than
  independently reconstruct or validate its ADMM/soft-thresholding internals.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: suffpcr
  role: 'Reference implementation: supplies the synthetic example, coef.suffpcr output,
    and suffPCR(X, y, d=1:3) fit against which the ADMM reimplementation is compared'
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Language in which the synthetic dataset is generated and the independent ADMM/soft-thresholding
    and elbow-thresholding implementations are written
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Used to install the reference suffpcr package from GitHub before running the
    comparison fit
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:2bde3374f868bde2e282ae57d55505ec03c9fee31bc60d1932e10d723ce124b5
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
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# admm-optimization-implementation

## Summary

Reimplementation of the SuffPCR estimation algorithm: an ADMM solver in which the Fantope projection and selection (FPS) problem is solved via truncated eigendecomposition with elementwise soft-thresholding, made row-sparse by hard-thresholding small row norms, with the threshold chosen by an automatic elbow criterion (Algorithm 2).

## When to use

Use when reconstructing or independently verifying the row-sparse FPS estimation path of SuffPCR (Algorithm 1 with AIRLB plus Algorithm 2 elbow thresholding), or when implementing an ADMM solver for sparse PCA loading matrices whose sparsity must match the suffpcr package.

## When NOT to use

- The goal is unsupervised dimension reduction only — SuffPCR is specifically 'row-sparse PCA followed by ordinary least squares or logistic regression', so without a phenotype y to regress there is no sufficient-PC objective to solve.
- The target subspace need not be row-sparse in features; existing sparse approaches are noted to sometimes 'fail to select the correct features' or 'predict poorly relative to non-sparse alternatives', so non-sparse PCR alternatives are preferable when sparsity assumptions do not hold.
- The analysis only needs to run the package as-is (out <- suffPCR(...)) rather than independently reconstruct or validate its ADMM/soft-thresholding internals.

## Inputs

- Synthetic benchmark matrix X (n=100, p=50) generated as 5*tcrossprod(U,V) + 0.1*matrix(rnorm(n*p), n) with U ~ N(0,1) and unit-norm sparse V (5 nonzero entries)
- Response vector y = U + rnorm(n) generated in R
- Dimension specification d = 1:3 for the number of sufficient principal components
- Row norms of the estimated FPS loading matrix V̂d (input to the Algorithm 2 elbow criterion)

## Outputs

- Row-sparse FPS loading estimates V̂d for each d in 1:3, with zero/nonzero row patterns determined by the elbow-selected threshold
- Downstream regression coefficients β̂ per dimension from the linear model fitted on the recovered subspace
- reimplementation_agreement.csv containing per-d max absolute differences versus package coefficients and agreement of zero/nonzero row patterns

## How to apply

Generate the synthetic benchmark exactly as in the package example (n=100, p=50; U ~ N(0,1); unit-norm loading V with 5 nonzero entries; X = 5*tcrossprod(U,V) + 0.1*Gaussian noise; y = U + noise). Independently of the package internals, implement the ADMM iteration in which the Fantope projection is approximated by a truncated eigendecomposition and elementwise soft-thresholding enforces the penalty; make the FPS solution row-sparse by hard-thresholding rows of V̂d whose ℓ2 norm is small (Steps 9–11 of Algorithm 1). Select the hard threshold t automatically with the elbow criterion (Algorithm 2): compute the empirical derivative of the observation-weighted variances of row norms on each side of a candidate threshold and maximize their difference to separate signal rows from noise rows. Fit the downstream linear model (OLS or logistic) on the recovered subspace to obtain β̂ for each dimension d = 1:3, then compare against the reference fit `out <- suffPCR(X, y, d=1:3)` via coef.suffpcr.

## Related tools

- **suffpcr** (Reference implementation: supplies the synthetic example, coef.suffpcr output, and suffPCR(X, y, d=1:3) fit against which the ADMM reimplementation is compared) — https://github.com/dajmcdon/suffpcr
- **R** (Language in which the synthetic dataset is generated and the independent ADMM/soft-thresholding and elbow-thresholding implementations are written) — https://github.com/dajmcdon/suffpcr
- **remotes** (Used to install the reference suffpcr package from GitHub before running the comparison fit) — https://github.com/dajmcdon/suffpcr

## Examples

```
library(suffpcr); n<-100; p<-50; U<-rnorm(n); V<-c(rnorm(5),rep(0,p-5)); V<-V/sqrt(sum(V^2)); X<-5*tcrossprod(U,V)+0.1*matrix(rnorm(n*p),n); y<-U+rnorm(n); out<-suffPCR(X, y, d=1:3)
```

## Evaluation signals

- For each d in 1:3, the independently estimated V̂d matches the package output (coef.suffpcr) to within numerical tolerance — no discrepancy beyond floating-point noise.
- For each d in 1:3, the downstream β̂ matches the package's regression coefficients within floating-point tolerance on the synthetic dataset.
- The estimated subspace is row-sparse for every d, and the zero/nonzero row pattern matches the reference fit (recorded in reimplementation_agreement.csv).
- The Algorithm 2 elbow criterion separates the 5 planted signal rows from the 45 noise rows on the synthetic data (V has exactly 5 nonzero entries).
- The ADMM Fantope approximation uses a truncated eigendecomposition and the penalty is enforced via elementwise soft-thresholding, consistent with Algorithm 1.

## Limitations

- Agreement is only verified on the synthetic benchmark (n=100, p=50, 5 signal features, noise sd 0.1 for X and unweighted rnorm(n) for y); performance on other regimes is not validated by this reconstruction.
- The Fantope projection is only approximated by a truncated eigendecomposition, so exact equality with the package's internal solver is subject to numerical tolerance rather than guaranteed.
- The hard-thresholding step relies on Algorithm 2's empirical derivative of observation-weighted variances of row norms; if signal and noise row norms are not well separated, the elbow threshold may misclassify rows.

## Evidence

- [intro] The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression.: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [abstract] SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace.: "SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace."
- [readme] X <- 5 * tcrossprod(U, V) + 0.1 * matrix(rnorm(n * p), n): "X <- 5 * tcrossprod(U, V) + 0.1 * matrix(rnorm(n * p), n)
y <- U + rnorm(n)
out <- suffPCR(X, y, d=1:3)"
- [abstract] SuffPCR works well on simulated and experimental transcriptomic data: "SuffPCR works well on a variety of simulated and experimental transcriptomic data, performing nearly optimally when the model assumptions are satisfied."
