---
name: fourier-inversion-density-evaluation
description: Use when fitting Tweedie (CPLM) or zero-inflated Tweedie (ZICP) models
  to scRNA-seq counts with the index parameter p constrained to (1, 2), where the
  Tweedie density is semi-continuous with a point mass at zero and its normalizing
  constant must be computed numerically.
when_to_use_negative:
- The response distribution is not Tweedie with index p in (1, 2) — for special cases
  with closed-form densities (e.g., Poisson or Gamma limits), numerical Fourier inversion
  is unnecessary
- The model requires gene-specific zero-inflation affecting only a subset of genes,
  which the article states Tweedieverse is not architected for
- Data are continuous measurements where a non-semi-continuous (non-CPLM) likelihood
  is appropriate
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: tweedie (R package)
  role: Carries out the Fourier-inversion computation of the Tweedie density for 1
    < p < 2
- name: cplm (R package)
  role: Fits the initial Compound Poisson GLM (p = 1.5) that supplies the beta initialization
    around which densities are evaluated
- name: statmod (R package)
  role: Supporting computation for the CPLM/ZICP likelihood evaluation
- name: Tweedieverse
  role: Framework that embeds the Fourier-inversion density evaluation in its per-gene
    CPLM/ZICP fitting loop with profile-likelihood and Newton-Raphson optimization
  repo: https://github.com/himelmallick/Tweedieverse
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:d4d55451a3d0ed0993387153f8828d63fb793e5da52461de19f41da17bd8bc5d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - Bioconductor
  - R
  - SCRAN
  - Tweedieverse
  - cplm (R package)
  - statmod (R package)
  - tweedie (R package)
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Fourier-inversion Tweedie density evaluation

## Summary

Numerically evaluates the Tweedie density function for 1 < p < 2 via the Fourier inversion strategy implemented in the R package tweedie, because the normalizing constant has no closed form. This is the likelihood-evaluation engine underlying CPLM and ZICP fits in the Tweedieverse framework.

## When to use

Use when fitting Tweedie (CPLM) or zero-inflated Tweedie (ZICP) models to scRNA-seq counts with the index parameter p constrained to (1, 2), where the Tweedie density is semi-continuous with a point mass at zero and its normalizing constant must be computed numerically.

## When NOT to use

- The response distribution is not Tweedie with index p in (1, 2) — for special cases with closed-form densities (e.g., Poisson or Gamma limits), numerical Fourier inversion is unnecessary
- The model requires gene-specific zero-inflation affecting only a subset of genes, which the article states Tweedieverse is not architected for
- Data are continuous measurements where a non-semi-continuous (non-CPLM) likelihood is appropriate

## Inputs

- Per-gene scRNA-seq count vector (UMI or read counts) for a single feature across cells
- Modeled mean vector mu from the log-link GLM log(mu) = X*beta + log(S), with library size S as offset
- Candidate dispersion (phi) and index (p) parameter values, with phi > 0 and p in (1, 2)
- Design matrix X (e.g., intercept plus binary group indicator for DE testing)

## Outputs

- Numerically evaluated Tweedie density values (and hence the per-gene log-likelihood) at each candidate (mu, phi, p)
- Converged per-gene parameter estimates (phi_hat, p_hat) with p_hat in (1, 2), used by CPLM/ZICP downstream inference

## How to apply

For each per-gene fit, evaluate the Tweedie density at the observed counts using the Fourier inversion method implemented in the R package tweedie, since the normalizing constant has no closed form when 1 < p < 2 (the Compound Poisson-Gamma regime with a point mass at zero). Feed these density evaluations into the log-likelihood used by the compound Poisson GLM: initialize beta by fitting a Compound Poisson GLM with the index parameter fixed at p = 1.5 (cplm R package), then optimize theta = (phi, p) by profiling beta out of the log-likelihood (CPLM, via optim()) or by Newton-Raphson (ZICP), subject to phi > 0 and p in (1, 2). The library size or normalization factor S enters as an offset in the log link log(mu) = X*beta + log(S), so the density is evaluated at the modeled mean mu for each cell. Re-evaluate the density at each iteration of the alternating theta/beta updates until convergence, then refit the final GLM at (phi_hat, p_hat) for inference.

## Related tools

- **tweedie (R package)** (Carries out the Fourier-inversion computation of the Tweedie density for 1 < p < 2)
- **cplm (R package)** (Fits the initial Compound Poisson GLM (p = 1.5) that supplies the beta initialization around which densities are evaluated)
- **statmod (R package)** (Supporting computation for the CPLM/ZICP likelihood evaluation)
- **Tweedieverse** (Framework that embeds the Fourier-inversion density evaluation in its per-gene CPLM/ZICP fitting loop with profile-likelihood and Newton-Raphson optimization) — https://github.com/himelmallick/Tweedieverse

## Examples

```
library(tweedie); loglik <- sum(dtweedie(y, mu = fitted_mu, phi = phi, power = p, log = TRUE))
```

## Evaluation signals

- Every optimized index parameter estimate p_hat falls within the constrained interval (1, 2), per the model's stated constraints
- Dispersion estimates satisfy phi > 0 across all fitted genes
- The optimization (profile likelihood via optim() for CPLM, Newton-Raphson for ZICP) reports convergence for each fitted gene
- Per-gene fits initialized at p = 1.5 (Compound Poisson GLM via cplm) reproduce consistent likelihood values with the final refit at (phi_hat, p_hat)
- Evaluated densities reflect the semi-continuous Tweedie form with a point mass at zero for 1 < p < 2

## Limitations

- The Tweedie normalizing constant has no closed form for 1 < p < 2, so density evaluation relies on numerical Fourier inversion, which is computationally demanding for per-gene scans across many datasets
- Numerical approximation error in Fourier inversion propagates into likelihood, parameter estimates, and Wald-test p-values
- Only the Compound Poisson-Gamma regime (1 < p < 2) needs this treatment; other Tweedie regimes were not systematically studied in this context

## Evidence
