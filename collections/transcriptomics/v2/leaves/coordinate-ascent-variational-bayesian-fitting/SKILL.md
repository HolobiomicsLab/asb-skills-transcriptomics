---
name: coordinate-ascent-variational-bayesian-fitting
description: Use when fitting a PhenoPath model to an expression matrix (e.g. 284
  TCGA-COAD tumour samples x 4801 highly variable genes, log2(TPM+1)) with a sample-level
  covariate such as MSI status, and convergence must be checked via the ELBO.
when_to_use_negative:
- The scientific question is retrospective association of a pre-computed trajectory
  with a covariate rather than joint inference of trajectory and covariate effects
  — the article motivates PhenoPath precisely because factors should be explicitly
  incorporated, not only examined afterwards.
- Features are expected to evolve nonlinearly with pseudotime — the article states
  an assumption of PhenoPath is that features evolve linearly with respect to pseudotime,
  and model fit checks are required.
- The covariate structure is a complex mixture of continuous and discrete covariates
  in a high-dimensional setting — the discussion defers this to future work.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: PhenoPath
  role: R/Bioconductor package whose phenopath() function performs the CAVI fit; plot_elbo,
    trajectory, interaction_effects, interaction_sds and significant_interactions
    act on the returned fit
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: Rcpp
  role: Underlies the R implementation that makes the coordinate ascent variational
    inference scalable to thousands of samples and genes
- name: R
  role: Language in which the CAVI implementation of PhenoPath is provided and invoked
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:7027ea453e0009ab63fe1e1ca5afee5c967436da9741413e3fbf4b580842d09b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PhenoPath
  - R
  - Rcpp
  - dplyr
  - ggplot2
  - quanTIseq
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# coordinate-ascent-variational-bayesian-fitting

## Summary

Fit PhenoPath's Bayesian pseudotime-covariate model by coordinate ascent mean-field variational inference (CAVI), implemented in Rcpp, which iteratively minimises the KL-divergence between a full-factorised mean-field approximating distribution and the true posterior to infer latent pseudotimes and gene-covariate interactions from bulk or single-cell expression data.

## When to use

Use when fitting a PhenoPath model to an expression matrix (e.g. 284 TCGA-COAD tumour samples x 4801 highly variable genes, log2(TPM+1)) with a sample-level covariate such as MSI status, and convergence must be checked via the ELBO.

## When NOT to use

- The scientific question is retrospective association of a pre-computed trajectory with a covariate rather than joint inference of trajectory and covariate effects — the article motivates PhenoPath precisely because factors should be explicitly incorporated, not only examined afterwards.
- Features are expected to evolve nonlinearly with pseudotime — the article states an assumption of PhenoPath is that features evolve linearly with respect to pseudotime, and model fit checks are required.
- The covariate structure is a complex mixture of continuous and discrete covariates in a high-dimensional setting — the discussion defers this to future work.

## Inputs

- Expression matrix y (samples x genes; e.g. TCGA-COAD 284 tumour samples x 4801 highly variable genes, log2(TPM+1), centre-scaled)
- Sample-level covariate vector x (e.g. binary MSI status per sample)
- Inference settings: elbo_tol, thin (interval), z_init = 1 (PCA initialisation of the latent space)

## Outputs

- A fitted PhenoPath model object containing the ELBO trace over CAVI iterations
- Posterior mean pseudotimes per sample (trajectory(fit); 284 entries for the TCGA-COAD cohort)
- Posterior mean interaction effects and standard deviations (interaction_effects, interaction_sds) for each gene-covariate pair
- Ranked/filtered interaction table from significant_interactions (|m_beta| / s_beta > k, e.g. ranked_interactions.csv with MLH1 as the by-far-largest effect)

## How to apply

Call phenopath() on the centre-scaled expression matrix y with the covariate vector x (e.g. binary MSI status), relying on the default coordinate ascent mean-field variational inference initialised on the first principal component (z_init = 1). Let CAVI optimise the full-factorised approximating distributions over the latent pseudotime z, gene loading/interaction parameters (including the beta interaction terms), and precision parameters until the ELBO gain falls below elbo_tol across the specified thin interval. Plot the ELBO with plot_elbo(fit) to confirm it is approximately flat — a flat ELBO is the primary convergence diagnostic, so if it is still rising, re-run with more iterations or a different thin lag. Once converged, extract posterior means via trajectory(fit) (one pseudotime per sample, e.g. 284 values), and rank interactions using interaction_effects / interaction_sds, labelling an interaction 'of interest' by the posterior mean over posterior standard deviation threshold (the |m_beta| / s_beta > k criterion used by significant_interactions). The mean-field approximation is intentionally full-factorised for scalability to thousands of samples and genes.

## Related tools

- **PhenoPath** (R/Bioconductor package whose phenopath() function performs the CAVI fit; plot_elbo, trajectory, interaction_effects, interaction_sds and significant_interactions act on the returned fit) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **Rcpp** (Underlies the R implementation that makes the coordinate ascent variational inference scalable to thousands of samples and genes)
- **R** (Language in which the CAVI implementation of PhenoPath is provided and invoked) — https://bioconductor.org/packages/release/bioc/html/phenopath.html

## Examples

```
fit <- phenopath(y, x = msi_status, z_init = 1); plot_elbo(fit); pt <- trajectory(fit); sig <- significant_interactions(fit, k)
```

## Evaluation signals

- The ELBO plotted via plot_elbo is approximately flat at convergence given elbo_tol and the thin interval
- trajectory(fit) returns exactly one posterior mean pseudotime per sample (284 entries for the TCGA-COAD application)
- Ranked interactions reproduce the expected positive control: MLH1 appears as the by-far-largest covariate-pseudotime interaction effect for MSI status (ranked_interactions.csv)
- significant_interactions applies the posterior mean over posterior standard deviation criterion (|m_beta| / s_beta > k) and its output is a subset of all gene-covariate interactions
- Model fit consistency checks against the data are conducted, as recommended in the discussion

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects cannot be discounted and checks should be conducted to verify model fits are consistent with the data
- The current implementation cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings (deferred to future work)
- Inference uses a full-factorised mean-field approximation, which discards posterior correlations between latent quantities
- Convergence must be verified: a non-flat ELBO given elbo_tol and the thin interval means the CAVI fit is not trustworthy

## Evidence

- [readme] CAVI minimises KL to approximate the posterior: [evidence span withheld — anchor status: external_doc]
- [readme] Mean-field approximation: [evidence span withheld — anchor status: external_doc]
- [methods] Coordinate ascent mean-field VI in methods: "We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form"
- [methods] Interaction classification criterion: "We classify or label an interaction as of interest if"
- [discussion] Linear evolution assumption and fit checks: "An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime."
