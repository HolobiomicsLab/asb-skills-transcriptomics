---
name: variational-bayes-model-fitting
description: Use when fitting a Bayesian trajectory model whose posterior is intractable
  or multi-modal — e.g. PhenoPath on scRNA-seq with a stimulation covariate — and
  a fast, scalable approximate posterior is needed instead of MCMC.
when_to_use_negative:
- Features are expected to evolve nonlinearly with pseudotime — PhenoPath assumes
  linear evolution and nonlinearity must be checked separately
- The data contain complex mixtures of continuous and discrete covariates in high-dimensional
  settings, which current PhenoPath cannot handle (deferred to future work)
- You need exact posterior inference rather than a mean-field approximation, since
  CAVI uses a full-factorised approximation that minimises KL-divergence to the true
  posterior
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: PhenoPath
  role: The R/Bioconductor package whose phenopath() function performs the CAVI variational
    Bayesian fit of the trajectory-covariate model
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: Rcpp
  role: C++ backend that implements the coordinate ascent variational inference for
    increased performance
- name: R
  role: Environment in which the PhenoPath variational inference is run and results
    are extracted and exported
- name: SummarizedExperiment
  role: Bioconductor container for the expression matrix input to the fit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:085e196e86c8149ff0285a51daf91400d407ac66a1119c83439274ba6348c3db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle 2
  - PhenoPath
  - R
  - Rcpp
  - SummarizedExperiment
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Variational Bayes model fitting

## Summary

Fits PhenoPath's Bayesian latent-variable trajectory model using coordinate ascent mean-field variational inference (CAVI, implemented in Rcpp), jointly inferring pseudotime and covariate-interaction effects from single-cell or bulk expression data.

## When to use

Use when fitting a Bayesian trajectory model whose posterior is intractable or multi-modal — e.g. PhenoPath on scRNA-seq with a stimulation covariate — and a fast, scalable approximate posterior is needed instead of MCMC.

## When NOT to use

- Features are expected to evolve nonlinearly with pseudotime — PhenoPath assumes linear evolution and nonlinearity must be checked separately
- The data contain complex mixtures of continuous and discrete covariates in high-dimensional settings, which current PhenoPath cannot handle (deferred to future work)
- You need exact posterior inference rather than a mean-field approximation, since CAVI uses a full-factorised approximation that minimises KL-divergence to the true posterior

## Inputs

- Logged gene expression matrix (single-cell or bulk), e.g. Salmon re-quantified counts for 820 dendritic cells restricted to the 7500 most variable genes, on a scale comparable to log2(TPM+1)
- Sample-specific covariate vector x (e.g. binary LPS vs PAM stimulation status)
- Initialisation option for the latent space (z_init = 1 for PCA-based initialisation)
- ELBO tolerance and ELBO thinning interval parameters

## Outputs

- Fitted PhenoPath model object containing posterior-mean pseudotimes (fit$m_z, retrievable via trajectory(fit))
- ELBO trace for convergence checking via plot_elbo(fit)
- Per-cell posterior-mean pseudotimes written to CSV
- Posterior estimates of covariate-pseudotime interaction effects (rankable via significant_interactions(phenopath_fit, n))

## How to apply

Provide PhenoPath's phenopath() function with a logged expression matrix (e.g. 7500 most variable genes from the 820 dendritic cells of GSE48963, on a scale comparable to log2(TPM+1)) and a covariate vector x encoding sample-specific factors such as LPS vs PAM stimulation status; capture times are deliberately excluded from the model input. Inference is performed by coordinate ascent mean-field variational inference with an ELBO tolerance and ELBO thinning interval, and because the posterior is multi-modal, initialise the latent space on the first principal component (z_init = 1). Verify convergence by checking that the ELBO trace is approximately flat via plot_elbo(fit). Extract posterior-mean pseudotimes per cell with trajectory(fit), which retrieves fit$m_z, then validate externally (e.g. Pearson correlation against physical capture times, expected R2 = 0.68 on the dendritic-cell data, exceeding Monocle 2, DPT and TSCAN).

## Related tools

- **PhenoPath** (The R/Bioconductor package whose phenopath() function performs the CAVI variational Bayesian fit of the trajectory-covariate model) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **Rcpp** (C++ backend that implements the coordinate ascent variational inference for increased performance)
- **R** (Environment in which the PhenoPath variational inference is run and results are extracted and exported)
- **SummarizedExperiment** (Bioconductor container for the expression matrix input to the fit)

## Examples

```
fit <- phenopath(Y, x, z_init = 1, elbo_tol = 1e-5, thin = 20); plot_elbo(fit); pseudotime <- trajectory(fit)
```

## Evaluation signals

- ELBO trace from plot_elbo(fit) is approximately flat, indicating the coordinate ascent variational inference has converged
- trajectory(fit) returns fit$m_z with one posterior-mean pseudotime per cell (e.g. 820 values for the dendritic-cell dataset)
- Pearson correlation of inferred pseudotimes against held-out physical capture times gives R2 = 0.68 on the Shalek dendritic-cell data, exceeding Monocle 2, DPT and TSCAN
- For every input gene set size, the fitted pseudotimes report a higher correlation with capture time than the competing methods
- Latent space is initialised on the first principal component (z_init = 1) to mitigate the multi-modal posterior; different initialisations should be checked for consistency

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects may occur and model fits should be verified for consistency with the data
- The posterior is multi-modal, so results depend on initialisation (z_init = 1, first principal component, is recommended)
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings
- CAVI uses a full-factorised mean-field approximation, so posterior correlations between latent variables are not captured

## Evidence

- [methods] Coordinate ascent mean field variational inference: "We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form"
- [readme] Inference implemented with Rcpp: [evidence span withheld — anchor status: external_doc]
- [other] Multi-modal posterior and PCA initialisation: [evidence span withheld — anchor status: not_found]
- [discussion] Linear evolution assumption: "An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime."
- [discussion] Model fit verification caveat: "However, it cannot be discounted that nonlinear effects may occur and checks should be conducted to verify that PhenoPath model fits are consistent with the data."
