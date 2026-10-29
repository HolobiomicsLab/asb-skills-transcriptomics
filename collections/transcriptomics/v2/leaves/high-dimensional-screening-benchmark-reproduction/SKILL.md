---
name: high-dimensional-screening-benchmark-reproduction
description: 'Use when you need to demonstrate or benchmark that marginal-correlation
  screening fails to recover predictive genes in a high-dimensional transcriptomic
  setting where p >> n (e.g., AML: 116 samples, 6283 genes).'
when_to_use_negative:
- Your goal is to build a predictive model on the data rather than to evaluate/benchmark
  a screening rule — SuffPCR itself (row-sparse PCA plus OLS or logistic regression)
  is the fitting method, this skill is the counterexample benchmark
- The feature dimension is not substantially larger than n, so the sample covariance
  is not rank-deficient and the top-n-entries covariance truncation of Φ is not required
- You already have ground-truth coefficients from a known simulation design, so the
  pseudo-true β* via precision-matrix estimation is unnecessary
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: suffpcr
  role: Provides the packaged aml dataset (116 obs, 6283 genes) used as the counterexample
    input
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment in which the package is installed and the covariance/precision-matrix
    benchmark is executed
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs suffpcr from GitHub via remotes::install_github("dajmcdon/suffpcr")
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:ffdb069c3016fb177cdf9ea5e53499f310a33a714584296a178f39ee95a709fe
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# high-dimensional-screening-benchmark-reproduction

## Summary

Reproduces the AML counterexample from the suffPCR paper, quantifying the false negative rate of marginal-correlation (SPC-style) gene screening against a pseudo-true coefficient vector β* = Σ^-1 Φ under increasingly sparse Graphical Lasso estimates of the precision matrix.

## When to use

Use when you need to demonstrate or benchmark that marginal-correlation screening fails to recover predictive genes in a high-dimensional transcriptomic setting where p >> n (e.g., AML: 116 samples, 6283 genes).

## When NOT to use

- Your goal is to build a predictive model on the data rather than to evaluate/benchmark a screening rule — SuffPCR itself (row-sparse PCA plus OLS or logistic regression) is the fitting method, this skill is the counterexample benchmark
- The feature dimension is not substantially larger than n, so the sample covariance is not rank-deficient and the top-n-entries covariance truncation of Φ is not required
- You already have ground-truth coefficients from a known simulation design, so the pseudo-true β* via precision-matrix estimation is unnecessary

## Inputs

- aml dataset packaged in suffpcr (116 observations, 6283 genes, transcriptomic expression matrix)
- precision-matrix sparsity level grid (100% down to 95%)

## Outputs

- fnr_screening_table.csv with sparsity level versus screening false negative rate
- per-sparsity false negative rate of marginal-correlation screening against β* = Σ^-1 Φ

## How to apply

Install the suffpcr package from GitHub (remotes::install_github("dajmcdon/suffpcr")) and load the packaged aml dataset with 116 observations and 6283 genes. Compute the empirical covariance matrix Φ, retaining only the top n=116 entries of the rank-deficient p>>n sample covariance. Estimate the sparse inverse covariance Σ^-1 with Graphical Lasso across precision-matrix sparsity levels from 100% down to 95%, then form the pseudo-true coefficient vector β* = Σ^-1 Φ. Evaluate marginal-correlation feature screening against β* at each sparsity level, computing the screening false negative rate, and write the sparsity-versus-FNR table to fnr_screening_table.csv. The reproduction is correct if the FNR exceeds 40% even at 99.9% precision-matrix sparsity (matching Table 1), and exceeds 95% at 95% sparsity.

## Related tools

- **suffpcr** (Provides the packaged aml dataset (116 obs, 6283 genes) used as the counterexample input) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment in which the package is installed and the covariance/precision-matrix benchmark is executed) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs suffpcr from GitHub via remotes::install_github("dajmcdon/suffpcr")) — https://github.com/dajmcdon/suffpcr

## Examples

```
remotes::install_github("dajmcdon/suffpcr"); library(suffpcr); data(aml)
```

## Evaluation signals

- False negative rate of marginal-correlation screening exceeds 40% even when the precision matrix is estimated at 99.9% sparsity, matching Table 1
- FNR exceeds 95% at 95% precision-matrix sparsity, showing screening degrades further as sparsity relaxes
- The empirical covariance Φ is rank-deficient (p=6283 >> n=116) and exactly the top n=116 entries are retained
- fnr_screening_table.csv contains one FNR per sparsity level spanning 100% down to 95%
- β* is formed as the product Σ^-1 Φ from the Graphical Lasso precision estimate and the truncated covariance

## Limitations

- The benchmark requires p >> n so the sample covariance is rank-deficient; the pseudo-true β* construction depends on this regime
- The counterexample is specific to screening genes by marginal correlation with the phenotype (as in SPC) and does not generalize to all feature-selection methods
- Results depend on the Graphical Lasso precision-matrix sparsity grid chosen (100%–95%); other grids would yield different FNR trajectories

## Evidence

- [results] In the AML counterexample, even when the precision matrix is estimated at 99.9% sparsity, marginal-correlation screening has a false negative rate over 40%: [evidence span withheld — anchor status: not_found]
- [methods] Installation of suffpcr from GitHub: "remotes::install_github("dajmcdon/suffpcr")"
- [readme] README installation instructions: "remotes::install_github("dajmcdon/suffpcr")
```

## Example"
- [abstract] Sparse linear methods can fail to select correct features: "These methods can fail to select the correct features, predict poorly relative to non-sparse alternatives or ignore any unknown grouping structures for the features."
