---
name: elbow-criterion-sparsity-selection
description: Use when a Fantope/sparse PCA loading matrix must be made row-sparse
  and the hard-threshold t on row ℓ2 norms cannot be set by hand — e.g. estimated
  subspace V̂d rows must be split into signal vs noise without a known sparsity level.
when_to_use_negative:
- The true sparsity level or loading support is already known and a fixed threshold
  on row norms is defensible without data-driven selection
- The loading estimate is not row-structured (e.g. dense PCA scores or elementwise-sparse
  solutions), so row-norm distributions have no signal/noise separation to exploit
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: suffpcr
  role: R package whose Algorithm 1 (row-sparse FPS with AIRLB) and Algorithm 2 (elbow
    threshold selection) this skill reconstructs; provides suffPCR() and coef.suffpcr
    for reference comparison
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Implementation environment for the ADMM/Fantope reimplementation, elbow criterion,
    and benchmark generation
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub for reference-output comparison
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:15dfed474500778795c2f8c0047f82c63c4bdb13f2314c8ff17155d970bf8493
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - remotes
  - suffpcr
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# elbow-criterion-sparsity-selection

## Summary

Algorithm 2 of SuffPCR: an automatic 'elbow' thresholding rule that separates signal rows from noise rows in a loading matrix by maximizing the difference of empirical derivatives of observation-weighted row-norm variances on either side of a candidate cutoff.

## When to use

Use when a Fantope/sparse PCA loading matrix must be made row-sparse and the hard-threshold t on row ℓ2 norms cannot be set by hand — e.g. estimated subspace V̂d rows must be split into signal vs noise without a known sparsity level.

## When NOT to use

- The true sparsity level or loading support is already known and a fixed threshold on row norms is defensible without data-driven selection
- The loading estimate is not row-structured (e.g. dense PCA scores or elementwise-sparse solutions), so row-norm distributions have no signal/noise separation to exploit

## Inputs

- Estimated loading matrix V̂d (rows = features, columns = principal dimensions d) from the row-sparse FPS/ADMM step of SuffPCR
- Optional: the full data matrix X and phenotype y used to fit V̂d (observation weighting of row-norm variances)

## Outputs

- Automatically selected hard threshold t on row ℓ2 norms
- Row-sparse loading matrix V̂d with noise rows hard-thresholded to zero
- Zero/nonzero row-pattern assignment (signal rows vs noise rows) used downstream in the OLS/logistic regression on the recovered subspace

## How to apply

Compute the ℓ2 norm of every row of the estimated loading matrix V̂d (the FPS/Fantope solution from Algorithm 1). For each candidate threshold t, compute the empirical derivative of the observation-weighted variances of the row norms on each side of t, and select the t that maximizes the difference between these two quantities — the 'elbow' that separates signal rows from noise rows. Apply the chosen t as a hard threshold: zero out rows of V̂d whose norm falls below it (Steps 9–11 of Algorithm 1). Validate on synthetic benchmarks (e.g. n=100, p=50, unit-norm V with 5 nonzero entries, X = 5*tcrossprod(U,V) + 0.1*Gaussian noise) by confirming the recovered subspace is row-sparse with the correct 5 nonzero rows.

## Related tools

- **suffpcr** (R package whose Algorithm 1 (row-sparse FPS with AIRLB) and Algorithm 2 (elbow threshold selection) this skill reconstructs; provides suffPCR() and coef.suffpcr for reference comparison) — https://github.com/dajmcdon/suffpcr
- **R** (Implementation environment for the ADMM/Fantope reimplementation, elbow criterion, and benchmark generation) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub for reference-output comparison) — https://github.com/dajmcdon/suffpcr

## Examples

```
remotes::install_github("dajmcdon/suffpcr"); library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- For each d in 1:3, V̂d after thresholding matches the package output (coef.suffpcr) to within floating-point noise on the synthetic benchmark (n=100, p=50)
- The thresholded subspace is exactly row-sparse with the 5 signal rows retained and 45 noise rows zeroed on the benchmark where V has 5 nonzero entries
- Zero/nonzero row patterns agree between the reimplementation and the installed suffpcr package (recorded in reimplementation_agreement.csv)
- Downstream β̂ from the linear model on the recovered subspace agrees with the package coefficients per dimension
- The selected t corresponds to a clear maximum in the difference of empirical derivatives of observation-weighted row-norm variances (a visible elbow in the row-norm distribution)

## Limitations

- The elbow criterion assumes a separable distribution of row norms (signal rows clearly larger than noise rows); with weak signal or highly correlated features the elbow may be ambiguous
- Threshold choice directly determines the final row sparsity, so a mis-detected elbow propagates to the downstream regression on the recovered subspace
- Only supports hard row-thresholding of V̂d; elementwise sparsity or group structures are not handled by this criterion

## Evidence

- [methods] row-sparse via hard-thresholding: [evidence span withheld — anchor status: not_found]
- [methods] Algorithm 2 elbow selection: [evidence span withheld — anchor status: not_found]
- [intro] suffPCR pipeline definition: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [abstract] sparse subspace rationale: "Because the estimated subspace is sparse in the features, the resulting predictions will depend on only a small subset of genes."
- [readme] README benchmark example: [evidence span withheld — anchor status: not_found]
