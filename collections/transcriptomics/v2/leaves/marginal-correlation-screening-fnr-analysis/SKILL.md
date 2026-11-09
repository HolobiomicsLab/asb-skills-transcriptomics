---
name: marginal-correlation-screening-fnr-analysis
description: Use when evaluating whether marginal-correlation (SPC-style) feature
  screening can recover predictive genes in a high-dimensional transcriptomic dataset
  (p >> n), by benchmarking its false negative rate against a pseudo-true coefficient
  vector.
when_to_use_negative:
- The goal is prediction or feature selection on new data with SuffPCR itself, rather
  than benchmarking the failure of marginal-correlation screening against a pseudo-true
  β*
- The dataset is not high-dimensional (p >> n) transcriptomic data, so the rank-deficient
  sample covariance and its top n=116 entries cannot be formed as in the AML setup
- A true coefficient vector is already known analytically (e.g. simulation), making
  the Graphical Lasso pseudo-true approximation β* = Σ^-1 Φ unnecessary
tools:
- name: suffpcr
  role: provides the aml dataset used as the counterexample input for the screening
    FNR analysis
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: installs the suffpcr package from GitHub to access the aml data
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: analysis environment for computing Φ, running Graphical Lasso, forming β*
    = Σ^-1 Φ, and computing screening FNRs
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:b1df0642a80d8b819e9ec5700b02be75a86e22950a6705659ff34ccf4b624b6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - remotes
  - stats
  - suffpcr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# marginal-correlation-screening-fnr-analysis

## Summary

Reproduces the AML counterexample from the SuffPCR paper showing that marginal-correlation screening (as in SPC) misses >40% of true genes even when the precision matrix is estimated at 99.9% sparsity under highly favorable conditions.

## When to use

Use when evaluating whether marginal-correlation (SPC-style) feature screening can recover predictive genes in a high-dimensional transcriptomic dataset (p >> n), by benchmarking its false negative rate against a pseudo-true coefficient vector.

## When NOT to use

- The goal is prediction or feature selection on new data with SuffPCR itself, rather than benchmarking the failure of marginal-correlation screening against a pseudo-true β*
- The dataset is not high-dimensional (p >> n) transcriptomic data, so the rank-deficient sample covariance and its top n=116 entries cannot be formed as in the AML setup
- A true coefficient vector is already known analytically (e.g. simulation), making the Graphical Lasso pseudo-true approximation β* = Σ^-1 Φ unnecessary

## Inputs

- aml dataset from the suffpcr R package (116 observations, 6283 genes)
- empirical covariance matrix Φ with only the top n=116 entries of the rank-deficient sample covariance retained
- sequence of precision-matrix sparsity levels (100% down to 95%) for Graphical Lasso

## Outputs

- fnr_screening_table.csv: false negative rate of marginal-correlation screening at each precision-matrix sparsity level
- pseudo-true coefficient vector β* = Σ^-1 Φ used as the screening benchmark

## How to apply

Install the suffpcr package from GitHub (remotes::install_github("dajmcdon/suffpcr")) and load the packaged aml dataset (116 observations, 6283 genes). Compute the empirical covariance matrix Φ from the rank-deficient p >> n sample covariance, retaining only the top n=116 entries. Estimate the sparse inverse covariance Σ^-1 with Graphical Lasso across precision-matrix sparsity levels from 100% down to 95%, then form the pseudo-true coefficient vector β* = Σ^-1 Φ. Evaluate marginal-correlation feature screening against β* at each sparsity level, computing the false negative rate of screening, and write the sparsity-versus-FNR table to fnr_screening_table.csv. The rationale is that this counterexample demonstrates screening failure under extremely favorable sparse precision estimation, motivating subspace methods like SuffPCR that estimate sparse principal components followed by ordinary least squares or logistic regression.

## Related tools

- **suffpcr** (provides the aml dataset used as the counterexample input for the screening FNR analysis) — https://github.com/dajmcdon/suffpcr
- **remotes** (installs the suffpcr package from GitHub to access the aml data) — https://github.com/dajmcdon/suffpcr
- **R** (analysis environment for computing Φ, running Graphical Lasso, forming β* = Σ^-1 Φ, and computing screening FNRs) — https://github.com/dajmcdon/suffpcr

## Examples

```
remotes::install_github("dajmcdon/suffpcr"); library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- Validation: the reported FNR exceeds 40% even at 99.9% precision-matrix sparsity, matching the Table 1 result
- At 95% precision-matrix sparsity, the FNR of marginal-correlation screening exceeds 95%
- fnr_screening_table.csv contains FNR values across the full sparsity sweep from 100% down to 95%
- The aml data loaded from suffpcr has dimensions 116 observations × 6283 genes before covariance computation

## Limitations

- The sample covariance is rank-deficient because p >> n (6283 genes vs 116 observations), so only the top n=116 entries can be retained, which constrains the pseudo-true β* construction
- The result is a counterexample computed under highly favorable, extremely sparse precision estimation; it demonstrates screening failure in this AML setting rather than a general FNR law
- The demonstration motivates SuffPCR (sparse PCA followed by OLS or logistic regression) but does not itself establish SuffPCR's predictive performance

## Evidence

- [intro] The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression.: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [abstract] These methods can fail to select the correct features, predict poorly relative to non-sparse alternatives or ignore any unknown grouping structures for the features.: "These methods can fail to select the correct features, predict poorly relative to non-sparse alternatives or ignore any unknown grouping structures for the features."
- [methods] remotes::install_github("dajmcdon/suffpcr"): "remotes::install_github("dajmcdon/suffpcr")"
- [abstract] SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace.: "SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace."
- [readme] The goal of suffpcr is to estimate sufficient principal component regression: "The goal of suffpcr is to estimate sufficient principal component regression
by implementing row-sparse PCA followed by ordinary least squares or logistic
regression."
