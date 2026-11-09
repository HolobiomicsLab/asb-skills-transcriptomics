---
name: tumour-immune-fraction-deconvolution
description: Use when bulk RNA-seq tumour samples (e.g. log2(TPM+1) TCGA data) are
  being ordered along a covariate-modulated pseudotime and you need per-sample immune
  cell fractions to validate or annotate the trajectory.
when_to_use_negative:
- Input is single-cell expression data rather than bulk tumour expression — quanTIseq
  deconvolution is designed for bulk samples
- Expression data is not bulk gene expression on a comparable scale (e.g. raw counts
  without normalisation/TPM scaling), as deconvolution fractions and the PhenoPath
  centre-scaled input would be inconsistent
- The analysis goal is only covariate-pseudotime interaction ranking (significant_interactions
  with |m_beta|/s_beta > k) with no immune-fraction validation step
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: quanTIseq
  role: performs bulk-expression immune cell deconvolution to yield per-sample immune
    cell fractions correlated with the inferred pseudotimes
- name: PhenoPath
  role: fits the covariate-modulated pseudotime model on the same bulk expression
    matrix whose trajectory the immune fractions are correlated against
  repo: https://github.com/kieranrcampbell/phenopath
- name: R
  role: environment in which quanTIseq outputs and PhenoPath fits (phenopath(), trajectory(),
    plot_elbo()) are combined and correlations tabulated
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: ggplot2
  role: used to plot the immune-fraction vs pseudotime correlations (Fig 4d-equivalent
    visualisation)
- name: dplyr
  role: used to tabulate and manipulate the per-sample immune fractions and correlation
    results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:f5ed10ed4ceed49ecb592cbb479a49ea362940045f37126db3f724891ee50ebd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kieranrcampbell/phenopath
  tools:
  - PhenoPath
  - R
  - dplyr
  - ggplot2
  - quanTIseq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# tumour-immune-fraction-deconvolution

## Summary

Estimate per-sample immune cell fractions from bulk tumour expression data via quanTIseq deconvolution, then correlate these fractions with PhenoPath-inferred pseudotimes to validate that the inferred trajectory captures immune infiltration, as demonstrated in the TCGA colorectal cancer analysis.

## When to use

Use when bulk RNA-seq tumour samples (e.g. log2(TPM+1) TCGA data) are being ordered along a covariate-modulated pseudotime and you need per-sample immune cell fractions to validate or annotate the trajectory.

## When NOT to use

- Input is single-cell expression data rather than bulk tumour expression — quanTIseq deconvolution is designed for bulk samples
- Expression data is not bulk gene expression on a comparable scale (e.g. raw counts without normalisation/TPM scaling), as deconvolution fractions and the PhenoPath centre-scaled input would be inconsistent
- The analysis goal is only covariate-pseudotime interaction ranking (significant_interactions with |m_beta|/s_beta > k) with no immune-fraction validation step

## Inputs

- bulk tumour gene expression matrix (samples x genes, e.g. TCGA-COAD 284 samples x 4801 genes, log2(TPM+1)-scale, centre-scaled)
- per-sample covariate vector (e.g. MSI status) used by PhenoPath
- quanTIseq-compatible gene expression data for immune cell fraction estimation

## Outputs

- per-sample immune cell fraction table (one column per cell type from quanTIseq deconvolution)
- vector of posterior mean pseudotimes from trajectory(fit) with one entry per sample
- table of correlations between each immune-cell fraction and the inferred pseudotime (Fig 4d-equivalent)

## How to apply

Start from the bulk expression matrix used for the PhenoPath fit (e.g. 284 TCGA-COAD tumour samples x 4801 highly variable genes on log2(TPM+1) scale, centre-scaled). Run quanTIseq deconvolution on this bulk expression data to obtain per-sample fractions of immune cell types. Independently fit the PhenoPath model with the phenopath() function using coordinate ascent mean-field variational inference, initialising the latent space on the first principal component (z_init = 1), and check convergence via plot_elbo against elbo_tol and the thin interval. Extract posterior mean pseudotimes with trajectory(fit), then correlate each quanTIseq immune-cell fraction with the inferred pseudotimes and tabulate the correlations (the Fig 4d-equivalent step). Rationale: quanTIseq fractions provide an orthogonal biological signal that should track the immune component of the trajectory, so high or structured correlations support that the pseudotime captures real tumour biology rather than noise.

## Related tools

- **quanTIseq** (performs bulk-expression immune cell deconvolution to yield per-sample immune cell fractions correlated with the inferred pseudotimes)
- **PhenoPath** (fits the covariate-modulated pseudotime model on the same bulk expression matrix whose trajectory the immune fractions are correlated against) — https://github.com/kieranrcampbell/phenopath
- **R** (environment in which quanTIseq outputs and PhenoPath fits (phenopath(), trajectory(), plot_elbo()) are combined and correlations tabulated) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **ggplot2** (used to plot the immune-fraction vs pseudotime correlations (Fig 4d-equivalent visualisation))
- **dplyr** (used to tabulate and manipulate the per-sample immune fractions and correlation results)

## Evaluation signals

- The extracted pseudotime vector from trajectory(fit) has exactly one entry per sample (284 entries for the TCGA-COAD cohort)
- plot_elbo shows the ELBO is approximately flat at convergence given elbo_tol and the thin interval
- Every quanTIseq immune-cell type yields a defined correlation coefficient with the pseudotime, and the full set is tabulated (Fig 4d-equivalent)
- The downstream interaction ranking reproduces MLH1 as the by-far-largest covariate-pseudotime interaction with MSI status, serving as the positive control for the mismatch-repair/MSI axis

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects may occur and checks should be conducted to verify that model fits are consistent with the data
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings; this is deferred to future work
- Deconvolution-based validation depends on quanTIseq cell-type fractions being meaningful for the bulk tumour mixture; correlations are descriptive, not a formal test of trajectory correctness

## Evidence

- [readme] PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates"
- [methods] We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form: "We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form"
- [discussion] We also demonstrated applications to The Cancer Genome Atlas using PhenoPath to model disease trajectories in colorectal and breast cancer.: "We also demonstrated applications to The Cancer Genome Atlas using PhenoPath to model disease trajectories in colorectal and breast cancer."
- [discussion] checks should be conducted to verify that PhenoPath model fits are consistent with the data: "checks should be conducted to verify that PhenoPath model fits are consistent with the data"
- [readme] Inference is performed using co-ordinate ascent variational inference (CAVI) implemented using the Rcpp library for increased performance.: [evidence span withheld — anchor status: external_doc]
