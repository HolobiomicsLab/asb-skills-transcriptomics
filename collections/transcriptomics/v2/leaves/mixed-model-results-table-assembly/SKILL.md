---
name: mixed-model-results-table-assembly
description: Use when kmFit has been run with run_lme enabled on paired/repeated-measure
  RNA-seq counts and the per-gene kimma_lme output (estimates, SE, P-values, AIC/BIC)
  must be compiled into one validated results table.
when_to_use_negative:
- The design is unpaired with no donor/patient random effect — kimma's plain linear
  model path (kimma_lm) rather than the mixed-model path applies
- Results already exist as a per-gene data frame/table and only need reformatting
  rather than re-running the mixed-model workflow
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: kimma
  role: R package whose kmFit/kimma_lme functions fit the per-gene linear mixed models
    and report fit metrics that populate the results table
  repo: https://github.com/BIGslu/kimma
- name: lme4
  role: Imports dependency providing lmer, which actually fits the per-gene mixed
    models; must be explicitly loaded since Imports dependencies are not attached
    automatically
- name: devtools
  role: Used to install kimma from the BIGslu GitHub repository
  repo: https://github.com/BIGslu/kimma
- name: R
  role: Runtime environment for kimma and lme4
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:f0c193991496658a2ee462be4588cfcb2d798ee59dc441a18afac65fb11d03b5
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# mixed-model-results-table-assembly

## Summary

Assembles per-gene linear mixed-model results from kimma's kmFit (kimma_lme module, powered by lme4::lmer) — fixed-effect estimates, standard errors, P-values, and fit metrics (AIC/BIC) — into a single validated CSV table for paired-sample RNA-seq differential expression analysis.

## When to use

Use when kmFit has been run with run_lme enabled on paired/repeated-measure RNA-seq counts and the per-gene kimma_lme output (estimates, SE, P-values, AIC/BIC) must be compiled into one validated results table.

## When NOT to use

- The design is unpaired with no donor/patient random effect — kimma's plain linear model path (kimma_lm) rather than the mixed-model path applies
- Results already exist as a per-gene data frame/table and only need reformatting rather than re-running the mixed-model workflow

## Inputs

- Gene count matrix (e.g. example.count) from the kimma package
- Sample metadata with patient/donor and library identifiers (e.g. example.dat)
- Optional kinship/covariance matrix (e.g. example.kin) for related cohorts
- kimma model output object containing kimma_lme per-gene results

## Outputs

- kmfit_lme_results.csv — one row per gene with fixed-effect estimates, standard errors, P-values, and populated AIC/BIC fit-metric columns

## How to apply

After loading the kimma example data (example.count for gene counts, example.dat for sample metadata, example.kin if kinship is modeled), call kmFit with run_lme enabled and pass a donor/patient identifier (patientID) so lme4::lmer fits a per-gene linear mixed model with a random intercept for the paired design. kimma fits models using REML so P-values come from restricted maximum likelihood. Collect the kimma_lme output, which returns per-gene fixed-effect estimates, standard errors, P-values, and fit metrics (AIC/BIC); AIC is a distinguishing capability kimma provides unlike other DEG software. Stack the per-gene rows into one table and write it to kmfit_lme_results.csv.

## Related tools

- **kimma** (R package whose kmFit/kimma_lme functions fit the per-gene linear mixed models and report fit metrics that populate the results table) — https://github.com/BIGslu/kimma
- **lme4** (Imports dependency providing lmer, which actually fits the per-gene mixed models; must be explicitly loaded since Imports dependencies are not attached automatically)
- **devtools** (Used to install kimma from the BIGslu GitHub repository) — https://github.com/BIGslu/kimma
- **R** (Runtime environment for kimma and lme4)

## Examples

```
kmFit(dat = example.dat, counts = example.count, patientID = "patientID", libraryID = "libraryID", run.lme = TRUE)
```

## Evaluation signals

- kmfit_lme_results.csv exists and parses as valid CSV
- The table contains one row per modeled gene
- Fit-metric (AIC/BIC) columns are populated for the mixed-model results
- Fixed-effect estimates, standard errors, and P-values are present alongside AIC, reflecting kimma_lme's metrics output
- The model included the patientID random intercept, confirming a paired (mixed-effect) rather than fixed-only fit

## Limitations

- lme4 is an Imports (not Suggests) dependency of kimma and is not attached automatically, so fitting fails if it is not explicitly loaded
- Kinship covariance modeling (kimma_lmerel) requires a kin subset and patientID; plain kimma_lme output does not include kinship-based model fits
- kimma's reported P-values depend on REML estimation via lmer, so re-fitting with different likelihood settings changes the reported statistics

## Evidence

- [abstract] Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC): "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
- [methods] kimma_lme(model_lme, to_model_gene, gene, use_weights, metrics): "kimma_lme(model_lme, to_model_gene, gene, use_weights, metrics)"
- [intro] kimma (Kinship In Mixed Model Analysis), an open-source R package for flexible linear mixed effects modeling of RNA-seq: "kimma (Kinship In Mixed Model Analysis), an open-source R package for flexible linear mixed effects modeling of RNA-seq"
- [readme] devtools::install_github("BIGslu/kimma"): "devtools::install_github("BIGslu/kimma")"
