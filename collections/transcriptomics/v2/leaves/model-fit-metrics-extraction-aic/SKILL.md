---
name: model-fit-metrics-extraction-aic
description: Use when a paired/repeated-measure RNA-seq differential expression analysis
  needs per-gene fit metrics (AIC/BIC) to compare mixed-effects models, since kimma
  is the only tool among limma/dream that reports AIC for its models.
when_to_use_negative:
- The design is unpaired with no random effect or kinship to compare — kimma_lm (fixed-effect
  lm) models suffice and mixed-model AIC is not the target quantity.
- The goal is genotype/eQTL association modeling (kmFit_eQTL with dat_snp/dat_map),
  which is a separate kimma workflow rather than per-gene fit-metric extraction.
- The model must include a genetic kinship covariance matrix (kimma_lmerel with kin_subset);
  plain kimma_lme fit metrics do not capture kinship's effect on model fit.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: kimma
  role: Primary R package whose kmFit/kimma_lme functions fit the per-gene linear
    mixed model and report AIC/BIC fit metrics
  repo: https://github.com/BIGslu/kimma
- name: lme4
  role: Backend used by kimma_lme to fit each gene's linear mixed model via lme4::lmer
    with REML-based P-value estimation
- name: devtools
  role: Used to install kimma from GitHub before fitting
- name: R
  role: Runtime environment in which kimma, lme4, and devtools are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:cb9a7ecc89728430b3533cc4cc1b13802d952d0818a4d53023fe8d889d4ba8a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - devtools
  - kimma
  - lme4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Model fit metrics extraction (AIC) from kimma mixed models

## Summary

Extract per-gene model fit metrics (AIC/BIC) from kimma's kmFit linear mixed effects runs, which fit RNA-seq paired designs via lme4::lmer and uniquely report fit metrics alongside fixed-effect estimates and P-values.

## When to use

Use when a paired/repeated-measure RNA-seq differential expression analysis needs per-gene fit metrics (AIC/BIC) to compare mixed-effects models, since kimma is the only tool among limma/dream that reports AIC for its models.

## When NOT to use

- The design is unpaired with no random effect or kinship to compare — kimma_lm (fixed-effect lm) models suffice and mixed-model AIC is not the target quantity.
- The goal is genotype/eQTL association modeling (kmFit_eQTL with dat_snp/dat_map), which is a separate kimma workflow rather than per-gene fit-metric extraction.
- The model must include a genetic kinship covariance matrix (kimma_lmerel with kin_subset); plain kimma_lme fit metrics do not capture kinship's effect on model fit.

## Inputs

- Gene count matrix (e.g. kimma example.count) per gene/sample
- Sample metadata table (e.g. kimma example.dat) with patientID and libraryID columns
- Optional kinship covariance matrix (e.g. example.kin) for kimma_lmerel comparisons
- Kimma R package installed from BIGslu GitHub; lme4 loaded explicitly

## Outputs

- Per-gene mixed-model results table with fixed-effect estimates, standard errors, P-values, and AIC/BIC fit metrics
- Compiled CSV file (e.g. kmfit_lme_results.csv) with one row per gene and populated fit-metric columns

## How to apply

Install kimma from GitHub (devtools::install_github("BIGslu/kimma")) and explicitly load lme4, which is an Imports dependency kimma does not attach automatically. Run kmFit on example.count-style gene counts with example.dat-style sample metadata, enabling the linear mixed model run (kimma_lme) with a patientID random intercept for the paired design and requesting fit metrics. The per-gene mixed-model output returns fixed-effect estimates, standard errors, P-values, and fit metrics (AIC/BIC) for each gene. Compile these results into a single table (e.g. kmfit_lme_results.csv) and validate that the CSV exists with per-gene rows and populated AIC columns; AIC can then be compared against the non-mixed kimma_lm fit or kinship-based kimma_lmerel fit to judge whether the random effect or kinship covariance improves model fit.

## Related tools

- **kimma** (Primary R package whose kmFit/kimma_lme functions fit the per-gene linear mixed model and report AIC/BIC fit metrics) — https://github.com/BIGslu/kimma
- **lme4** (Backend used by kimma_lme to fit each gene's linear mixed model via lme4::lmer with REML-based P-value estimation)
- **devtools** (Used to install kimma from GitHub before fitting)
- **R** (Runtime environment in which kimma, lme4, and devtools are executed)

## Examples

```
library(kimma); library(lme4); fit <- kmFit(dat = example.dat, counts = example.count, patientID = "patientID", run_lme = TRUE)
```

## Evaluation signals

- The output CSV (kmfit_lme_results.csv) exists and parses as valid CSV.
- The results table contains one row per gene with gene identifiers populated.
- Fit-metric columns (AIC/BIC) are present and non-NA for modeled genes, matching the paper's claim that kimma reports AIC for mixed models.
- Fixed-effect estimates, standard errors, and P-values accompany each gene's fit metrics in the kimma_lme output.
- AIC values differ between the kimma_lm (fixed) and kimma_lme (mixed) fits where a donor/patient random intercept meaningfully changes model fit.

## Limitations

- Fit metrics are only returned when metrics are requested in the kimma_lme/kimma_lm/kimma_lmerel function signatures; without this, estimates and P-values are returned without AIC/BIC.
- lme4 is an Imports dependency of kimma and is not attached automatically, so calls fail unless it is explicitly loaded.
- Fit metrics for kinship-affected models require kimma_lmerel with a kinship matrix and patientID; kimma_lme metrics alone do not reflect kinship's impact on model fit.

## Evidence

- [abstract] kimma supports covariance matrices as well as fit metrics like AIC, unlike other software: "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
- [methods] kimma_lme function signature with metrics parameter: "kimma_lme(model_lme, to_model_gene, gene, use_weights, metrics)"
- [methods] kmFit main entry point accepting patientID and other model arguments: "kmFit(dat, kin, patientID, libraryID, counts, meta, genes, weights, subset_var, subset_lvl"
- [methods] kimma_lmerel signature showing kinship-based model variant with metrics: "kimma_lmerel(model_lme, to_model_gene, gene, kin_subset, use_weights, patientID, metrics)"
- [readme] README installation instructions for kimma: "install.packages("devtools")
devtools::install_github("BIGslu/kimma")"
- [intro] kimma as R package with covariates, random effects, and fit metrics: "kimma (Kinship In Mixed Model Analysis), an open-source R package for flexible linear mixed effects modeling of RNA-seq including covariates, weights, random effects, covariance"
