---
name: variational-inference-convergence-monitoring
description: Use when fitting PhenoPath to single-cell or bulk expression matrices
  (e.g. log2(TPM+1) TCGA-BRCA data) and you must confirm the variational posterior
  has converged before extracting interaction effects or trajectories.
when_to_use_negative:
- You are running a different pseudotime method (Monocle 2, DPT, Wishbone, TSCAN)
  that does not use variational inference and exposes no ELBO to monitor
- You only need a quick exploratory trajectory and will discard the fit — skipping
  convergence checks is acceptable, but the fit must never be used for published interaction
  rankings without verifying the flat ELBO trace
- The model has already been verified as converged and you are only re-extracting
  posteriors from the saved fit object
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: PhenoPath
  role: Provides phenopath() with elbo_tol/thin convergence control, plot_elbo() for
    ELBO trace visualisation, and CAVI inference implemented in Rcpp
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: Rcpp
  role: Backend library that implements the co-ordinate ascent variational inference
    whose convergence is being monitored
- name: R
  role: Runtime environment for invoking phenopath() and plot_elbo() during fit and
    convergence checking
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:c76dea875f63cbec4261da6a5e38adb1a9f6a3c1c4bf6cb5b4a4017cd0c951e1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle 2
  - PhenoPath
  - R
  - Rcpp
  - dplyr
  - ggplot2
  - tidyr
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# variational-inference-convergence-monitoring

## Summary

Monitors convergence of PhenoPath's co-ordinate ascent mean field variational inference (CAVI) by computing the ELBO at regular iteration intervals, stopping when the ELBO change falls below a tolerance, and visually verifying a flat ELBO trace with plot_elbo(fit) before trusting the fit.

## When to use

Use when fitting PhenoPath to single-cell or bulk expression matrices (e.g. log2(TPM+1) TCGA-BRCA data) and you must confirm the variational posterior has converged before extracting interaction effects or trajectories.

## When NOT to use

- You are running a different pseudotime method (Monocle 2, DPT, Wishbone, TSCAN) that does not use variational inference and exposes no ELBO to monitor
- You only need a quick exploratory trajectory and will discard the fit — skipping convergence checks is acceptable, but the fit must never be used for published interaction rankings without verifying the flat ELBO trace
- The model has already been verified as converged and you are only re-extracting posteriors from the saved fit object

## Inputs

- Gene-by-sample expression matrix (e.g. log2(TPM+1) TCGA-BRCA matrix restricted to 4579 highly variable genes over 1135 samples, or simulated matrices from simulate_phenopath())
- Covariate vector per sample (e.g. ER status in TCGA-BRCA)
- Convergence-control parameters: elbo_tol and thin, plus z_init (e.g. 1 for principal-component initialisation)

## Outputs

- A converged phenopath fit object with a flat ELBO trace for plot_elbo(fit)
- ELBO trace series confirming convergence before downstream extraction
- Posterior quantities fit for extraction: interaction_effects(fit), interaction_sds(fit), trajectory(fit), significant_interactions(fit)

## How to apply

Pass elbo_tol and thin to the phenopath() call so the evidence lower bound (ELBO) is evaluated every `thin` iterations during co-ordinate ascent mean field variational inference (implemented in Rcpp) and inference halts when the ELBO improvement drops below elbo_tol. Choose elbo_tol conservatively (tighter tolerance means longer runs but more stable posteriors) since premature stopping yields noisy interaction effect sizes (posterior means over posterior sds). After fitting, call plot_elbo(fit) and confirm the ELBO trace is approximately flat — a still-rising or erratic trace means the fit should be rerun with a smaller elbo_tol or different thinning. Only then extract downstream quantities such as interaction_effects(fit), interaction_sds(fit), trajectory(fit), and significant_interactions(fit). This matters because CAVI minimises the KL-divergence to a full-factorised mean-field approximation, and non-convergence directly corrupts the interaction-effect ranking.

## Related tools

- **PhenoPath** (Provides phenopath() with elbo_tol/thin convergence control, plot_elbo() for ELBO trace visualisation, and CAVI inference implemented in Rcpp) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **Rcpp** (Backend library that implements the co-ordinate ascent variational inference whose convergence is being monitored)
- **R** (Runtime environment for invoking phenopath() and plot_elbo() during fit and convergence checking)

## Examples

```
fit <- phenopath(Y, x, z_init = 1, elbo_tol = 1e-5, thin = 20); plot_elbo(fit)
```

## Evaluation signals

- plot_elbo(fit) shows an approximately flat ELBO trace, indicating the CAVI optimisation has converged
- The ELBO is non-decreasing across iterations (a valid variational lower bound property); any decrease signals a numerical problem
- Rerunning phenopath() with a tighter elbo_tol yields materially the same interaction effects and trajectory, showing the result is not an artefact of early stopping
- Downstream outputs (interaction_effects, interaction_sds, trajectory) are finite and stable, with the significance criterion |posterior mean| / posterior sd > k recoverable from the fit
- Convergence is achieved across replicates (e.g. the 40 simulations per setting in the benchmark) rather than for only a single stochastic run

## Limitations

- CAVI uses a full-factorised mean-field approximation that minimises KL-divergence to the true posterior, so convergence of the ELBO does not guarantee the approximation itself is accurate
- PhenoPath assumes features evolve linearly with pseudotime; even a converged fit should be checked for consistency with the data since nonlinear effects are not discounted
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings, which limits the model the convergence check applies to

## Evidence

- [methods] We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form: "We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form"
- [discussion] checks should be conducted to verify that PhenoPath model fits are consistent with the data: "checks should be conducted to verify that PhenoPath model fits are consistent with the data"
- [other] Fit the PhenoPath model with the phenopath() function, passing the expression matrix and ER covariate, initialising the latent space on the first principal component (z_init = 1), with ELBO tolerance elbo_tol: [evidence span withheld — anchor status: not_found]
- [other] Verify convergence by confirming the ELBO trace is approximately flat via plot_elbo(fit): [evidence span withheld — anchor status: not_found]
- [readme] This minimises the KL-divergence between a set of approximating distributions and the true posterior by making a full-factorized mean-field approximation.: [evidence span withheld — anchor status: external_doc]
