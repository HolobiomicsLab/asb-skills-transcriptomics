---
name: rna-seq-linear-mixed-effects-modeling
description: Use when RNA-seq differential expression data contain paired or repeated
  samples from the same donor (e.g., pre/post or within-patient designs) and you need
  per-gene mixed-model P-values plus AIC/BIC fit metrics.
when_to_use_negative:
- The design is unpaired or has no repeated measures per donor — a plain linear model
  (kimma_lm / limma-style) or a paired-specific competitor like dream may be simpler
  and the paper benchmarks kimma against limma unpaired and dream paired models rather
  than claiming superiority in all settings
- The goal is genotype–expression association rather than differential expression
  — use kimma's dedicated kmFit_eQTL entry point instead of kmFit with run_lme
- You need kinship-adjusted relatedness modeling with the kinship covariance (kimma_lmerel
  path) rather than a simple donor random intercept — the generic run_lme path does
  not incorporate kin
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: kimma
  role: Core R package; kmFit with run_lme drives per-gene linear mixed effects fitting
    (kimma_lme) and returns AIC/BIC fit metrics
  repo: https://github.com/BIGslu/kimma
- name: lme4
  role: Imports dependency providing lme4::lmer, which actually fits each per-gene
    mixed model with a patientID random intercept under REML; must be loaded explicitly
- name: devtools
  role: Installs kimma from GitHub (BIGslu/kimma) when not on CRAN
- name: R
  role: Runtime environment for the kimma package and mixed model fitting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:3e55f9783a0b5b4a0ea6096450ecfb367094f039b7c659bf4cdcd88506f87ecb
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
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# rna-seq-linear-mixed-effects-modeling

## Summary

Fit per-gene linear mixed effects models to RNA-seq counts with kimma's kmFit (backed by lme4::lmer under REML) to handle paired/repeated-measure designs via a patientID random intercept, returning fixed-effect estimates, standard errors, P-values, and fit metrics (AIC/BIC) for each gene.

## When to use

Use when RNA-seq differential expression data contain paired or repeated samples from the same donor (e.g., pre/post or within-patient designs) and you need per-gene mixed-model P-values plus AIC/BIC fit metrics.

## When NOT to use

- The design is unpaired or has no repeated measures per donor — a plain linear model (kimma_lm / limma-style) or a paired-specific competitor like dream may be simpler and the paper benchmarks kimma against limma unpaired and dream paired models rather than claiming superiority in all settings
- The goal is genotype–expression association rather than differential expression — use kimma's dedicated kmFit_eQTL entry point instead of kmFit with run_lme
- You need kinship-adjusted relatedness modeling with the kinship covariance (kimma_lmerel path) rather than a simple donor random intercept — the generic run_lme path does not incorporate kin

## Inputs

- example.count — per-gene RNA-seq count matrix (from the installed kimma package)
- example.dat — sample metadata with donor/patient identifiers for pairing
- example.kin — genetic kinship covariance matrix (optional, for related cohorts)
- gene weights (e.g., example.voom / example.seasnake intermediates, optional)

## Outputs

- kimma_lme per-gene results: fixed-effect estimates, standard errors, P-values, AIC/BIC fit metrics
- kmfit_lme_results.csv — compiled per-gene mixed-model table with fit-metric columns populated

## How to apply

Load the kimma example datasets (example.count for gene counts, example.dat for sample metadata, example.kin if kinship is needed) and install kimma from the BIGslu GitHub repository via devtools::install_github("BIGslu/kimma"); explicitly attach lme4, which is an Imports dependency and is not attached automatically. Call kmFit with run_lme enabled so lme4::lmer fits one linear mixed model per gene, passing the donor/patient identifier (patientID argument) as a random intercept to capture within-donor correlation in the paired design; optionally pass libraryID, counts, weights, or a kinship matrix (kin) depending on design. Collect the kimma_lme output, which returns fixed-effect estimates, standard errors, P-values, and fit metrics (AIC/BIC) for each gene, then compile per-gene rows into a single table and write kmfit_lme_results.csv. Validate that the CSV exists, parses, and has populated AIC columns — matching the paper's claim that kimma reports fit metrics like AIC for mixed models, unlike other DEG software.

## Related tools

- **kimma** (Core R package; kmFit with run_lme drives per-gene linear mixed effects fitting (kimma_lme) and returns AIC/BIC fit metrics) — https://github.com/BIGslu/kimma
- **lme4** (Imports dependency providing lme4::lmer, which actually fits each per-gene mixed model with a patientID random intercept under REML; must be loaded explicitly)
- **devtools** (Installs kimma from GitHub (BIGslu/kimma) when not on CRAN)
- **R** (Runtime environment for the kimma package and mixed model fitting)

## Examples

```
devtools::install_github("BIGslu/kimma"); library(kimma); library(lme4); kmFit(dat=example.count, meta=example.dat, patientID="patientID", run_lme=TRUE)
```

## Evaluation signals

- Output file kmfit_lme_results.csv exists and parses as valid CSV
- Result table contains one row per gene with AIC/BIC fit-metric columns populated (paper claims kimma reports fit metrics like AIC, unlike other software)
- Mixed-model rows include fixed-effect estimates, standard errors, and P-values estimated under REML
- Random-intercept term for patientID appears in the fitted model output, confirming the paired design was modeled as a mixed effect
- P-values are non-degenerate (not all NA/1) after accounting for the donor random intercept, indicating within-donor correlation was captured

## Limitations

- lme4 is an Imports (not attached) dependency of kimma, so it must be explicitly loaded or the mixed model path will fail
- kimma's claimed advantage over limma unpaired and dream paired models is parity (similar specificity, sensitivity, computational time), not universal outperformance
- Computational cost scales per gene; the paper reports comparable computational time to limma/dream rather than a speed win

## Evidence

- [abstract] kimma supports covariance matrices as well as fit metrics like AIC, unlike other software: "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
- [other] kimma models paired designs as linear mixed effects fit by lme4::lmer under REML: [evidence span withheld — anchor status: not_found]
- [methods] kmFit signature exposing patientID, libraryID, counts, kin, weights parameters: "kmFit(dat, kin, patientID, libraryID, counts, meta, genes, weights, subset_var, subset_lvl"
- [methods] kimma_lme per-gene mixed model fitting routine with metrics: "kimma_lme(model_lme, to_model_gene, gene, use_weights, metrics)"
- [readme] Installation route via devtools from the BIGslu/kimma GitHub repository: "install.packages("devtools")
devtools::install_github("BIGslu/kimma")"
