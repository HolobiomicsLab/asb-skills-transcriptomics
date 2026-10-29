---
name: covariate-gene-interaction-significance-testing
description: Use when a cross-sectional expression cohort (e.g. TCGA-BRCA log2(TPM+1)
  matrix of 4579 HVGs x 1135 samples) must be ordered along a pseudotemporal trajectory
  while testing which genes' loadings interact with a sample covariate such as ER
  status.
when_to_use_negative:
- Features are expected to evolve nonlinearly with pseudotime — PhenoPath assumes
  features evolve linearly, so strongly nonlinear or branching trajectories are the
  wrong model
- The analysis requires complex mixtures of continuous and discrete covariates in
  high-dimensional settings, which the current PhenoPath implementation explicitly
  defers to future work
- Only retrospective association testing is needed on a fixed trajectory with no covariate-modulated
  loadings — a plain pseudotime method (e.g. Monocle 2) suffices
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PhenoPath
  role: Fits the hybrid regression-latent variable Bayesian model; supplies phenopath(),
    interaction_effects(), interaction_sds(), significant_interactions(), interactions()
    and trajectory() accessors
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: R
  role: Runtime environment for the PhenoPath Bioconductor package and the interaction-ranking
    workflow
- name: dplyr
  role: Assembles and filters the ranked interaction table (gene, covariate, effect
    size, significance flag, cross-over point)
- name: ggplot2
  role: Visualises ELBO convergence traces and covariate-specific loading cross-over
    points along the trajectory
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:3aeecf68369d2d839c533aa9e61c655c453582dd82949a6de5d1f1aee231be88
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PhenoPath
  - R
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# covariate-gene-interaction-significance-testing

## Summary

Fits PhenoPath's Bayesian covariate-pseudotime model to a bulk/single-cell expression cohort with a sample-level covariate (e.g. TCGA-BRCA ER status), then flags genes whose trajectory loadings significantly interact with that covariate. It turns an implicit homogeneous-trajectory assumption into explicit covariate-modulated pathway inference.

## When to use

Use when a cross-sectional expression cohort (e.g. TCGA-BRCA log2(TPM+1) matrix of 4579 HVGs x 1135 samples) must be ordered along a pseudotemporal trajectory while testing which genes' loadings interact with a sample covariate such as ER status.

## When NOT to use

- Features are expected to evolve nonlinearly with pseudotime — PhenoPath assumes features evolve linearly, so strongly nonlinear or branching trajectories are the wrong model
- The analysis requires complex mixtures of continuous and discrete covariates in high-dimensional settings, which the current PhenoPath implementation explicitly defers to future work
- Only retrospective association testing is needed on a fixed trajectory with no covariate-modulated loadings — a plain pseudotime method (e.g. Monocle 2) suffices

## Inputs

- log2(TPM+1) expression matrix restricted to highly variable genes (TCGA-BRCA: 4579 HVGs x 1135 samples from GDC)
- Sample-level covariate vector (e.g. binary ER status per tumour)
- Gene identifiers matching the expression matrix rows (e.g. FBP1, FOXC1, ESR1 annotation)

## Outputs

- Ranked covariate-gene interaction table (gene, covariate, effect size, significance flag, ARD precision chi, pathway loading lambda_g, cross-over point), e.g. tcga_brca_interaction_table.csv
- Per-gene posterior interaction effect sizes and standard deviations (interaction_effects / interaction_sds)
- Vector of significant interaction genes and the inferred pseudotime trajectory trajectory(fit)

## How to apply

Fit the PhenoPath model with phenopath() on the log2(TPM+1) expression matrix (TCGA-BRCA: 4579 highly variable genes, 1135 samples) plus the ER-status covariate vector, initialising the latent space on the first principal component (z_init = 1); inference is coordinate ascent mean-field variational (CAVI) via Rcpp, so verify convergence by confirming the ELBO trace is approximately flat (plot_elbo, with elbo_tol checked every thin iterations). Extract per-gene posterior interaction effect sizes and standard deviations with interaction_effects(fit) and interaction_sds(fit), and label interactions of interest using significant_interactions(fit) under the criterion |posterior mean| / posterior sd > k. Retrieve pathway loadings lambda_g via interactions(fit) and compute per-gene cross-over points where the covariate-specific loading lambda_g + beta_pg*x changes sign along trajectory(fit). Assemble the ranked covariate-pathway interaction table (gene, covariate, effect size, significance flag, ARD precision chi, pathway loading, cross-over point), e.g. tcga_brca_interaction_table.csv, ranking interactions as in the paper's 'Ranking Covariate-Pathway Interactions' procedure. Use a known biology control (e.g. ESR1 must be flagged significant for ER status) as a positive-control sanity check.

## Related tools

- **PhenoPath** (Fits the hybrid regression-latent variable Bayesian model; supplies phenopath(), interaction_effects(), interaction_sds(), significant_interactions(), interactions() and trajectory() accessors) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **R** (Runtime environment for the PhenoPath Bioconductor package and the interaction-ranking workflow)
- **dplyr** (Assembles and filters the ranked interaction table (gene, covariate, effect size, significance flag, cross-over point))
- **ggplot2** (Visualises ELBO convergence traces and covariate-specific loading cross-over points along the trajectory)

## Examples

```
fit <- phenopath(expr_matrix, x, z_init = 1); sig <- significant_interactions(fit)
```

## Evaluation signals

- plot_elbo(fit) trace is approximately flat at termination (elbo_tol satisfied, ELBO computed every thin iterations), confirming CAVI convergence
- ESR1 is flagged as a significant ER-pseudotime interaction (positive control from the TCGA-BRCA run)
- Proportion of significant interactions is in the expected range for the paper's cohort: 1932 of 4579 HVGs (~42%) affected by the ER x pseudotime interaction
- Significance flags are consistent with the stated criterion |posterior mean| / posterior sd > k for every flagged gene
- Output CSV contains all schema columns: gene, covariate, effect size, significance flag, ARD precision chi, pathway loading lambda_g, cross-over point

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects cannot be discounted and checks should verify model fits are consistent with the data
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings (deferred to future work)
- Model outputs are posterior approximations from mean-field variational inference, so downstream significance depends on the |posterior mean| / posterior sd threshold k

## Evidence

- [abstract] We describe a novel statistical framework that learns how pseudotime trajectories can be modulated through covariates that encode such factors.: "We describe a novel statistical framework that learns how pseudotime trajectories can be modulated through covariates that encode such factors."
- [methods] We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form: "We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form"
- [methods] We provide an R implementation of our method PhenoPath at https://bioconductor.org/packages/release/bioc/html/phenopath.html.: "We provide an R implementation of our method PhenoPath at https://bioconductor.org/packages/release/bioc/html/phenopath.html."
- [discussion] An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime.: "An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime."
- [readme] PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates"
