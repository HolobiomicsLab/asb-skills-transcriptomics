---
name: single-cell-pseudotime-inference
description: Use when a scRNA-seq expression matrix from a cross-sectional cohort
  must be ordered along a one-dimensional biological progression without capture times,
  especially when covariates (stimulus, mutation, drug) may perturb the trajectory.
when_to_use_negative:
- Features must evolve linearly with pseudotime — when strong nonlinear gene-pseudotime
  dynamics are expected, PhenoPath's linear assumption may be inappropriate
- When physical capture-time information is available and should be used directly
  for ordering (e.g. DeLorean-style capture-time models) rather than inferred
- When data require complex mixtures of continuous and discrete covariates in high-dimensional
  settings, which current PhenoPath cannot handle
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: PhenoPath
  role: 'Core method: Bayesian hybrid regression-latent variable model inferring pseudotimes
    and covariate interactions via CAVI; R/Bioconductor package'
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: R
  role: Implementation language for phenopath() fitting, ELBO checks, and pseudotime
    extraction
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: Rcpp
  role: Backend implementing coordinate ascent variational inference for performance
    on thousands of samples and genes
- name: SummarizedExperiment
  role: Container for the expression matrix and covariates passed to phenopath()
- name: Monocle 2
  role: Baseline pseudotime method used for comparison (PhenoPath outperformed it,
    R2 = 0.68 vs baselines on Shalek data)
- name: dplyr
  role: Data manipulation of expression matrix and per-cell pseudotime output table
- name: ggplot2
  role: Visualisation of fitted pseudotimes and ELBO/convergence diagnostics
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:6d5add0b78043b5720b278988bd8e85fa83a4f5f513deb012e9a020c65ea2cb0
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
  - tibble
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-pseudotime-inference

## Summary

Infers per-cell pseudotimes (latent pathway scores) from single-cell RNA-seq expression matrices, here via PhenoPath, a Bayesian hybrid regression-latent variable model that can additionally modulate trajectories by covariates such as LPS/PAM stimulation status, using coordinate ascent mean-field variational inference.

## When to use

Use when a scRNA-seq expression matrix from a cross-sectional cohort must be ordered along a one-dimensional biological progression without capture times, especially when covariates (stimulus, mutation, drug) may perturb the trajectory.

## When NOT to use

- Features must evolve linearly with pseudotime — when strong nonlinear gene-pseudotime dynamics are expected, PhenoPath's linear assumption may be inappropriate
- When physical capture-time information is available and should be used directly for ordering (e.g. DeLorean-style capture-time models) rather than inferred
- When data require complex mixtures of continuous and discrete covariates in high-dimensional settings, which current PhenoPath cannot handle

## Inputs

- Single-cell RNA-seq expression matrix (e.g. Salmon re-quantified GSE48963 dendritic-cell data, 820 cells), logged scale comparable to log2(TPM+1)
- Feature-filtered matrix retaining the 7500 most variable genes
- Per-cell covariate vector (e.g. binary LPS vs PAM stimulation status), excluding capture times

## Outputs

- Per-cell posterior-mean pseudotimes (fit$m_z via trajectory(fit)), writable as a CSV file
- Convergence diagnostic: ELBO trace (plot_elbo(fit))
- Optional downstream: covariate-pseudotime interaction effect sizes ranked per gene (e.g. Tnf, Mef2c)

## How to apply

Load a re-quantified expression matrix (e.g. Salmon output for the 820 Shalek dendritic cells, GSE48963), restrict to the 7500 most variable genes, and ensure values are on a logged scale comparable to log2(TPM+1). Encode sample-level covariates (e.g. binary LPS vs PAM stimulation status) and deliberately exclude physical capture times from the model input. Fit PhenoPath with the phenopath() function using coordinate ascent mean-field variational inference (CAVI, Rcpp-backed) with an ELBO tolerance and thinning interval, initialising the latent space on the first principal component (z_init = 1) because the posterior is multi-modal. Verify convergence by confirming the ELBO trace is approximately flat with plot_elbo(fit), then extract posterior-mean pseudotimes with trajectory(fit), which returns fit$m_z. Validate externally, e.g. by computing the Pearson correlation of fitted pseudotimes against physical capture times (expected R2 = 0.68 on the Shalek data, exceeding Monocle 2, DPT and TSCAN).

## Related tools

- **PhenoPath** (Core method: Bayesian hybrid regression-latent variable model inferring pseudotimes and covariate interactions via CAVI; R/Bioconductor package) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **R** (Implementation language for phenopath() fitting, ELBO checks, and pseudotime extraction) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **Rcpp** (Backend implementing coordinate ascent variational inference for performance on thousands of samples and genes)
- **SummarizedExperiment** (Container for the expression matrix and covariates passed to phenopath())
- **Monocle 2** (Baseline pseudotime method used for comparison (PhenoPath outperformed it, R2 = 0.68 vs baselines on Shalek data))
- **dplyr** (Data manipulation of expression matrix and per-cell pseudotime output table)
- **ggplot2** (Visualisation of fitted pseudotimes and ELBO/convergence diagnostics)

## Examples

```
fit <- phenopath(Y = exprs_7500hv_genes, x = lps_pam_covariate, z_init = 1); plot_elbo(fit); pseudotimes <- trajectory(fit)
```

## Evaluation signals

- ELBO trace from plot_elbo(fit) is approximately flat, indicating CAVI convergence
- Pearson correlation of inferred pseudotimes vs physical capture times reaches R2 = 0.68 on the Shalek dendritic-cell data, exceeding Monocle 2, DPT and TSCAN
- For every input gene set size, PhenoPath reports a higher correlation with capture time than the competing methods tested
- Extracted pseudotimes (trajectory(fit) = fit$m_z) are defined for all 820 cells and export cleanly to CSV
- Latent space initialised on PC1 (z_init = 1) because the posterior is multi-modal; reruns with the same initialisation are stable

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects may occur and model fits should be checked for consistency with the data
- Posterior is multi-modal, requiring initialisation of the latent space on the first principal component (z_init = 1)
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings (deferred to future work)
- Application to high-throughput single-cell CRISPR experiments remains unexplored

## Evidence

- [abstract] We describe a novel statistical framework that learns how pseudotime trajectories can be modulated through covariates that encode such factors.: "We describe a novel statistical framework that learns how pseudotime trajectories can be modulated through covariates"
- [methods] We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form: "We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form"
- [discussion] An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime.: "An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime."
- [readme] PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates.: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates"
- [readme] Inference is performed using co-ordinate ascent variational inference (CAVI) implemented using the Rcpp library for increased performance.: [evidence span withheld — anchor status: external_doc]
