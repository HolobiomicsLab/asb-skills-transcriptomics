---
name: breast-cancer-er-status-stratified-trajectory-analysis
description: Use when you have bulk (or single-cell) expression data from a cohort
  with a binary sample-level covariate (e.g. ER status in TCGA-BRCA) and want to detect
  covariate–pseudotime interaction genes rather than a single averaged trajectory.
when_to_use_negative:
- The data lack a sample-level covariate (e.g. ER/mutation/drug status) — PhenoPath's
  interaction terms require a covariate to perturb the trajectory
- Nonlinear evolution of features along pseudotime is expected — PhenoPath assumes
  features evolve linearly with respect to pseudotime
- The analysis needs complex mixtures of continuous and discrete covariates in high-dimensional
  settings, which current PhenoPath defers to future work
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3525
- http://edamontology.org/topic_2640
tools:
- name: PhenoPath
  role: Bayesian pseudotime-with-covariates model fitted via CAVI; provides phenopath(),
    interaction_effects(), interaction_sds(), significant_interactions(), interactions(),
    trajectory(), plot_elbo()
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: R
  role: Implementation environment for fitting PhenoPath and assembling the ranked
    interaction table
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: ggplot2
  role: Visualisation of ELBO trace and interaction loadings
- name: dplyr
  role: Data manipulation and assembly of the ranked interaction table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:613462f14060783edf75424e9b00e6fdc9e708e68a3383886d6938f0839c69ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PhenoPath
  - R
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3525
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# breast-cancer-er-status-stratified-trajectory-analysis

## Summary

Fits the PhenoPath Bayesian pseudotime model to bulk TCGA-BRCA log2(TPM+1) expression (4579 highly variable genes, 1135 samples) with estrogen receptor status as a covariate, to identify genes whose trajectory loading is modulated by ER status (1932 genes, 42%, including ESR1 as positive control).

## When to use

Use when you have bulk (or single-cell) expression data from a cohort with a binary sample-level covariate (e.g. ER status in TCGA-BRCA) and want to detect covariate–pseudotime interaction genes rather than a single averaged trajectory.

## When NOT to use

- The data lack a sample-level covariate (e.g. ER/mutation/drug status) — PhenoPath's interaction terms require a covariate to perturb the trajectory
- Nonlinear evolution of features along pseudotime is expected — PhenoPath assumes features evolve linearly with respect to pseudotime
- The analysis needs complex mixtures of continuous and discrete covariates in high-dimensional settings, which current PhenoPath defers to future work

## Inputs

- TCGA-BRCA log2(TPM+1) expression matrix restricted to 4579 highly variable genes for 1135 samples
- ER-status covariate vector for the 1135 TCGA-BRCA samples
- PhenoPath fit control parameters (z_init = 1, elbo_tol, thin)

## Outputs

- PhenoPath model fit object with inferred latent trajectory and ELBO trace
- Per-gene posterior interaction effect sizes and standard deviations (interaction_effects, interaction_sds)
- Significance flags for ER–pseudotime interactions (|posterior mean| / posterior sd > k)
- Pathway loadings lambda_g and per-gene cross-over points along trajectory(fit)
- Ranked interaction table written to tcga_brca_interaction_table.csv (gene, covariate, effect size, significance flag, ARD precision chi, pathway loading, cross-over point)

## How to apply

Load the TCGA-BRCA log2(TPM+1) matrix restricted to 4579 highly variable genes (1135 samples) from GDC/TCGA-BRCA plus the ER-status covariate vector. Fit PhenoPath with phenopath(), passing the expression matrix and covariate, initialising the latent space on the first principal component (z_init = 1), with ELBO computed every `thin` iterations and convergence controlled by elbo_tol; confirm the ELBO trace is approximately flat via plot_elbo(fit). Extract posterior interaction effect sizes and standard deviations with interaction_effects(fit) and interaction_sds(fit), and flag significant ER–pseudotime interactions with significant_interactions(fit) using the criterion |posterior mean| / posterior sd > k, the article's ranking rule for covariate-pathway interactions. Retrieve pathway loadings lambda_g via interactions(fit) and compute per-gene cross-over points where the covariate-specific loading lambda_g + beta_pg*x changes sign along trajectory(fit); assemble a ranked table (gene, covariate, effect size, significance flag, ARD precision chi, pathway loading, cross-over point) and write it to tcga_brca_interaction_table.csv. Inference uses co-ordinate ascent mean field variational inference (CAVI) implemented in Rcpp for scalability to thousands of samples and genes.

## Related tools

- **PhenoPath** (Bayesian pseudotime-with-covariates model fitted via CAVI; provides phenopath(), interaction_effects(), interaction_sds(), significant_interactions(), interactions(), trajectory(), plot_elbo()) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **R** (Implementation environment for fitting PhenoPath and assembling the ranked interaction table) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **ggplot2** (Visualisation of ELBO trace and interaction loadings)
- **dplyr** (Data manipulation and assembly of the ranked interaction table)

## Examples

```
fit <- phenopath(exprs = tcga_brca_lcpm, x = er_status, z_init = 1, elbo_tol = 1e-5, thin = 40); sig <- significant_interactions(fit)
```

## Evaluation signals

- ELBO trace is approximately flat (plot_elbo(fit)) indicating CAVI convergence under elbo_tol
- ESR1 appears among significant interaction genes, serving as a positive control for ER-status effects
- Approximately 1932 of 4579 genes (~42%) are flagged as significant ER–pseudotime interactions
- Significance criterion holds: |posterior mean| / posterior sd > k per gene (significant_interactions)
- Output tcga_brca_interaction_table.csv contains all required columns (gene, covariate, effect size, significance flag, ARD precision chi, pathway loading, cross-over point)

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects cannot be discounted, so checks should be conducted to verify model fits are consistent with the data
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings (deferred to future work)
- Interaction classification depends on the posterior mean over posterior standard deviation threshold k, which is a user-set decision point

## Evidence

- [methods] We provide an R implementation of our method PhenoPath: "We provide an R implementation of our method PhenoPath at https://bioconductor.org/packages/release/bioc/html/phenopath.html."
- [methods] Ranking covariate-pathway interactions via posterior mean over posterior sd: "We classify or label an interaction as of interest if"
- [readme] PhenoPath models trajectories modulated by sample-specific covariates: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates"
- [readme] Inference is performed using co-ordinate ascent variational inference implemented in Rcpp: [evidence span withheld — anchor status: external_doc]
- [discussion] Linear pseudotime assumption and fit verification: "However, it cannot be discounted that nonlinear effects may occur and checks should be conducted to verify that PhenoPath model fits are consistent with the data."
- [discussion] Application to TCGA including breast cancer: "We also demonstrated applications to The Cancer Genome Atlas using PhenoPath to model disease trajectories in colorectal and breast cancer."
