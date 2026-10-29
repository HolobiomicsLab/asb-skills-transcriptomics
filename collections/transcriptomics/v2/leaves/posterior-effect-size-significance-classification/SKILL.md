---
name: posterior-effect-size-significance-classification
description: Use when a fitted PhenoPath variational model (phenopath_fit) contains
  posterior interaction coefficients and you must rank and call significant gene-covariate
  (e.g. MSI-pseudotime) interactions.
when_to_use_negative:
- The model of interest does not produce posterior interaction coefficients (m_beta,
  s_beta), e.g. a purely deterministic pseudotime method with no variational posterior
- The analysis requires complex mixtures of continuous and discrete covariates in
  high-dimensional settings, which current PhenoPath does not handle (deferred to
  future work)
- The goal is to detect nonlinear covariate effects, since PhenoPath assumes features
  evolve linearly with pseudotime
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: PhenoPath
  role: Provides interaction_effects, interaction_sds, and significant_interactions
    functions that supply the posterior means and standard deviations used for the
    |m_beta|/s_beta > k classification
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: R
  role: Execution environment for the PhenoPath Bioconductor package and the significant_interactions
    call
- name: dplyr
  role: Tabulation and ranking of interaction effects into the Fig 5a-equivalent ranked
    table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:7836c94334edba252a416d4ee407364063811f6b36511ce86f6e4aeda466beda
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PhenoPath
  - R
  - dplyr
  - ggplot2
  - quanTIseq
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# posterior-effect-size-significance-classification

## Summary

Classify gene-covariate interaction effects from a fitted PhenoPath model as 'of interest' by thresholding the ratio of posterior mean interaction effect to its posterior standard deviation (|m_beta| / s_beta > k), producing a ranked table of significant covariate-pseudotime interactions.

## When to use

Use when a fitted PhenoPath variational model (phenopath_fit) contains posterior interaction coefficients and you must rank and call significant gene-covariate (e.g. MSI-pseudotime) interactions.

## When NOT to use

- The model of interest does not produce posterior interaction coefficients (m_beta, s_beta), e.g. a purely deterministic pseudotime method with no variational posterior
- The analysis requires complex mixtures of continuous and discrete covariates in high-dimensional settings, which current PhenoPath does not handle (deferred to future work)
- The goal is to detect nonlinear covariate effects, since PhenoPath assumes features evolve linearly with pseudotime

## Inputs

- Fitted PhenoPath model object (phenopath_fit) containing posterior interaction means (m_beta) and posterior standard deviations (s_beta)
- Gene-covariate interaction effects matrix from a PhenoPath run on log2(TPM+1), centre-scaled expression data (e.g. 284 TCGA-COAD tumour samples x 4801 highly variable genes with MSI status covariate)

## Outputs

- Ranked table of gene-covariate interactions (ranked_interactions.csv equivalent, Fig 5a-style)
- List of significant interactions passing the |m_beta| / s_beta > k criterion (significant_interactions output)

## How to apply

Fit PhenoPath with coordinate ascent mean-field variational inference (CAVI) first, then extract interaction effects and their posterior standard deviations. Rank genes by the magnitude of the posterior interaction effect and classify an interaction as of interest when the posterior mean exceeds the posterior standard deviation by a factor k (|m_beta| / s_beta > k), as implemented by significant_interactions(phenopath_fit, n). Apply this to the TCGA-COAD use case by ranking all 4801 genes against the MSI status covariate; a correct run yields MLH1 as the by-far-largest interaction effect, serving as a positive control because MLH1 is a well-known DNA mismatch repair gene associated with MSI. Rationale: the Bayesian posterior quantifies uncertainty per gene, so a large effect with small s_beta indicates a robustly identified interaction rather than an inference artifact.

## Related tools

- **PhenoPath** (Provides interaction_effects, interaction_sds, and significant_interactions functions that supply the posterior means and standard deviations used for the |m_beta|/s_beta > k classification) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **R** (Execution environment for the PhenoPath Bioconductor package and the significant_interactions call)
- **dplyr** (Tabulation and ranking of interaction effects into the Fig 5a-equivalent ranked table)

## Examples

```
significant_interactions(phenopath_fit, n)
```

## Evaluation signals

- Every returned interaction satisfies |m_beta| / s_beta > k; inspect interaction_effects / interaction_sds directly from the fit
- significant_interactions(phenopath_fit, n) returns exactly n top-ranked genes with finite posterior statistics
- For the TCGA-COAD validation, MLH1 appears as the by-far-largest interaction effect in ranked_interactions.csv (expected positive control)
- The fitted model's ELBO is approximately flat at convergence (checked with plot_elbo given elbo_tol and thin) before trusting the posterior classification
- The ranked table contains entries for all 4801 highly variable genes against the MSI covariate before thresholding

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects may occur and are not discounted, so checks should verify the fit is consistent with the data
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings (deferred to future work)
- The classification is only as reliable as the variational posterior; ill-converged CAVI runs (non-flat ELBO) can yield spurious m_beta/s_beta ratios

## Evidence

- [methods] We classify or label an interaction as of interest if: "We classify or label an interaction as of interest if"
- [methods] significant_interactions(phenopath_fit, n): "significant_interactions(phenopath_fit, n)"
- [methods] Ranking Covariate-Pathway Interactions: "## Methods > Ranking Covariate-Pathway Interactions"
- [readme] PhenoPath identifies interactions between trajectories and covariates: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates and identifies interactions"
- [readme] CAVI inference implemented in Rcpp: [evidence span withheld — anchor status: external_doc]
