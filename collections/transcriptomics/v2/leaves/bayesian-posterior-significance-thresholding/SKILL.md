---
name: bayesian-posterior-significance-thresholding
description: Use when you have a fitted PhenoPath model (phenopath_fit) and need to
  rank and call significant covariate-trajectory interaction effects per gene, e.g.
  stimulant-pseudotime interactions for LPS vs PAM in the Shalek dendritic cell data.
when_to_use_negative:
- When genes are known to evolve nonlinearly with pseudotime — PhenoPath assumes features
  evolve linearly with respect to pseudotime, so posterior interaction calls may not
  fit the data
- When the data contain complex mixtures of continuous and discrete covariates in
  high-dimensional settings, which current PhenoPath explicitly defers to future work
- When a standard differential-expression comparison between covariate-positive and
  covariate-negative cells suffices — benchmarking shows DE-based methods (Limma Voom,
  DESeq2, MAST, Monocle 2) only match PhenoPath AUC at low interaction fractions (~5-10%)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: PhenoPath
  role: Provides the variational fit and the interaction_effects / interaction_sds
    / significant_interactions / interactions accessor functions implementing the
    |m_beta|/s_beta > k decision rule
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: dplyr
  role: Tidy manipulation and ranking of the per-gene interaction table by |beta|
    magnitude
- name: tibble
  role: Constructs the tidy per-gene interaction table (interaction_table.csv)
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:5c48ea51d915e69554361736a26733e614cceab372b27fa6c8ed5f532041391e
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
  - limma-voom
  - tibble
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# bayesian-posterior-significance-thresholding

## Summary

Labels covariate-pseudotime interactions from a PhenoPath variational fit as 'of interest' using a Bayesian decision rule on the posterior mean over posterior standard deviation of the interaction effect size (|m_beta|/s_beta > k), rather than frequentist p-values.

## When to use

Use when you have a fitted PhenoPath model (phenopath_fit) and need to rank and call significant covariate-trajectory interaction effects per gene, e.g. stimulant-pseudotime interactions for LPS vs PAM in the Shalek dendritic cell data.

## When NOT to use

- When genes are known to evolve nonlinearly with pseudotime — PhenoPath assumes features evolve linearly with respect to pseudotime, so posterior interaction calls may not fit the data
- When the data contain complex mixtures of continuous and discrete covariates in high-dimensional settings, which current PhenoPath explicitly defers to future work
- When a standard differential-expression comparison between covariate-positive and covariate-negative cells suffices — benchmarking shows DE-based methods (Limma Voom, DESeq2, MAST, Monocle 2) only match PhenoPath AUC at low interaction fractions (~5-10%)

## Inputs

- Fitted PhenoPath model object (phenopath_fit) from variational inference on single-cell or bulk expression data with a sample-level covariate (e.g. LPS vs PAM vs untreated)
- Simulated or observed expression matrix with ground-truth interaction labels (for benchmarking AUC of the ranking threshold)

## Outputs

- Per-gene interaction table (feature, covariate, interaction_effect_size, significant, chi, pathway_loading), e.g. interaction_table.csv
- Binary significant/not-significant calls per gene from significant_interactions(fit, n)
- Ranked genes by |interaction_effect_size| for ROC/AUC benchmarking against Limma Voom, DESeq2, MAST, Monocle 2

## How to apply

Fit PhenoPath with coordinate ascent mean-field variational inference (e.g. elbo_tol = 1e-6, thin = 40), then extract posterior interaction effect sizes with interaction_effects(fit) and their posterior standard deviations with interaction_sds(fit). Assemble a per-gene table via interactions(phenopath_fit), which returns feature, covariate, interaction_effect_size, significant, chi (ARD prior precision), and pathway_loading columns. Apply the Bayesian decision rule |m_beta|/s_beta > k to classify an interaction as of interest, equivalently via significant_interactions(fit, n) for the top-n calls. Rank genes by |beta| magnitude; in the Shalek LPS/PAM application this correctly surfaces Tnf as the largest interaction (decreasing under LPS yet increasing under PAM) and Mef2c as flat under LPS but downregulated under PAM, reproducing Fig 3a/2e. The ARD prior precision chi and pathway_loading columns support downstream inspection of sparsity and pathway coupling.

## Related tools

- **PhenoPath** (Provides the variational fit and the interaction_effects / interaction_sds / significant_interactions / interactions accessor functions implementing the |m_beta|/s_beta > k decision rule) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **dplyr** (Tidy manipulation and ranking of the per-gene interaction table by |beta| magnitude)
- **tibble** (Constructs the tidy per-gene interaction table (interaction_table.csv))

## Examples

```
it <- interactions(phenopath_fit); sig <- significant_interactions(phenopath_fit, n = 10)
```

## Evaluation signals

- The interactions(phenopath_fit) table must contain exactly the columns feature, covariate, interaction_effect_size, significant, chi, pathway_loading
- A gene is flagged significant only where the posterior mean over posterior standard deviation ratio exceeds threshold k (|m_beta|/s_beta > k)
- On the Shalek dataset, Tnf must rank as the top interaction (~twice any other gene) with opposite LPS/PAM pseudotime behaviour, and Mef2c must appear flat under LPS but downregulated under PAM (Fig 3a/2e)
- In the simulation benchmark (Fig 1f), PhenoPath AUC computed from this ranking must remain high as the interaction gene fraction grows, while Limma Voom, DESeq2, MAST and Monocle 2 AUCs decrease
- All reported AUC values from the ranked lists must lie in [0, 1], with one row per method per interaction fraction

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects may occur and checks should be conducted to verify that model fits are consistent with the data
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings (deferred to future work)
- Significance depends on posterior standard deviations from a mean-field variational approximation, which may be overconfident relative to the true posterior

## Evidence

- [methods] Bayesian decision rule for calling interactions: "We classify or label an interaction as of interest if"
- [methods] significance accessor: "significant_interactions(phenopath_fit, n)"
- [methods] variational inference engine behind the posterior quantities: "We perform co-ordinate ascent mean field variational inference (see ref. 45) with an approximating distribution of the form"
- [readme] scalability rationale: "Scalable variational Bayesian inference allows the trajectory and interactions to be inferred for thousands of samples and genes quickly."
- [discussion] model assumption limiting interpretation of the thresholded calls: "An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime."
