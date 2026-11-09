---
name: covariate-encoded-trajectory-modelling
description: Use when a cross-sectional single-cell expression cohort carries a sample-level
  covariate (stimulant, mutation, drug) that may modulate progression, and pseudotime
  must be inferred without explicit time-series labels.
when_to_use_negative:
- The population background is homogeneous with no sample-level covariate of interest
  — existing pseudotime methods that assume homogeneous backgrounds may suffice
- Features are expected to evolve strongly nonlinearly with pseudotime, since PhenoPath
  assumes features evolve linearly with respect to pseudotime and nonlinear effects
  require additional model-fit checks
- A complex high-dimensional mixture of continuous and discrete covariates is needed
  — the current PhenoPath defers this to future work
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PhenoPath
  role: Core variational Bayesian model that infers pseudotimes and covariate–trajectory
    interactions from expression data plus covariate vector
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: Monocle 2
  role: Baseline pseudotime method for comparison against PhenoPath's covariate-aware
    pseudotimes
- name: R
  role: Implementation language for the phenopath() fitting, ELBO inspection, trajectory
    extraction and validation workflow
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: SummarizedExperiment
  role: Bioconductor container for the expression matrix input
- name: dplyr
  role: Listed Imports dependency used for data manipulation in the R workflow
- name: ggplot2
  role: Listed Imports dependency used for visualisation (e.g. ELBO trace, pseudotime
    plots)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:4fa1dba7f8c86030825b177e344d9eeb66e2102c1337cbe17d9e68c87907b8c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle 2
  - PhenoPath
  - R
  - SummarizedExperiment
  - dplyr
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# covariate-encoded-trajectory-modelling

## Summary

Infers pseudotemporal cell trajectories in the presence of heterogeneous backgrounds by explicitly encoding covariates (e.g. LPS vs PAM stimulation status) in PhenoPath, a hybrid regression–latent variable Bayesian model fitted via coordinate ascent mean-field variational inference.

## When to use

Use when a cross-sectional single-cell expression cohort carries a sample-level covariate (stimulant, mutation, drug) that may modulate progression, and pseudotime must be inferred without explicit time-series labels.

## When NOT to use

- The population background is homogeneous with no sample-level covariate of interest — existing pseudotime methods that assume homogeneous backgrounds may suffice
- Features are expected to evolve strongly nonlinearly with pseudotime, since PhenoPath assumes features evolve linearly with respect to pseudotime and nonlinear effects require additional model-fit checks
- A complex high-dimensional mixture of continuous and discrete covariates is needed — the current PhenoPath defers this to future work

## Inputs

- Gene-by-cell expression matrix (e.g. Salmon re-quantified counts for 820 dendritic cells, top 7500 most variable genes, logged scale comparable to log2(TPM+1))
- Sample-level binary or categorical covariate vector (e.g. LPS vs PAM stimulation status), one entry per cell

## Outputs

- Posterior-mean pseudotime per cell (fit$m_z via trajectory(fit)), exportable as per-cell CSV
- ELBO convergence trace (inspectable with plot_elbo(fit))
- Ranked covariate–pathway interaction effects (significant_interactions(phenopath_fit, n))

## How to apply

Load a quantified expression matrix (e.g. Salmon re-quantified counts for the 820 Shalek dendritic cells, GSE48963), retain the ~7500 most variable genes on a logged scale comparable to log2(TPM+1), and encode the binary covariate (LPS vs PAM) as vector x, deliberately excluding physical capture times. Fit the model with phenopath() using coordinate ascent mean-field variational inference, initialising the latent space on the first principal component (z_init = 1) because the posterior is multi-modal. Verify convergence by confirming the ELBO trace is approximately flat with plot_elbo(fit). Extract posterior-mean pseudotimes with trajectory(fit) (which returns fit$m_z) and validate externally by correlating against capture times; on the Shalek dendritic-cell benchmark this yields R2 = 0.68, exceeding Monocle 2, DPT and TSCAN. Significant covariate–pathway interactions can then be ranked/labelled via the posterior-mean-over-posterior-standard-deviation criterion using significant_interactions(phenopath_fit, n).

## Related tools

- **PhenoPath** (Core variational Bayesian model that infers pseudotimes and covariate–trajectory interactions from expression data plus covariate vector) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **Monocle 2** (Baseline pseudotime method for comparison against PhenoPath's covariate-aware pseudotimes)
- **R** (Implementation language for the phenopath() fitting, ELBO inspection, trajectory extraction and validation workflow) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **SummarizedExperiment** (Bioconductor container for the expression matrix input)
- **dplyr** (Listed Imports dependency used for data manipulation in the R workflow)
- **ggplot2** (Listed Imports dependency used for visualisation (e.g. ELBO trace, pseudotime plots))

## Examples

```
fit <- phenopath(Y, x, z_init = 1); plot_elbo(fit); pseudotime <- trajectory(fit)
```

## Evaluation signals

- ELBO trace from the coordinate ascent variational fit is approximately flat (plot_elbo(fit)), indicating convergence of the mean-field approximation
- Pearson correlation of inferred pseudotimes against held-out physical capture times reaches R2 = 0.68 on the Shalek dendritic-cell data (GSE48963)
- For every input gene set size, PhenoPath reports a higher correlation with capture time than Monocle 2, DPT and TSCAN
- Pseudotimes were inferred with capture times deliberately excluded from the model input, so validation correlation is a genuine external check
- Per-cell posterior-mean pseudotimes (fit$m_z) are retrieved reproducibly via trajectory(fit) and written to CSV

## Limitations

- An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime; nonlinear effects cannot be discounted and model fits should be checked for consistency with the data
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings (deferred to future work)
- The posterior is multi-modal, so latent-space initialisation matters — z_init = 1 (first principal component) is used to mitigate this
- Application to high-throughput single-cell CRISPR experiments remains unexplored

## Evidence

- [abstract] We describe a novel statistical framework that learns how pseudotime trajectories can be modulated through covariates that encode such factors.: "We describe a novel statistical framework that learns how pseudotime trajectories can be modulated through covariates that encode such factors."
- [methods] CAVI inference: "We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form"
- [discussion] An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime.: "An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime."
- [readme] README: covariates and interactions: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates"
- [readme] README: CAVI via Rcpp: [evidence span withheld — anchor status: external_doc]
