---
name: rna-seq-tmm-normalization-log2-cpm
description: Use when you have raw gene-level counts (e.g. kimma's example.count)
  plus sample metadata (example.dat) and want per-gene unpaired linear-model DEG testing
  on TMM-normalized log2 CPM expression with fit metrics.
when_to_use_negative:
- Data are paired or have repeated measures per patient — use kimma's lme/kimship
  (kimma_lme, kimma_lmerel) modes instead of run_lm
- Input is already normalized (e.g. example.voom voom-transformed data) or you need
  voom/limma-style precision weights — this skill models TMM log2 CPM without weights
- You need kinship covariance or genotype QTL modeling — those require kmFit's kin
  argument or kmFit_eQTL, not the unpaired lm path
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: kimma
  role: 'Runs the unpaired linear-model workflow: kmFit with run_lm=TRUE normalizes
    counts to TMM log2 CPM and dispatches to kimma_lm per gene'
  repo: https://github.com/BIGslu/kimma
- name: R
  role: Runtime environment and package installation (devtools::install_github) for
    kimma
- name: stats
  role: Base R linear model engine (stats::lm) used by kimma_lm for the per-gene OLS
    fits
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:70bd5cdee33a81de775e21fca8e9a57dfc0218ee7d9ce5dcdb09eaa1fb4f5d25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - kimma
  - stats
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# rna-seq-tmm-normalization-log2-cpm

## Summary

Fits per-gene unpaired linear models (stats::lm) on RNA-seq expression expressed as TMM-normalized log2 CPM, using kimma's kmFit/kimma_lm machinery to produce per-gene coefficients, fit metrics, and p-values/FDR for DEG detection.

## When to use

Use when you have raw gene-level counts (e.g. kimma's example.count) plus sample metadata (example.dat) and want per-gene unpaired linear-model DEG testing on TMM-normalized log2 CPM expression with fit metrics.

## When NOT to use

- Data are paired or have repeated measures per patient — use kimma's lme/kimship (kimma_lme, kimma_lmerel) modes instead of run_lm
- Input is already normalized (e.g. example.voom voom-transformed data) or you need voom/limma-style precision weights — this skill models TMM log2 CPM without weights
- You need kinship covariance or genotype QTL modeling — those require kmFit's kin argument or kmFit_eQTL, not the unpaired lm path

## Inputs

- Gene-level count matrix (kimma example.count, counts argument to kmFit)
- Sample metadata data.frame (kimma example.dat, meta argument)
- Gene list to model (genes argument to kmFit)

## Outputs

- Per-gene lm results table with coefficient estimates, fit metrics (e.g. AIC), and p-values/FDR
- Written CSV of results, e.g. kimma_lm_results.csv, one record per modeled gene

## How to apply

Load the kimma R package and its bundled example datasets example.count and example.dat, matching the instructional vignette. Call kmFit with run_lm=TRUE (unpaired linear model mode), supplying the counts, meta, and genes arguments; expression is handled internally as TMM-normalized log2 CPM. One ordinary least-squares model is then fit per gene through the kimma_lm machinery (stats::lm), with the metrics argument enabled for AIC-style fit metrics and use_weights left unset so fitting is unweighted. Collect the returned per-gene results table with coefficient estimates, fit metrics, and p-values/FDR, and write it out (e.g. kimma_lm_results.csv).

## Related tools

- **kimma** (Runs the unpaired linear-model workflow: kmFit with run_lm=TRUE normalizes counts to TMM log2 CPM and dispatches to kimma_lm per gene) — https://github.com/BIGslu/kimma
- **R** (Runtime environment and package installation (devtools::install_github) for kimma)
- **stats** (Base R linear model engine (stats::lm) used by kimma_lm for the per-gene OLS fits)

## Examples

```
devtools::install_github("BIGslu/kimma"); kmFit(dat = example.count, meta = example.dat, genes = genes, run_lm = TRUE, metrics = TRUE)
```

## Evaluation signals

- kimma_lm_results.csv exists and contains one record per modeled gene from example.count/example.dat
- Each record has estimate, fit-metric, and p-value (plus FDR) fields populated by the run_lm=TRUE kmFit call
- Results are consistent with the unpaired lm-mode run shown in the kimma vignette (https://bigslu.github.io/kimma_vignette/kimma_vignette.html)
- Expression scale matches TMM-normalized log2 CPM and use_weights was unset (unweighted fits), since no weights were supplied

## Limitations

- Unpaired lm mode does not handle patient-level pairing or kinship covariance; for related cohorts kimma uses genetic kinship covariance (kimma_lmerel), which the paper showed impacts model fit and DEG detection
- By default this path is unweighted; kimma also supports weights (e.g. voom-derived, as in example.voom) that are not used here
- Kimma is intended for RNA-seq DEG modeling; adding covariates, random effects, or covariance matrices changes the appropriate mode (lm vs lme/lmerel)

## Evidence

- [methods] kimma_lm model_lm, to_model_gene, gene, use_weights, metrics: "kimma_lm(model_lm, to_model_gene, gene, use_weights, metrics)"
- [methods] kmFit signature including counts/meta/genes arguments: "kmFit(dat, kin, patientID, libraryID, counts, meta, genes, weights, subset_var, subset_lvl"
- [intro] kimma is an R package for linear (mixed effects) RNA-seq modeling: "kimma (Kinship In Mixed Model Analysis), an open-source R package for flexible linear mixed effects modeling of RNA-seq"
- [readme] Installation from GitHub via devtools: "install.packages("devtools")
devtools::install_github("BIGslu/kimma")"
- [abstract] Fit metrics such as AIC supported, distinguishing kimma from other tools: "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
