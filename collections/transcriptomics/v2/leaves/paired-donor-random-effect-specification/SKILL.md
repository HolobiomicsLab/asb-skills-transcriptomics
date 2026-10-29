---
name: paired-donor-random-effect-specification
description: Use when RNA-seq expression data contain paired samples or repeated measurements
  per donor (e.g. multiple libraries from the same patientID) and you need per-gene
  differential expression estimates that account for within-donor correlation.
when_to_use_negative:
- Samples are unpaired with no donor/patient grouping — the article benchmarks kimma
  against limma unpaired and dream paired models, and a random-intercept model is
  unnecessary without repeated measures.
- Kinship-related cohort modeling is the goal — that requires the kimma_lmerel path
  with the kin matrix (example.kin), not the plain random-intercept kimma_lme run.
- Data are single-cell or non-RNA-seq matrices incompatible with kmFit's counts/meta/genes
  argument structure.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: kimma
  role: R package providing kmFit and the kimma_lme routine that fits the per-gene
    donor random-intercept mixed model and returns fit metrics
  repo: https://github.com/BIGslu/kimma
- name: lme4
  role: Imports dependency whose lmer function performs the actual per-gene linear
    mixed model fit under REML; must be explicitly loaded
- name: devtools
  role: Used to install kimma from the BIGslu GitHub repository before running kmFit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:17eb3616b61ed69c3471e328d5a467b89f838d9f19479e360bf5275cbcfe05e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - devtools
  - kimma
  - lme4
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# paired-donor-random-effect-specification

## Summary

Specify paired-sample (repeated-measure) RNA-seq designs in kimma's kmFit as a per-gene linear mixed model with a donor/patient random intercept, fitted via lme4::lmer under REML, with per-gene fit metrics such as AIC reported.

## When to use

Use when RNA-seq expression data contain paired samples or repeated measurements per donor (e.g. multiple libraries from the same patientID) and you need per-gene differential expression estimates that account for within-donor correlation.

## When NOT to use

- Samples are unpaired with no donor/patient grouping — the article benchmarks kimma against limma unpaired and dream paired models, and a random-intercept model is unnecessary without repeated measures.
- Kinship-related cohort modeling is the goal — that requires the kimma_lmerel path with the kin matrix (example.kin), not the plain random-intercept kimma_lme run.
- Data are single-cell or non-RNA-seq matrices incompatible with kmFit's counts/meta/genes argument structure.

## Inputs

- Gene count matrix (e.g. kimma example.count)
- Sample metadata with donor/patient identifier (e.g. example.dat with patientID)
- Optional kinship covariance matrix (e.g. example.kin) for related-cohort designs
- lme4 fitted per gene after lme4 is explicitly loaded from kimma's Imports dependencies

## Outputs

- Per-gene mixed-model result table (kimma_lme output) with fixed-effect estimates, standard errors, P-values, and AIC/BIC fit metrics
- Compiled CSV of per-gene results (e.g. kmfit_lme_results.csv) with populated fit-metric columns

## How to apply

Load kimma's example data (example.count for gene counts, example.dat for sample metadata, example.kin if kinship is relevant), install kimma from BIGslu GitHub via devtools, and explicitly load lme4 since it is an Imports (not attached) dependency. Call kmFit on the metadata and gene matrix with run_lme enabled and patientID set to the donor identifier, so lme4::lmer fits a per-gene mixed model including a donor random intercept for the paired design. kmFit dispatches the kimma_lme routine, returning fixed-effect estimates, standard errors, P-values, and fit metrics (AIC/BIC) for each gene; collect the per-gene rows from kimma_lme into one table (e.g. kmfit_lme_results.csv). Rationale: the paper reports that kimma models paired designs as linear mixed effects via lme4::lmer with REML-based P-values, and — unlike other software — supports fit metrics like AIC for comparing model fits.

## Related tools

- **kimma** (R package providing kmFit and the kimma_lme routine that fits the per-gene donor random-intercept mixed model and returns fit metrics) — https://github.com/BIGslu/kimma
- **lme4** (Imports dependency whose lmer function performs the actual per-gene linear mixed model fit under REML; must be explicitly loaded)
- **devtools** (Used to install kimma from the BIGslu GitHub repository before running kmFit)

## Examples

```
Rscript -e 'library(kimma); library(lme4); fit <- kmFit(dat=example.dat, genes=example.count, patientID="patientID", run_lme=TRUE, metrics=TRUE)'
```

## Evaluation signals

- Output CSV exists, parses as valid CSV, and contains one row per gene.
- Fit-metric columns (AIC/BIC) are populated for each gene row, per the paper's claim that kimma reports fit metrics like AIC for mixed models.
- Each gene row includes fixed-effect estimates, standard errors, and P-values (REML-based) rather than only log-fold-changes.
- The donor/patient random effect appears in the model specification (patientID argument honored), distinguishable from a plain lm run without run_lme.
- Computational time is comparable to limma/dream paired models, consistent with the article's benchmarking claim.

## Limitations

- lme4 is an Imports (not attached) dependency of kimma, so the fit fails if lme4 is not explicitly loaded before kmFit run_lme.
- Fit metrics like AIC and covariance-matrix support are described as distinctive of kimma ('Unlike other software'), so cross-tool comparison of AIC values to limma/dream outputs is not straightforward.
- Plain donor random-intercept modeling (kimma_lme) does not incorporate genetic kinship; that requires the separate kimma_lmerel path with a kin matrix.

## Evidence

- [abstract] Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC): "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
- [abstract] kimma detects DEGs with similar specificity, sensitivity, and computational time as limma unpaired and dream paired models: "kimma detects DEGs with similar specificity, sensitivity, and computational time as limma unpaired and dream paired models"
- [methods] kmFit signature exposes the patientID random-effect argument for mixed modeling: "kmFit(dat, kin, patientID, libraryID, counts, meta, genes, weights, subset_var, subset_lvl"
- [methods] kimma_lme is the per-gene mixed-model routine returning fit metrics: "kimma_lme(model_lme, to_model_gene, gene, use_weights, metrics)"
- [readme] Installation of kimma from GitHub via devtools as prerequisite: "install.packages("devtools")
devtools::install_github("BIGslu/kimma")"
