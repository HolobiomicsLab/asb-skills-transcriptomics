---
name: gene-interaction-table-export
description: Use when a fitted PhenoPath model exists and you must rank genes by |covariate-pseudotime
  interaction effect|, label interactions of interest, and write the results as a
  per-gene CSV table.
when_to_use_negative:
- The pseudotime/latent pathway has not yet been fitted — run PhenoPath first, since
  interactions() requires a converged phenopath_fit object
- The goal is differential expression along pseudotime without a covariate — a plain
  pseudotime DE analysis is the wrong tool for covariate-pseudotime interaction effects
- Features evolve nonlinearly with pseudotime and that nonlinearity is the quantity
  of interest — PhenoPath assumes features evolve linearly with respect to pseudotime
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: PhenoPath
  role: Source of the fitted model and the interactions()/significant_interactions()
    accessors that produce the per-gene interaction table
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: dplyr
  role: Sorting/assembling the ranked per-gene table (e.g. arrange by abs(interaction_effect_size))
    before export
- name: tibble
  role: Structures the exported per-gene table with the interaction columns
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:4d742da214df89fcdbb531fa42d250589bea0c6e14a32dadd979c7d99bbbd326
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PhenoPath
  - dplyr
  - tibble
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gene-interaction-table-export

## Summary

Exports a ranked per-gene table of covariate-pseudotime interaction effect sizes from a fitted PhenoPath model (e.g. the Shalek LPS/PAM dendritic-cell dataset), flagging Bayesian-significant interactions such as Tnf and Mef2c for downstream review or figure reproduction.

## When to use

Use when a fitted PhenoPath model exists and you must rank genes by |covariate-pseudotime interaction effect|, label interactions of interest, and write the results as a per-gene CSV table.

## When NOT to use

- The pseudotime/latent pathway has not yet been fitted — run PhenoPath first, since interactions() requires a converged phenopath_fit object
- The goal is differential expression along pseudotime without a covariate — a plain pseudotime DE analysis is the wrong tool for covariate-pseudotime interaction effects
- Features evolve nonlinearly with pseudotime and that nonlinearity is the quantity of interest — PhenoPath assumes features evolve linearly with respect to pseudotime

## Inputs

- Fitted PhenoPath model object (e.g. phenopath_fit from the Shalek et al. dendritic-cell stimulation dataset with LPS/PAM/untreated covariate)
- Posterior interaction effect sizes and standard deviations (interaction_effects(fit), interaction_sds(fit))

## Outputs

- Ranked per-gene interaction table (columns: feature, covariate, interaction_effect_size, significant, chi, pathway_loading) written as interaction_table.csv
- Flagged interactions of interest defined by |m_beta|/s_beta > k

## How to apply

Load the PhenoPath fit object (e.g. produced for the Shalek data with stimulus covariate LPS vs PAM vs untreated), then extract posterior interaction effects via interaction_effects(fit), posterior standard deviations via interaction_sds(fit), and significance via significant_interactions(fit). Assemble the per-gene table with the interactions(phenopath_fit) helper, which returns the columns feature, covariate, interaction_effect_size, significant, chi (ARD prior precision), and pathway_loading. Rank genes by the magnitude |beta| of the interaction effect size and apply the article's Bayesian decision rule — label an interaction as of interest when the posterior mean divided by posterior standard deviation exceeds threshold k. Verify the expected biology before exporting: the top-ranked interaction should be Tnf (largest effect, roughly twice any other gene; decreasing along pseudotime under LPS but increasing under PAM), with Mef2c flat under LPS but downregulated under PAM. Finally write the ranked data frame (via dplyr/tibble) to interaction_table.csv.

## Related tools

- **PhenoPath** (Source of the fitted model and the interactions()/significant_interactions() accessors that produce the per-gene interaction table) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **dplyr** (Sorting/assembling the ranked per-gene table (e.g. arrange by abs(interaction_effect_size)) before export)
- **tibble** (Structures the exported per-gene table with the interaction columns)

## Examples

```
itab <- interactions(phenopath_fit); itab %>% dplyr::arrange(desc(abs(interaction_effect_size))) %>% write.csv("interaction_table.csv")
```

## Evaluation signals

- interaction_table.csv contains exactly the columns feature, covariate, interaction_effect_size, significant, chi, and pathway_loading
- Tnf ranks first by |interaction_effect_size| with an effect roughly twice that of any other gene
- Tnf's sign pattern matches expectation: negative along pseudotime under LPS, positive under PAM; Mef2c is flat under LPS but downregulated under PAM
- Rows flagged 'significant' satisfy the Bayesian rule |m_beta|/s_beta > k and agree with significant_interactions(fit) output
- chi column values correspond to the ARD prior precision and pathway_loading to the pathway loading for each feature

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime; nonlinear effects may occur and model fits should be checked against the data
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings (deferred to future work)
- The interaction is defined per (feature, covariate) pair, so the table is one row per gene-covariate combination, not one row per gene

## Evidence

- [methods] assembling ranked interaction table: "We classify or label an interaction as of interest if"
- [abstract] recovering interaction effects: "show that the approach can recover known and novel covariate-pseudotime interaction effects"
- [discussion] model fit verification caveat: "checks should be conducted to verify that PhenoPath model fits are consistent with the data"
- [readme] PhenoPath covariate interactions scope: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates"
