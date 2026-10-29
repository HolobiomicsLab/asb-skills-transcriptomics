---
name: fisher-information-standard-error-estimation
description: Use when fitting per-gene Tweedie (Compound Poisson) GLMs to scRNA-seq
  counts and Wald-type standard errors, p-values, or confidence intervals are needed
  for regression coefficients after estimating (ϕ, p) by profile likelihood.
when_to_use_negative:
- When only point estimates of β are needed (e.g., effect-size exploration) without
  inference — the full Fisher information computation adds cost without benefit
- When the goal is zero-inflated (ZICP) model inference, which is instead estimated
  via Newton-Raphson rather than this Fisher-information-based scoring step
- When zero-inflation is a property specific only to a subset of genes — the paper
  notes Tweedieverse is not architected for that situation
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Tweedieverse
  role: R/Bioconductor package implementing the per-gene Tweedie GLM fit whose final
    step computes Fisher-information-based standard errors of β̂
  repo: https://github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: Evaluates the Tweedie density (1 < p < 2) via Fourier inversion so the likelihood
    and its curvature can be formed
- name: cplm (R package)
  role: Fits the Compound Poisson GLM (p fixed at 1.5 for initialization, final refit
    at optimized p̂) whose β̂ standard errors are extracted
- name: statmod (R package)
  role: Supports the numerical computation underlying the likelihood/information calculations
- name: R (stats package)
  role: Provides glm() and optim() used to fit the GLM and maximize the profile likelihood
    before SE extraction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:9d7b745e11980c9e7b549ce9c760b42f680af2705e26466fbc0a5299cf263f0e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - R
  - R (stats package)
  - Tweedieverse
  - cplm (R package)
  - statmod (R package)
  - tweedie (R package)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# fisher-information-standard-error-estimation

## Summary

Computes standard errors of the Tweedie GLM coefficient vector (β̂) from the observed Fisher information matrix, whose (j,l)-th entry is (1/ϕ)Σ μ_i^(2−p) X_ij X_il, as the final step of the per-gene Tweedieverse fitting procedure.

## When to use

Use when fitting per-gene Tweedie (Compound Poisson) GLMs to scRNA-seq counts and Wald-type standard errors, p-values, or confidence intervals are needed for regression coefficients after estimating (ϕ, p) by profile likelihood.

## When NOT to use

- When only point estimates of β are needed (e.g., effect-size exploration) without inference — the full Fisher information computation adds cost without benefit
- When the goal is zero-inflated (ZICP) model inference, which is instead estimated via Newton-Raphson rather than this Fisher-information-based scoring step
- When zero-inflation is a property specific only to a subset of genes — the paper notes Tweedieverse is not architected for that situation

## Inputs

- Per-gene scRNA-seq count vector Y_gc with library-size/normalization offset S
- Design matrix X (e.g., univariate binary condition indicator, optionally with covariates)
- Fitted Tweedie GLM coefficients β̂ and optimized parameters (ϕ̂, p̂) from the profile-likelihood loop

## Outputs

- Per-gene coefficient standard errors from the inverted Fisher information matrix
- Wald test statistic with asymptotic χ²(1) p-value per coefficient
- FDR-adjusted q-values used for DE calls at α = 0.05

## How to apply

After the final Compound Poisson GLM refit with the optimized dispersion (ϕ̂) and index (p̂), compute the Fisher information matrix I(β̂) for each gene; its (j,l)-th entry is (1/ϕ)Σ_i μ_i^(2−p) X_ij X_il, i.e. a function of the fitted means, the design matrix, and the two optimized shape parameters. Invert I to obtain the asymptotic covariance matrix and extract square roots of diagonal elements as coefficient standard errors. Note the article's finding that the observed Fisher information is a decreasing function of both ϕ and p, so heavier-tailed or more dispersed genes yield larger variances. These standard errors feed directly into the Wald test statistic, which under the null follows an asymptotic χ²-distribution with 1 degree of freedom, producing p-values that are then FDR-adjusted and thresholded at α = 0.05 for DE calls. Use glm() and optim() from the stats base R package together with the tweedie, cplm, and statmod R packages, as implemented in Tweedieverse.

## Related tools

- **Tweedieverse** (R/Bioconductor package implementing the per-gene Tweedie GLM fit whose final step computes Fisher-information-based standard errors of β̂) — https://github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (Evaluates the Tweedie density (1 < p < 2) via Fourier inversion so the likelihood and its curvature can be formed)
- **cplm (R package)** (Fits the Compound Poisson GLM (p fixed at 1.5 for initialization, final refit at optimized p̂) whose β̂ standard errors are extracted)
- **statmod (R package)** (Supports the numerical computation underlying the likelihood/information calculations)
- **R (stats package)** (Provides glm() and optim() used to fit the GLM and maximize the profile likelihood before SE extraction)

## Examples

```
fit <- cplm::cplm(y ~ condition + offset(log(S)), data = gene_df); se <- sqrt(diag(solve(fisher_info(fit, phi = phi_hat, p = p_hat))))
```

## Evaluation signals

- Confirm the optimization converged and the estimated index parameter p̂ lies in the required interval (1, 2) before computing standard errors
- Check that the (j,l)-th information entry equals (1/ϕ)Σ_i μ_i^(2−p) X_ij X_il for the fitted μ_i — verify against a manual computation on a small gene
- Standard errors should increase as ϕ or p increase, consistent with the observed Fisher information being a decreasing function of both parameters
- Wald statistic under the null should be approximately χ²-distributed with 1 degree of freedom — check via simulated null genes
- FDR-adjusted q-values should reproduce Tweedieverse DE calls at α = 0.05 on benchmark outputs (marginal p-values, adjusted p-values, fold change per feature)

## Limitations

- Standard errors rely on the asymptotic Wald χ²(1) approximation, which may be unreliable for small cell counts or highly sparse genes
- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes
- No shrinkage/borrowed-information estimates exist for improved DE ranking; further research on regularization/pooling of the estimates is untested

## Evidence

- [methods] compute standard errors of β̂ from the Fisher information matrix, whose (j,l)-th entry is (1/ϕ)Σ μ_i^(2−p) X_ij X_il: [evidence span withheld — anchor status: not_found]
- [methods] we calculate the corresponding p-value based on the Wald's test statistic: [evidence span withheld — anchor status: not_found]
