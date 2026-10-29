---
name: tabular-result-summarization
description: Use when per-gene result tables from paired/kinship vs non-kinship linear
  mixed model fits need to be thresholded at FDR < 0.05 and compared as DEG sets (new,
  lost, and agreeing calls).
when_to_use_negative:
- Input is raw RNA-seq count data requiring model fitting — use kmFit/kimma_lm/kimma_lme/kimma_lmerel
  first; this skill only summarises existing per-gene fit results.
- The analysis goal is model fitting or fit-metric comparison (AIC) rather than thresholding
  FDR tables into DEG call sets.
- Only a single model's results exist — set comparison and agreement percentages require
  results from both the kinship and non-kinship fits.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: kimma
  role: provides summarise_kmFit (and kmFit) to threshold per-gene FDR results at
    fdr.cutoff = 0.05 into DEG call sets
  repo: https://github.com/BIGslu/kimma
- name: dplyr
  role: set operations (new/lost DEGs, percentage agreement) on the two thresholded
    DEG sets
- name: data.table
  role: fast comparison and aggregation of the per-gene result tables
- name: R
  role: runtime environment; kimma is installed via devtools from GitHub
  repo: https://github.com/BIGslu/kimma
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:3ce46efa96155d4d56454dcddfd7e58701a0733fae9d7851e70b8980953a353e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - data.table
  - dplyr
  - kimma
  - tibble
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# tabular-result-summarization

## Summary

Thresholds and summarises per-gene linear mixed model fit results (e.g. kimma kmFit FDR tables) into concrete DEG sets at a chosen cutoff (FDR < 0.05) and writes set-comparison statistics such as newly detected DEGs and percentage agreement between model variants to a summary CSV.

## When to use

Use when per-gene result tables from paired/kinship vs non-kinship linear mixed model fits need to be thresholded at FDR < 0.05 and compared as DEG sets (new, lost, and agreeing calls).

## When NOT to use

- Input is raw RNA-seq count data requiring model fitting — use kmFit/kimma_lm/kimma_lme/kimma_lmerel first; this skill only summarises existing per-gene fit results.
- The analysis goal is model fitting or fit-metric comparison (AIC) rather than thresholding FDR tables into DEG call sets.
- Only a single model's results exist — set comparison and agreement percentages require results from both the kinship and non-kinship fits.

## Inputs

- per-gene FDR result tables from kimma_lmerel (kinship linear mixed model) fits
- per-gene FDR result tables from kimma_lme (non-kinship linear mixed model) fits
- example cohort data (example.count, example.dat, example.kin) as fitted upstream

## Outputs

- kinship-model DEG set defined at FDR < 0.05
- non-kinship-model DEG set defined at FDR < 0.05
- deg_kinship_comparison.csv containing comparison counts and percentage agreement of DEG calls

## How to apply

Load the per-gene FDR result tables produced by upstream kimma_lmerel (kinship) and kimma_lme (non-kinship) model fits. Apply summarise_kmFit with fdr.cutoff = 0.05 (the function also accepts p_cutoff, FCgroup, and intercept arguments) to flag genes as DEGs in each model. Use dplyr/data.table to compute set comparisons: DEGs newly detected with the kinship covariance random effect, DEGs detected only without kinship, and the percentage agreement of DEG calls between the two models. Write the comparison counts and agreement percentage to deg_kinship_comparison.csv. Rationale: the kimma paper shows kinship impacts model fit and DEG detection in a related cohort (e.g. 25 new DEGs detected at FDR < 0.05 in the paired Mtb kinship model), so summarising and diffing the two call sets quantifies that impact.

## Related tools

- **kimma** (provides summarise_kmFit (and kmFit) to threshold per-gene FDR results at fdr.cutoff = 0.05 into DEG call sets) — https://github.com/BIGslu/kimma
- **dplyr** (set operations (new/lost DEGs, percentage agreement) on the two thresholded DEG sets)
- **data.table** (fast comparison and aggregation of the per-gene result tables)
- **R** (runtime environment; kimma is installed via devtools from GitHub) — https://github.com/BIGslu/kimma

## Examples

```
devtools::install_github("BIGslu/kimma"); kmfit_summ <- summarise_kmFit(fit, fdr.cutoff = 0.05)
```

## Evaluation signals

- Both thresholded sets contain only genes with FDR < 0.05 in the respective kinship or non-kinship model table.
- The three comparison categories (new with kinship, only without kinship, agreeing) are mutually exclusive and their sizes reconcile with the total unique DEGs across both models.
- Percentage agreement is in [0, 100] and consistent with the reported counts (e.g. 25 new DEGs with kinship at FDR < 0.05).
- deg_kinship_comparison.csv exists, parses as a table, and contains both the comparison counts and the agreement percentage.
- Re-running the thresholding on the same FDR tables reproduces identical DEG sets (deterministic cutoff, no stochastic step).

## Limitations

- Results depend on the upstream model specification (kimma_lme vs kimma_lmerel with kinship covariance); different random effects or covariates change the DEG sets being compared.
- The comparison reflects one cohort and one cutoff (FDR < 0.05); kinship impact on DEG detection may differ at other cutoffs or in unrelated cohorts.
- kimma requires the kinship matrix (e.g. example.kin) and patient identifiers, so the kinship-model call set is only available for genetically related designs.

## Evidence

- [methods] summarise_kmFit(fdr, fdr_cutoff, p_cutoff, FCgroup, intercept, fdr.cutoff, p.cutoff): "summarise_kmFit(fdr, fdr_cutoff, p_cutoff, FCgroup, intercept, fdr.cutoff, p.cutoff)"
- [abstract] kinship impacts DEG detection: "Utilizing genetic kinship covariance, kimma revealed that kinship impacts model fit and DEG detection in a related cohort"
- [methods] kimma_lmerel filter step: "kimma_lmerel(model_lme, to_model_gene, gene, kin_subset, use_weights, patientID, metrics)"
- [readme] devtools install: "install.packages("devtools")
devtools::install_github("BIGslu/kimma")"
