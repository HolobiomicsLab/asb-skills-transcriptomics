---
name: graphical-lasso-precision-matrix-estimation
description: Use when testing whether marginal-correlation feature screening recovers
  predictive genes in p>>n transcriptomic data (e.g. the aml dataset, 116 observations
  x 6283 genes) by benchmarking screening FNR against a graphical-lasso-derived pseudo-true
  β*.
when_to_use_negative:
- The goal is prediction or classification itself (e.g. fitting SuffPCR on X and y
  with suffPCR(X, y, d=1:3)) rather than evaluating or reproducing the screening counterexample.
- The dataset is not p>>n transcriptomic data, so the sample covariance is full-rank
  and marginal-correlation screening is not the object of study.
- You need guaranteed feature recovery under conditions where model assumptions hold;
  the paper's own scope is that 'These methods can fail to select the correct features,
  predict poorly relative to non-sparse alternatives or ignore any unknown grouping
  structures for the features.'
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0203
tools:
- name: suffpcr
  role: R package supplying the packaged aml dataset (116 obs, 6283 genes) and the
    SuffPCR context against which screening FNR is benchmarked
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub so the aml dataset and covariance
    computation are available in R
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Environment in which the empirical covariance Φ, Graphical Lasso precision
    estimation, β* computation, and FNR evaluation are carried out
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:051f8d2616071fd43aaf3894892b3f6f0e9030a8ffc3f310606fe38ed84a0998
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
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# graphical-lasso-precision-matrix-estimation

## Summary

Reproduces the AML counterexample from the SuffPCR paper by estimating a sparse inverse covariance (precision matrix) with Graphical Lasso across sparsity levels, forming the pseudo-true coefficient vector β* = Σ^-1 Φ, and showing marginal-correlation screening has high false negative rates even under highly favorable sparsity assumptions.

## When to use

Use when testing whether marginal-correlation feature screening recovers predictive genes in p>>n transcriptomic data (e.g. the aml dataset, 116 observations x 6283 genes) by benchmarking screening FNR against a graphical-lasso-derived pseudo-true β*.

## When NOT to use

- The goal is prediction or classification itself (e.g. fitting SuffPCR on X and y with suffPCR(X, y, d=1:3)) rather than evaluating or reproducing the screening counterexample.
- The dataset is not p>>n transcriptomic data, so the sample covariance is full-rank and marginal-correlation screening is not the object of study.
- You need guaranteed feature recovery under conditions where model assumptions hold; the paper's own scope is that 'These methods can fail to select the correct features, predict poorly relative to non-sparse alternatives or ignore any unknown grouping structures for the features.'

## Inputs

- aml transcriptomic dataset packaged in suffpcr (116 observations, 6283 genes)
- Empirical covariance matrix Φ computed from the aml data, truncated to the top n=116 entries of the rank-deficient sample covariance
- Grid of precision-matrix sparsity levels from 100% down to 95%

## Outputs

- Estimated sparse inverse covariance Σ^-1 at each Graphical Lasso sparsity level
- Pseudo-true coefficient vector β* = Σ^-1 Φ per sparsity level
- fnr_screening_table.csv: sparsity-versus-FNR results for marginal-correlation screening

## How to apply

Install the suffpcr package from GitHub with remotes::install_github("dajmcdon/suffpcr") and load the packaged aml dataset (116 observations, 6283 genes). Compute the empirical covariance Φ, retaining only the top n=116 entries of the rank-deficient p>>n sample covariance. Estimate the sparse inverse covariance Σ^-1 with Graphical Lasso across precision-matrix sparsity levels from 100% down to 95%, then form the pseudo-true coefficient vector β* = Σ^-1 Φ. Evaluate marginal-correlation screening against β* at each sparsity level, computing the false negative rate, and write the sparsity-versus-FNR results to fnr_screening_table.csv. The rationale is to show that even under extremely favorable, highly sparse precision-matrix conditions, correlation screening misses predictive genes, motivating SuffPCR's row-sparse PCA approach that does not rely on marginal correlation.

## Related tools

- **suffpcr** (R package supplying the packaged aml dataset (116 obs, 6283 genes) and the SuffPCR context against which screening FNR is benchmarked) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub so the aml dataset and covariance computation are available in R) — https://github.com/dajmcdon/suffpcr
- **R** (Environment in which the empirical covariance Φ, Graphical Lasso precision estimation, β* computation, and FNR evaluation are carried out)

## Examples

```
remotes::install_github("dajmcdon/suffpcr"); library(suffpcr); data(aml); S <- cov(aml$X)
```

## Evaluation signals

- At 99.9% precision-matrix sparsity, the computed screening false negative rate exceeds 40%, matching the Table 1 AML result.
- At 95% precision-matrix sparsity, the FNR exceeds 95%.
- The empirical covariance Φ uses only the top n=116 entries of the rank-deficient sample covariance (n=116 observations, p=6283 genes).
- The sparsity-versus-FNR results are written to fnr_screening_table.csv with one row per sparsity level from 100% down to 95%.
- The installed package passes R-CMD-check, and data(aml) loads an object with 6283 genes across 116 observations.

## Limitations

- The sample covariance is rank-deficient in the p>>n regime (6283 genes, 116 observations), so Φ must be truncated to the top n=116 entries; the counterexample is specific to this regime.
- The result is a counterexample under deliberately favorable (99.9% to 95% sparse) precision-matrix assumptions; it does not by itself characterize screening performance under arbitrary conditions.
- The suffpcr repository has no changelog, so version provenance of the packaged aml data should be checked separately.

## Evidence

- [other] screening counterexample thresholds: [evidence span withheld — anchor status: not_found]
- [abstract] motivation from abstract: "These methods can fail to select the correct features, predict poorly relative to non-sparse alternatives or ignore any unknown grouping structures for the features."
- [intro] SuffPCR alternative approach: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [readme] installation: "You can install the package with

``` r
remotes::install_github("dajmcdon/suffpcr")
```"
