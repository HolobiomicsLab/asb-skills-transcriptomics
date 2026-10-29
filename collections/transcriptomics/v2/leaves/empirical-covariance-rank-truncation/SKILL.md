---
name: empirical-covariance-rank-truncation
description: Use when the sample covariance of a high-dimensional transcriptomic matrix
  (p >> n, e.g. 116 obs × 6283 genes) is rank deficient and you need a full-rank empirical
  covariance Φ or a pseudo-true β* to test whether marginal-correlation screening
  recovers true predictive genes.
when_to_use_negative:
- The dataset has n >= p so the sample covariance is full rank and no truncation is
  needed
- The goal is prediction or classification with SuffPCR itself rather than benchmarking
  marginal-correlation screening against a pseudo-true β*
- Features are known to be uncorrelated, so marginal screening is expected to work
  and the counterexample regime does not apply
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: suffpcr
  role: Provides the packaged aml dataset (116 obs × 6283 genes) and the SuffPCR framework
    within which the pseudo-true β* screening benchmark is constructed
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub so its datasets are available for
    covariance computation
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment in which the covariance truncation, Graphical Lasso estimation,
    and FNR evaluation are executed
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:466472364f05263341622295a355930ddbe4d8bbae744d9339330869d24587be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - remotes
  - stats
  - suffpcr
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Empirical covariance rank truncation

## Summary

Constructs a usable empirical covariance from the rank-deficient p>>n sample covariance (e.g. the suffpcr AML data: 116 observations × 6283 genes) by retaining only the top n entries, then combines it with a Graphical-Lasso-estimated sparse precision matrix to form the pseudo-true coefficient vector β* = Σ^-1 Φ used to benchmark marginal-correlation feature screening.

## When to use

Use when the sample covariance of a high-dimensional transcriptomic matrix (p >> n, e.g. 116 obs × 6283 genes) is rank deficient and you need a full-rank empirical covariance Φ or a pseudo-true β* to test whether marginal-correlation screening recovers true predictive genes.

## When NOT to use

- The dataset has n >= p so the sample covariance is full rank and no truncation is needed
- The goal is prediction or classification with SuffPCR itself rather than benchmarking marginal-correlation screening against a pseudo-true β*
- Features are known to be uncorrelated, so marginal screening is expected to work and the counterexample regime does not apply

## Inputs

- High-dimensional gene-expression matrix from a suffpcr packaged dataset (e.g. aml: 116 observations, 6283 genes)
- Empirical sample covariance matrix of the features (rank deficient when p >> n)
- Grid of precision-matrix sparsity levels (e.g. 100%, 99.9%, ..., 95%)

## Outputs

- Truncated empirical covariance matrix Φ (top n entries retained)
- Sparse precision matrix Σ^-1 estimated by Graphical Lasso at each sparsity level
- Pseudo-true coefficient vector β* = Σ^-1 Φ
- Sparsity-versus-FNR results table (fnr_screening_table.csv)

## How to apply

Load the relevant dataset from the suffpcr package (e.g. aml with 116 observations and 6283 genes) and compute the sample covariance matrix. Because p >> n makes this covariance rank deficient, retain only the top n=116 entries to obtain the empirical covariance Φ. Estimate the sparse inverse covariance Σ^-1 with Graphical Lasso across precision-matrix sparsity levels from 100% down to 95% (the extreme-sparse regime, e.g. 99.9% sparsity, is the key diagnostic point). Form the pseudo-true coefficient vector β* = Σ^-1 Φ and evaluate the false negative rate of marginal-correlation screening against β* at each sparsity level, writing the sparsity-versus-FNR table to fnr_screening_table.csv. The rationale is that highly favorable sparse precision conditions still let marginal correlation fail as a screen, motivating the row-sparse-PCA sufficiency approach of SuffPCR.

## Related tools

- **suffpcr** (Provides the packaged aml dataset (116 obs × 6283 genes) and the SuffPCR framework within which the pseudo-true β* screening benchmark is constructed) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub so its datasets are available for covariance computation) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment in which the covariance truncation, Graphical Lasso estimation, and FNR evaluation are executed) — https://github.com/dajmcdon/suffpcr

## Examples

```
remotes::install_github("dajmcdon/suffpcr"); library(suffpcr); data(aml)
```

## Evaluation signals

- The retained empirical covariance Φ contains exactly the top n entries (e.g. n=116 for aml), matching the sample size and restoring invertibility of Σ^-1 Φ
- The reported false negative rate of marginal-correlation screening exceeds 40% even at 99.9% precision-matrix sparsity, matching the Table 1 counterexample result
- At 95% precision-matrix sparsity, the FNR exceeds 95%
- fnr_screening_table.csv contains one FNR value per evaluated sparsity level across the 100% to 95% grid

## Limitations

- The truncation retains only n entries from the rank-deficient p×p sample covariance, so covariance information outside the retained top entries is discarded
- Results depend on the Graphical Lasso sparsity level; the counterexample is demonstrated only across the 100% to 95% sparsity range under highly favorable sparse conditions
- β* is a pseudo-true quantity derived from estimated covariance and precision, not the true population coefficient vector

## Evidence

- [abstract] screening failure rationale: "These methods can fail to select the correct features, predict poorly relative to non-sparse alternatives or ignore any unknown grouping structures for the features."
- [readme] package installation: "You can install the package with

``` r
remotes::install_github("dajmcdon/suffpcr")
```"
- [intro] SuffPCR method description: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
