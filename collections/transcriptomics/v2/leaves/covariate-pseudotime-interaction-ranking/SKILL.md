---
name: covariate-pseudotime-interaction-ranking
description: Use when a PhenoPath fit is available and the goal is to rank genes by
  covariate-pseudotime interaction effect size and identify significant interactions
  (e.g. Shalek LPS/PAM genes, TCGA MSI analysis).
when_to_use_negative:
- The task is plain differential expression between covariate groups without a latent
  trajectory — the paper shows Limma Voom, DESeq2, MAST and Monocle 2 DE-based interaction
  tests lose AUC as the interaction fraction exceeds 10%, so only apply this skill
  when interactions must be modelled jointly with pseudotime.
- Nonlinear gene dynamics along the trajectory are expected or observed, since PhenoPath
  assumes features evolve linearly with pseudotime and model-fit checks are required.
- The covariate structure is a complex mixture of continuous and discrete factors
  in high-dimensional settings, which current PhenoPath cannot handle (deferred to
  future work).
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: PhenoPath
  role: Fits the Bayesian regression-latent variable model and provides interaction_effects,
    interaction_sds, significant_interactions and interactions helpers used to rank
    covariate-pseudotime interactions
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: R
  role: Implementation language in which the ranking workflow (extract posterior betas,
    compute |m_beta|/s_beta, write ranked CSV) is executed
- name: dplyr
  role: Imported by PhenoPath and used to assemble and rank the per-gene interaction
    table
- name: tibble
  role: Imported by PhenoPath; used to build the tidy per-gene interaction table
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:5417a83e8fc43006505fb57612219b20940b4ae4d514ade0253e03f7455fa29c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - MAST
  - Monocle 2
  - PhenoPath
  - R
  - dplyr
  - ggplot2
  - limma-voom
  - quanTIseq
  - tibble
  - tidyr
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# covariate-pseudotime-interaction-ranking

## Summary

Ranks gene-covariate interactions from a fitted PhenoPath model by posterior interaction effect size, using the Bayesian decision rule |m_beta|/s_beta > k (via interaction_effects, interaction_sds, significant_interactions) to flag genes whose temporal behaviour is perturbed by a covariate such as LPS vs PAM stimulation or MSI status.

## When to use

Use when a PhenoPath fit is available and the goal is to rank genes by covariate-pseudotime interaction effect size and identify significant interactions (e.g. Shalek LPS/PAM genes, TCGA MSI analysis).

## When NOT to use

- The task is plain differential expression between covariate groups without a latent trajectory — the paper shows Limma Voom, DESeq2, MAST and Monocle 2 DE-based interaction tests lose AUC as the interaction fraction exceeds 10%, so only apply this skill when interactions must be modelled jointly with pseudotime.
- Nonlinear gene dynamics along the trajectory are expected or observed, since PhenoPath assumes features evolve linearly with pseudotime and model-fit checks are required.
- The covariate structure is a complex mixture of continuous and discrete factors in high-dimensional settings, which current PhenoPath cannot handle (deferred to future work).

## Inputs

- Fitted PhenoPath model object (phenopath_fit) produced by coordinate ascent mean-field variational inference
- Gene expression matrix (e.g. single-cell Shalek LPS/PAM data or 284-sample TCGA-COAD bulk log2(TPM+1) matrix, 4801 highly variable genes, centre-scaled)
- Per-sample covariate vector (e.g. LPS vs PAM stimulation, MSI status)

## Outputs

- Ranked interaction table with columns feature, covariate, interaction_effect_size, significant, chi (ARD prior precision), pathway_loading (e.g. interaction_table.csv)
- Significant interaction calls from significant_interactions(phenopath_fit, n) using the |m_beta|/s_beta > k threshold

## How to apply

Fit PhenoPath with the phenopath() function using coordinate ascent mean-field variational inference (ELBO-based convergence control via elbo_tol and thin, PC initialisation z_init = 1), then extract posterior interaction effect sizes with interaction_effects(fit) and posterior standard deviations with interaction_sds(fit). Assemble the per-gene table with the interactions(phenopath_fit) helper, which returns the columns feature, covariate, interaction_effect_size, significant, chi (ARD prior precision) and pathway_loading. Rank genes by |beta| magnitude and flag interactions of interest with the Bayesian decision rule |m_beta|/s_beta > k as implemented in significant_interactions(phenopath_fit, n). Validate on known controls: in the Shalek dendritic-cell data the top interaction should be Tnf (largest effect, roughly twice any other gene, decreasing under LPS but increasing under PAM) with Mef2c flat under LPS yet downregulated under PAM; in TCGA COAD the by-far-largest interaction should be MLH1, a mismatch-repair positive control for MSI status.

## Related tools

- **PhenoPath** (Fits the Bayesian regression-latent variable model and provides interaction_effects, interaction_sds, significant_interactions and interactions helpers used to rank covariate-pseudotime interactions) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **R** (Implementation language in which the ranking workflow (extract posterior betas, compute |m_beta|/s_beta, write ranked CSV) is executed)
- **dplyr** (Imported by PhenoPath and used to assemble and rank the per-gene interaction table)
- **tibble** (Imported by PhenoPath; used to build the tidy per-gene interaction table)

## Examples

```
sig <- significant_interactions(phenopath_fit, n); it <- interactions(phenopath_fit)
```

## Evaluation signals

- The ranked table contains the expected columns: feature, covariate, interaction_effect_size, significant, chi (ARD prior precision), pathway_loading.
- On the Shalek dendritic-cell data the top-ranked interaction is Tnf with the largest effect (about twice any other gene), downregulated along pseudotime under LPS yet upregulated under PAM, and Mef2c is flat over pseudotime under LPS but downregulated under PAM.
- On TCGA-COAD, MLH1 appears as the by-far-largest interaction effect with MSI status in ranked_interactions.csv, serving as a positive control.
- Significance calls follow the Bayesian decision rule |m_beta|/s_beta > k and agree with significant_interactions(phenopath_fit, n).
- The PhenoPath fit converged: the ELBO trace is approximately flat given elbo_tol and the thin interval.

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects may occur and checks should be conducted to verify model fits are consistent with the data.
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings (deferred to future work).
- Potential usage in high-throughput single-cell CRISPR experiments remains unexplored.

## Evidence

- [methods] Ranking covariate-pathway interactions: "## Methods > Ranking Covariate-Pathway Interactions"
- [methods] significant_interactions call signature: "significant_interactions(phenopath_fit, n)"
- [methods] Decision rule for labelling interactions of interest: "We classify or label an interaction as of interest if"
- [readme] PhenoPath purpose statement: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates"
- [readme] Inference algorithm: [evidence span withheld — anchor status: external_doc]
- [discussion] Linear pseudotime assumption: "An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime."
