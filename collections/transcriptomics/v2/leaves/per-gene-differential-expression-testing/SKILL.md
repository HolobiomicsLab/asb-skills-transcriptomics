---
name: per-gene-differential-expression-testing
description: Use when RNA-seq count data include paired samples or repeated measures
  from the same donor/patient (e.g. a patientID variable) and per-gene differential
  expression with random effects and fit metrics like AIC is needed.
when_to_use_negative:
- Samples are unpaired / independent groups with no donor or patient repeated-measure
  structure — a simple fixed-effect linear model (kimma's lm run) or limma unpaired
  would suffice
- The analysis goal is genotyping/association testing rather than RNA-seq differential
  expression — kmFit_eQTL, not the per-gene DE mixed model, is the relevant kimma
  entry point
- Kinship/covariance structure among related individuals is not of interest and no
  kinship matrix is available — although kimma still supports plain random effects
  via lme4::lmer, the kinship-specific value of the package is not engaged
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: kimma
  role: Primary R package; kmFit with run_lme fits the per-gene linear mixed model
    and returns per-gene estimates and AIC/BIC metrics
  repo: https://github.com/BIGslu/kimma
- name: lme4
  role: Backend for the linear mixed model; kimma calls lme4::lmer with a patientID
    random intercept and REML-based P-value estimation
- name: devtools
  role: 'Used to install kimma from GitHub: install.packages("devtools"); devtools::install_github("BIGslu/kimma")'
- name: limma
  role: Benchmark pipeline against which kimma's DEG detection specificity, sensitivity,
    and computational time were compared
- name: dream
  role: Benchmark paired mixed-model pipeline used for comparison of kimma's DEG detection
    performance
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:09a2d4d3f43fba5522dc989720105896dd0c9bb34105f5f402c6105602c695e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - devtools
  - dream
  - kimma
  - limma
  - lme4
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# per-gene-differential-expression-testing

## Summary

Fit a per-gene linear mixed effects model to RNA-seq counts using kimma's kmFit (backed by lme4::lmer) to detect differentially expressed genes in paired/repeated-measure designs, reporting fixed-effect estimates, standard errors, P-values, and fit metrics (AIC/BIC) for each gene.

## When to use

Use when RNA-seq count data include paired samples or repeated measures from the same donor/patient (e.g. a patientID variable) and per-gene differential expression with random effects and fit metrics like AIC is needed.

## When NOT to use

- Samples are unpaired / independent groups with no donor or patient repeated-measure structure — a simple fixed-effect linear model (kimma's lm run) or limma unpaired would suffice
- The analysis goal is genotyping/association testing rather than RNA-seq differential expression — kmFit_eQTL, not the per-gene DE mixed model, is the relevant kimma entry point
- Kinship/covariance structure among related individuals is not of interest and no kinship matrix is available — although kimma still supports plain random effects via lme4::lmer, the kinship-specific value of the package is not engaged

## Inputs

- Gene count matrix (e.g. kimma's example.count dataset)
- Sample metadata table (e.g. kimma's example.dat dataset, including patientID/libraryID columns)
- Optional kinship/covariance matrix (e.g. kimma's example.kin dataset)
- Optional voom-normalized weights (e.g. example.voom)

## Outputs

- Per-gene mixed-model results table (kimma_lme output) with fixed-effect estimates, standard errors, P-values, and AIC/BIC fit metrics
- Compiled CSV of per-gene results (e.g. kmfit_lme_results.csv)

## How to apply

Install kimma from the BIGslu GitHub repository with devtools (devtools::install_github("BIGslu/kimma")) and explicitly load lme4, since it is an Imports (not attached) dependency. Load the kimma example datasets — example.count for gene counts, example.dat for sample metadata, and example.kin if kinship covariance is needed — then call kmFit with run_lme enabled, passing counts, meta, and a patientID argument so lme4::lmer fits a per-gene linear mixed model with a donor/patient random intercept. P-values are estimated using restricted maximum likelihood (REML). Collect the kimma_lme output, which returns per-gene fixed-effect estimates, standard errors, P-values, and AIC/BIC fit metrics, and compile the rows into a single table (e.g. kmfit_lme_results.csv). Correctness is judged by verifying the output exists, is valid CSV, and has per-gene rows with populated fit-metric columns; the article validates the approach by showing kimma detects DEGs with specificity, sensitivity, and computational time comparable to limma unpaired and dream paired models.

## Related tools

- **kimma** (Primary R package; kmFit with run_lme fits the per-gene linear mixed model and returns per-gene estimates and AIC/BIC metrics) — https://github.com/BIGslu/kimma
- **lme4** (Backend for the linear mixed model; kimma calls lme4::lmer with a patientID random intercept and REML-based P-value estimation)
- **devtools** (Used to install kimma from GitHub: install.packages("devtools"); devtools::install_github("BIGslu/kimma"))
- **limma** (Benchmark pipeline against which kimma's DEG detection specificity, sensitivity, and computational time were compared)
- **dream** (Benchmark paired mixed-model pipeline used for comparison of kimma's DEG detection performance)

## Examples

```
library(kimma); library(lme4); kmFit(dat = example.count, meta = example.dat, patientID = "patientID", counts = TRUE, run_lme = TRUE)
```

## Evaluation signals

- Output file (e.g. kmfit_lme_results.csv) exists and parses as valid CSV
- Result table contains one row per gene with fixed-effect estimate, standard error, and P-value columns populated
- Fit-metric columns (AIC, and BIC where returned) are non-empty for modeled genes, per the paper's claim that kimma reports fit metrics like AIC
- Model formula reflects the paired design: a random intercept keyed on patientID was included, confirming lme4::lmer (not a fixed-only lm) was run
- DEG calls agree in direction/rank with limma unpaired and dream paired models, which the article reports give similar specificity and sensitivity

## Limitations

- lme4 is an Imports dependency of kimma and is not attached automatically, so it must be loaded explicitly or mixed-model fitting will fail
- REML-based P-value estimation is the documented mode for the mixed model; alternative inference approaches are not covered here
- Performance parity (specificity, sensitivity, computational time) was established against limma unpaired and dream paired in simulated datasets, not exhaustive real-world benchmarks

## Evidence

- [full_text] kimma models paired sample designs as linear mixed effects using lme4::lmer, with P-values estimated using restricted maximum likelihood (REML).: [evidence span withheld — anchor status: not_found]
- [abstract] Unlike other software, kimma supports covariance matrices as well as fit metrics like AIC.: "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
- [results] In simulated datasets, kimma detects DEGs with specificity, sensitivity, and computational time similar to limma unpaired and dream paired models.: "In simulated datasets, kimma detects DEGs with similar specificity, sensitivity, and computational time as limma unpaired and dream paired models"
- [methods] kmFit accepts counts, meta, patientID, and kinship inputs for per-gene modeling.: "kmFit(dat, kin, patientID, libraryID, counts, meta, genes, weights, subset_var, subset_lvl"
- [methods] kimma_lme is the function that fits the linear mixed effect model per gene with weights and metrics.: "kimma_lme(model_lme, to_model_gene, gene, use_weights, metrics)"
- [readme] Install kimma from GitHub via devtools, per the README installation instructions.: "install.packages("devtools")
devtools::install_github("BIGslu/kimma")"
