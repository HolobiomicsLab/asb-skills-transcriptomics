---
name: variational-posterior-parameter-extraction
description: Use when a PhenoPath model fit (e.g. on Shalek et al. dendritic cell
  LPS/PAM stimulation data) is already trained and you need per-gene interaction effect
  sizes, posterior SDs, and significance calls.
when_to_use_negative:
- No PhenoPath fit object exists yet — you must first run the variational fitting
  (phenopath()) on the expression data; these helpers only read posteriors
- Features are not expected to evolve linearly with pseudotime — PhenoPath assumes
  linear evolution and post-hoc interaction extraction inherits that assumption
- The covariate is a complex mixture of continuous and discrete variables in a high-dimensional
  setting — the article defers such handling to future work
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PhenoPath
  role: Provides the fit object and the posterior-extraction helpers interaction_effects,
    interaction_sds, significant_interactions, and interactions(phenopath_fit)
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: dplyr
  role: Filtering/ranking the extracted interaction table by |beta| magnitude (dplyr
    is a PhenoPath package import)
- name: tibble
  role: Assembling the per-gene interaction table as a tibble before writing to CSV
    (tibble is a PhenoPath package import)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:2e1dbec1be2344a90cd76925830b0fabf4832762239f4dcff317ac05f9f09662
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PhenoPath
  - dplyr
  - tibble
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# variational-posterior-parameter-extraction

## Summary

Extracts posterior interaction effect sizes, posterior standard deviations, and Bayesian significance calls from a fitted PhenoPath CAVID variational model, turning the inferred latent trajectory into a per-gene covariate-pseudotime interaction table.

## When to use

Use when a PhenoPath model fit (e.g. on Shalek et al. dendritic cell LPS/PAM stimulation data) is already trained and you need per-gene interaction effect sizes, posterior SDs, and significance calls.

## When NOT to use

- No PhenoPath fit object exists yet — you must first run the variational fitting (phenopath()) on the expression data; these helpers only read posteriors
- Features are not expected to evolve linearly with pseudotime — PhenoPath assumes linear evolution and post-hoc interaction extraction inherits that assumption
- The covariate is a complex mixture of continuous and discrete variables in a high-dimensional setting — the article defers such handling to future work

## Inputs

- PhenoPath model fit object (posterior from CAVI inference, e.g. produced by phenopath() on single-cell expression data with a stimulus covariate such as LPS vs PAM vs untreated)
- Threshold k for the Bayesian decision rule |m_beta|/s_beta > k

## Outputs

- Per-gene interaction table (tibble/data.frame) with columns feature, covariate, interaction_effect_size, significant, chi (ARD prior precision), pathway_loading
- Ranked vector/table of genes ordered by |interaction effect size|
- Boolean significance flags per gene-covariate interaction

## How to apply

Fit or load the PhenoPath fit object first (inference is co-ordinate ascent mean-field variational Bayes implemented in Rcpp). Extract posterior interaction effect sizes with interaction_effects(fit) and posterior standard deviations with interaction_sds(fit); call significant_interactions(fit) to apply the Bayesian decision rule of flagging interactions whose posterior mean divided by posterior standard deviation exceeds a threshold k. Assemble the per-gene table with the interactions(phenopath_fit) helper, which returns columns feature, covariate, interaction_effect_size, significant, chi (the ARD prior precision), and pathway_loading. Rank genes by |beta| magnitude and verify against known results (e.g. Tnf having the largest LPS-vs-PAM stimulant-pseudotime interaction) before writing out the ranked table.

## Related tools

- **PhenoPath** (Provides the fit object and the posterior-extraction helpers interaction_effects, interaction_sds, significant_interactions, and interactions(phenopath_fit)) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **dplyr** (Filtering/ranking the extracted interaction table by |beta| magnitude (dplyr is a PhenoPath package import))
- **tibble** (Assembling the per-gene interaction table as a tibble before writing to CSV (tibble is a PhenoPath package import))

## Examples

```
itr <- interactions(phenopath_fit); sig <- significant_interactions(phenopath_fit, n = 10)
```

## Evaluation signals

- The interactions(phenopath_fit) call returns all six expected columns: feature, covariate, interaction_effect_size, significant, chi, pathway_loading
- Top-ranked gene by |interaction_effect_size| matches the expected result (e.g. Tnf with roughly twice the effect of any other gene, decreasing under LPS but increasing under PAM)
- Significance flags agree with the Bayesian decision rule |m_beta|/s_beta > k recomputed from interaction_effects(fit) and interaction_sds(fit)
- chi (ARD prior precision) values are present and finite for every gene-covariate interaction row
- Gene-specific directional checks hold, e.g. Mef2c flat over pseudotime under LPS yet downregulated under PAM

## Limitations

- PhenoPath assumes features evolve linearly with pseudotime; nonlinear effects may occur and cannot be discounted, so checks should verify the fit is consistent with the data
- Current PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings (deferred to future work)
- Significance depends on the choice of threshold k in the posterior-mean-over-posterior-SD rule

## Evidence

- [readme] CAVI posterior helpers: [evidence span withheld — anchor status: external_doc]
- [methods] Bayesian significance rule: "We classify or label an interaction as of interest if"
- [methods] significant_interactions helper: "significant_interactions(phenopath_fit, n)"
- [discussion] Linear evolution assumption: "An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime."
