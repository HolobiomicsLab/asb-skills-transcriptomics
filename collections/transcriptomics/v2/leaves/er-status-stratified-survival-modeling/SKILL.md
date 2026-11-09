---
name: er-status-stratified-survival-modeling
description: Use when bulk tumour samples (e.g. TCGA-BRCA) have a continuous pseudotime
  score from PhenoPath plus overall survival time/status and ER status, and the goal
  is to test pseudotime as an independent survival predictor.
when_to_use_negative:
- Pseudotimes were inferred jointly across ER+ and ER− tumours with ER only as a covariate
  rather than per-stratum trajectories — then a stratified design is not motivated
- The endpoint is not time-to-event survival data (e.g. only binary responder status)
  — coxph with Surv() is the wrong model class
- 'Input is single-cell data: the article applies PhenoPath pseudotime to bulk TCGA
  cohorts for survival association, not to individual cells'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: survival (R package, coxph)
  role: Fits the ER-stratified Cox proportional hazards model with pseudotime as predictor
    and extracts coefficient, hazard ratio, CI and Wald p-value
- name: PhenoPath
  role: Upstream method producing the covariate-modulated pseudotimes (ER-stratified
    angiogenesis trajectory) that feed the survival model
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: dplyr
  role: Merging PhenoPath pseudotimes with TCGA-BRCA survival and clinical covariate
    tables
- name: tibble
  role: Structuring the merged survival analysis data frame
- name: ggplot2
  role: Visualising the survival association (e.g. Supplementary Fig 17)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:ecf910f679323859f1feefdbffc913dc8f28b0aa817a4643e0c9c0c888516a33
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PhenoPath
  - R
  - dplyr
  - ggplot2
  - survival (R package, coxph)
  - tibble
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# er-status-stratified-survival-modeling

## Summary

Tests whether PhenoPath-inferred pseudotemporal progression (e.g. the TCGA-BRCA angiogenesis trajectory) is prognostic for overall survival using a Cox proportional hazards model stratified by ER status, adjusted for clinical covariates such as stage and age.

## When to use

Use when bulk tumour samples (e.g. TCGA-BRCA) have a continuous pseudotime score from PhenoPath plus overall survival time/status and ER status, and the goal is to test pseudotime as an independent survival predictor.

## When NOT to use

- Pseudotimes were inferred jointly across ER+ and ER− tumours with ER only as a covariate rather than per-stratum trajectories — then a stratified design is not motivated
- The endpoint is not time-to-event survival data (e.g. only binary responder status) — coxph with Surv() is the wrong model class
- Input is single-cell data: the article applies PhenoPath pseudotime to bulk TCGA cohorts for survival association, not to individual cells

## Inputs

- ER-stratified PhenoPath pseudotimes for TCGA-BRCA tumours (output of the upstream PhenoPath trajectory-inference task)
- TCGA-BRCA overall survival times and event status
- TCGA-BRCA clinical covariates (e.g. stage, age, ER status)

## Outputs

- Result record (tcga_brca_cox_results.csv) containing the pseudotime Cox coefficient, hazard ratio, confidence interval, and Wald p-value
- A fitted ER-stratified coxph model object that can be plotted with ggplot2 (e.g. Supplementary Fig 17-style survival association)

## How to apply

Merge the ER-stratified PhenoPath pseudotimes with TCGA-BRCA overall survival times/status and clinical covariates using dplyr and tibble in R. Fit a Cox proportional hazards model with the survival package's coxph, entering pseudotime as the primary predictor, stage and age as adjustment covariates, and ER status as a stratification factor via strata() rather than as a covariate, since the trajectory was inferred separately per ER stratum. Extract the pseudotime coefficient, hazard ratio, 95% confidence interval, and Wald p-value from the fitted model summary into a result data frame, and write it to tcga_brca_cox_results.csv. The stratified design is the rationale: it lets baseline hazards differ between ER+ and ER− tumours while sharing a single pseudotime effect.

## Related tools

- **survival (R package, coxph)** (Fits the ER-stratified Cox proportional hazards model with pseudotime as predictor and extracts coefficient, hazard ratio, CI and Wald p-value)
- **PhenoPath** (Upstream method producing the covariate-modulated pseudotimes (ER-stratified angiogenesis trajectory) that feed the survival model) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **dplyr** (Merging PhenoPath pseudotimes with TCGA-BRCA survival and clinical covariate tables)
- **tibble** (Structuring the merged survival analysis data frame)
- **ggplot2** (Visualising the survival association (e.g. Supplementary Fig 17))

## Examples

```
library(survival); fit <- coxph(Surv(OS_time, OS_status) ~ pseudotime + age + stage + strata(ER_status), data = tcga_brca); summary(fit)
```

## Evaluation signals

- Output file tcga_brca_cox_results.csv exists and contains the coefficient, hazard ratio, confidence-interval, and p-value fields
- The pseudotime Wald p-value reproduces the reported significant association (p = 0.0032, Supplementary Fig 17)
- The coxph call includes strata(ER_status) and adjustment covariates (stage, age), not ER as an ordinary covariate
- Pseudotime hazard ratio > 1 (greater angiogenic pseudotemporal progression associated with reduced overall survival)
- Number of complete cases in the model equals the merged pseudotime-survival table rows (no silent NA dropout)

## Limitations

- An assumption made by PhenoPath is that features evolve linearly with respect to pseudotime, so the pseudotime scores fed to the Cox model may miss nonlinear trajectory structure; checks should be conducted that model fits are consistent with the data
- PhenoPath could not handle complex mixtures of continuous and discrete covariates in high-dimensional settings, constraining how ER and other covariates enter the upstream trajectory inference
- Reported p-value (0.0032) should be reproduced exactly; deviation suggests mismatched ER stratification, covariate set, or survival endpoint definitions

## Evidence

- [methods] Workflow quote supporting the coxph design: [evidence span withheld — anchor status: not_found]
- [results] Reported survival finding: [evidence span withheld — anchor status: not_found]
- [discussion] TCGA application context: "We also demonstrated applications to The Cancer Genome Atlas using PhenoPath to model disease trajectories in colorectal and breast cancer."
- [readme] README describing covariate-modulated trajectories: "PhenoPath learns genomic trajectories (pseudotimes) in the presence of heterogenous environmental and genetic backgrounds encoded as additional covariates"
