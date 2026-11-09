---
name: bulk-tumour-pseudotime-trajectory-inference
description: Use when a bulk tumour expression matrix (log2(TPM+1), highly variable
  genes) with a per-sample covariate such as MSI or ER status must be mapped onto
  a 1D pseudotime and genes with covariate-pseudotime interactions identified.
when_to_use_negative:
- Features are known to evolve nonlinearly with pseudotime — PhenoPath assumes features
  evolve linearly with respect to pseudotime
- No covariate of interest exists and the goal is purely homogeneous trajectory inference,
  since the covariate-pseudotime interaction machinery is the point of this skill
- Complex mixtures of continuous and discrete covariates in high-dimensional settings
  are needed — the paper explicitly defers this to future work
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PhenoPath
  role: Fits the Bayesian covariate-pseudotime model to the bulk expression matrix
    via coordinate ascent mean-field variational inference; provides trajectory(),
    interaction_effects(), interaction_sds(), significant_interactions(), plot_elbo()
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: R
  role: Host language for the PhenoPath Bioconductor package and downstream correlation/tabulation
    of quanTIseq fractions against inferred pseudotimes
- name: quanTIseq
  role: Deconvolves bulk expression data into per-sample immune cell fractions that
    are correlated with inferred pseudotimes (Fig 4d equivalent)
- name: ggplot2
  role: Plots ELBO traces and pseudotime/immune-fraction correlation visualisations
- name: dplyr
  role: Assembles and ranks the gene-covariate interaction table written to CSV
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:9869ea88ffc1907c10eefabdb416618484f592ba48bd9e5fb92d00124d5cfc07
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PhenoPath
  - R
  - dplyr
  - ggplot2
  - quanTIseq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# bulk-tumour-pseudotime-trajectory-inference

## Summary

Infers latent pseudotemporal trajectories from bulk tumour expression cohorts (e.g. TCGA-COAD, TCGA-BRCA) with PhenoPath's Bayesian covariate model, so that gene-covariate interactions (e.g. MLH1 with MSI status, ESR1 with ER status) can be ranked along a disease trajectory.

## When to use

Use when a bulk tumour expression matrix (log2(TPM+1), highly variable genes) with a per-sample covariate such as MSI or ER status must be mapped onto a 1D pseudotime and genes with covariate-pseudotime interactions identified.

## When NOT to use

- Features are known to evolve nonlinearly with pseudotime — PhenoPath assumes features evolve linearly with respect to pseudotime
- No covariate of interest exists and the goal is purely homogeneous trajectory inference, since the covariate-pseudotime interaction machinery is the point of this skill
- Complex mixtures of continuous and discrete covariates in high-dimensional settings are needed — the paper explicitly defers this to future work

## Inputs

- Bulk tumour expression matrix on log2(TPM+1) scale, centre-scaled, restricted to highly variable genes (e.g. 284 samples x 4801 genes for TCGA-COAD; 1135 samples x 4579 genes for TCGA-BRCA)
- Per-sample covariate vector (e.g. MSI status for COAD, ER status for BRCA) from GDC/TCGA clinical annotations
- Optional: quanTIseq per-sample immune cell fractions estimated from the same bulk expression data

## Outputs

- Posterior mean pseudotime vector, one value per sample (trajectory(fit); 284 entries for TCGA-COAD)
- Posterior interaction effect sizes and standard deviations per gene (interaction_effects(fit), interaction_sds(fit))
- Ranked significant-interaction table using |m_beta| / s_beta > k (significant_interactions(fit)), e.g. ranked_interactions.csv
- ELBO convergence trace (plot_elbo(fit))

## How to apply

Restrict the log2(TPM+1) expression matrix to highly variable genes (4801 genes for TCGA-COAD's 284 tumour samples; 4579 genes for TCGA-BRCA's 1135 samples) and assemble the per-sample covariate vector (MSI or ER status). Run quanTIseq deconvolution first if immune-cell-fraction correlations with the inferred trajectory are needed (the Fig 4d analysis). Call phenopath() passing the expression matrix and covariate, initialising the latent space on the first principal component (z_init = 1); inference uses coordinate ascent mean-field variational inference with ELBO computed every `thin` iterations and convergence declared at elbo_tol. Confirm convergence by checking the ELBO trace is approximately flat via plot_elbo(fit). Extract posterior mean pseudotimes with trajectory(fit), then rank gene-covariate interactions from interaction_effects(fit)/interaction_sds(fit), flagging significant interactions with significant_interactions(fit) using the |posterior mean| / posterior sd > k criterion.

## Related tools

- **PhenoPath** (Fits the Bayesian covariate-pseudotime model to the bulk expression matrix via coordinate ascent mean-field variational inference; provides trajectory(), interaction_effects(), interaction_sds(), significant_interactions(), plot_elbo()) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **R** (Host language for the PhenoPath Bioconductor package and downstream correlation/tabulation of quanTIseq fractions against inferred pseudotimes)
- **quanTIseq** (Deconvolves bulk expression data into per-sample immune cell fractions that are correlated with inferred pseudotimes (Fig 4d equivalent))
- **ggplot2** (Plots ELBO traces and pseudotime/immune-fraction correlation visualisations)
- **dplyr** (Assembles and ranks the gene-covariate interaction table written to CSV)

## Examples

```
fit <- phenopath(y, x = msi_status, z_init = 1); plot_elbo(fit); pt <- trajectory(fit); sig <- significant_interactions(fit)
```

## Evaluation signals

- ELBO trace is approximately flat at convergence given elbo_tol and the thin interval (verified with plot_elbo(fit))
- trajectory(fit) returns a pseudotime vector with exactly one entry per sample (e.g. 284 for TCGA-COAD)
- Positive-control recovery: MLH1 appears as the by-far-largest interaction effect for MSI (COAD); ESR1 appears among significant interactions for ER (BRCA, 1932 interacting genes ≈ 42%)
- Every significant interaction satisfies the |posterior mean| / posterior sd > k criterion via significant_interactions(fit)
- Input matrix is restricted to the highly variable gene set (4801 genes COAD / 4579 genes BRCA) before fitting

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects may occur and are not discounted
- Checks should be conducted to verify that PhenoPath model fits are consistent with the data
- PhenoPath cannot yet handle complex mixtures of continuous and discrete covariates in high-dimensional settings (deferred to future work)

## Evidence

- [methods] We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form: "We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form"
- [methods] Ranking covariate-pathway interactions via posterior mean over posterior sd threshold: "We classify or label an interaction as of interest if"
- [discussion] Model fit verification caveat: "checks should be conducted to verify that PhenoPath model fits are consistent with the data"
- [discussion] TCGA application of the method: "We also demonstrated applications to The Cancer Genome Atlas using PhenoPath to model disease trajectories in colorectal and breast cancer."
- [readme] README statement of what PhenoPath learns: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates"
- [readme] README description of CAVI implementation: [evidence span withheld — anchor status: external_doc]
