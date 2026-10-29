---
name: deg-set-overlap-and-agreement-analysis
description: Use when you have two DEG result tables from alternative paired linear
  mixed model fits (e.g. with and without a kinship covariance random effect) and
  need to quantify how model specification changes detection at a fixed FDR cutoff
  such as 0.05.
when_to_use_negative:
- Input is a single model fit (e.g. only one kmFit/limma/dream result) rather than
  two alternative fits of the same data to contrast as sets
- The two model fits were run on different cohorts or different response variables,
  making set agreement meaningless
- The goal is to detect DEGs itself rather than to compare detection outcomes between
  model specifications — use kimma kmFit / kmFit_eQTL directly instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: kimma
  role: Source of the two model fits being compared (kimma_lmerel with kinship vs
    kimma_lme without) and of summarise_kmFit, whose fdr.cutoff argument thresholds
    genes to define each DEG set
  repo: https://github.com/BIGslu/kimma
- name: dplyr
  role: 'Set comparisons in R: identifying DEGs newly detected with kinship, DEGs
    only detected without kinship, and call agreement'
- name: data.table
  role: Set comparison and tabulation of DEG counts across the two result tables in
    R
- name: devtools
  role: Used to install kimma from GitHub (devtools::install_github("BIGslu/kimma"))
    before fitting the compared models
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:3dadbbf532ead8ab4f69c0c8dce57f68586a6b93ac5fa3a73e96e81f90595d53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - data.table
  - devtools
  - dplyr
  - kimma
  - tibble
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# deg-set-overlap-and-agreement-analysis

## Summary

Compares two differential expression result tables from alternative paired RNA-seq linear mixed model fits (e.g. kimma kimma_lmerel with kinship covariance vs kimma_lme without) as gene sets at FDR < 0.05, quantifying DEGs newly detected, DEGs lost, and percentage agreement between model calls.

## When to use

Use when you have two DEG result tables from alternative paired linear mixed model fits (e.g. with and without a kinship covariance random effect) and need to quantify how model specification changes detection at a fixed FDR cutoff such as 0.05.

## When NOT to use

- Input is a single model fit (e.g. only one kmFit/limma/dream result) rather than two alternative fits of the same data to contrast as sets
- The two model fits were run on different cohorts or different response variables, making set agreement meaningless
- The goal is to detect DEGs itself rather than to compare detection outcomes between model specifications — use kimma kmFit / kmFit_eQTL directly instead

## Inputs

- Per-gene FDR result table from kimma_lmerel fit (paired model with kinship covariance)
- Per-gene FDR result table from kimma_lme fit (paired model without kinship)
- FDR cutoff value, e.g. fdr.cutoff = 0.05 passed to summarise_kmFit

## Outputs

- Count of DEGs newly detected with kinship at FDR < 0.05
- Count of DEGs detected only without kinship
- Percentage agreement of DEG calls between the two models
- deg_kinship_comparison.csv with comparison counts and agreement percentage

## How to apply

Load the per-gene FDR result tables from the upstream kimma_lmerel (kinship) and kimma_lme (non-kinship) model fits on the same cohort. Threshold each gene at FDR < 0.05 (the fdr.cutoff argument of summarise_kmFit) to define the kinship-model and non-kinship-model DEG sets in R. Use dplyr/data.table set operations to compute DEGs newly detected with kinship (setdiff), DEGs detected only without kinship, and the percentage agreement of calls between the two models. The rationale is that kimma's genetic kinship covariance revealed kinship impacts model fit and DEG detection in a related cohort, so the same data can yield different DEG sets under different model specifications. Write the comparison counts and agreement percentage to deg_kinship_comparison.csv. In the motivating analysis, adding a kinship covariance random effect to the paired Mtb infected versus media model of genetically related individuals yielded 25 new DEGs detected in the kinship model at FDR < 0.05.

## Related tools

- **kimma** (Source of the two model fits being compared (kimma_lmerel with kinship vs kimma_lme without) and of summarise_kmFit, whose fdr.cutoff argument thresholds genes to define each DEG set) — https://github.com/BIGslu/kimma
- **dplyr** (Set comparisons in R: identifying DEGs newly detected with kinship, DEGs only detected without kinship, and call agreement)
- **data.table** (Set comparison and tabulation of DEG counts across the two result tables in R)
- **devtools** (Used to install kimma from GitHub (devtools::install_github("BIGslu/kimma")) before fitting the compared models)

## Examples

```
kin_degs <- summarise_kmFit(fit_lmerel, fdr.cutoff = 0.05); nonkin_degs <- summarise_kmFit(fit_lme, fdr.cutoff = 0.05); new_with_kinship <- setdiff(kin_degs$gene, nonkin_degs$gene)
```

## Evaluation signals

- The FDR < 0.05 cutoff is applied identically to both model fits (same fdr.cutoff passed to summarise_kmFit for the kimma_lmerel and kimma_lme tables)
- Counts are internally consistent: kinship-set size = newly-detected + shared, non-kinship-set size = lost-only + shared
- Percentage agreement falls in [0, 100] and equals shared DEGs relative to the union (or both model calls) as documented in the comparison
- deg_kinship_comparison.csv exists and contains the comparison counts plus the agreement percentage
- The comparison reproduces the paper's outcome pattern: a non-empty set of new DEGs after adding kinship (25 new DEGs in the paired Mtb model at FDR < 0.05)

## Limitations

- Results depend on the chosen FDR cutoff (fdr.cutoff); agreement counts are not meaningful if different cutoffs are applied to the two fits
- The kinship model comparison requires related individuals and a genetic kinship matrix; with unrelated samples the kimma_lmerel fit (and hence this contrast) is not applicable
- kimma reports similar specificity, sensitivity, and computational time to limma unpaired and dream paired models in simulations, so set differences reflect model specification (kinship covariance, random effects, AIC-based fit metrics) rather than a uniform detection advantage

## Evidence

- [abstract] Kinship alters DEG detection: "Utilizing genetic kinship covariance, kimma revealed that kinship impacts model fit and DEG detection in a related cohort"
- [full_text] Finding: 25 new DEGs with kinship: [evidence span withheld — anchor status: not_found]
- [methods] Threshold via summarise_kmFit fdr.cutoff: "summarise_kmFit(fdr, fdr_cutoff, p_cutoff, FCgroup, intercept, fdr.cutoff, p.cutoff)"
- [abstract] Benchmark against limma and dream: "kimma detects DEGs with similar specificity, sensitivity, and computational time as limma unpaired and dream paired models"
- [readme] kimma capabilities relevant to the compared fits: "an open-source R package for flexible linear mixed effects modeling of RNA-seq including covariates, weights, random effects, covariance matrices, and fit metrics"
