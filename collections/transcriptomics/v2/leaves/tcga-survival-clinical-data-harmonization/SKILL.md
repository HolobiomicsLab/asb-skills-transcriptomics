---
name: tcga-survival-clinical-data-harmonization
description: Use when sample-level pseudotimes or trajectory scores must be tested
  against TCGA overall survival outcomes, requiring clinical covariate adjustment
  and stratification (e.g. by ER status) via Cox regression.
when_to_use_negative:
- Input is per-cell single-cell expression data without matched bulk survival/clinical
  records — pseudotime–survival association cannot be modelled
- The covariate (e.g. ER status) violates proportional hazards itself and should be
  modelled as a time-varying effect rather than a strata factor
- The outcome of interest is not time-to-event (e.g. binary response) — logistic regression,
  not Cox modelling, is required
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: R
  role: Implementation environment for the survival analysis pipeline
  repo: https://bioconductor.org/packages/release/bioc/html/phenopath.html
- name: PhenoPath
  role: Source of the ER-stratified pseudotimes tested for survival association
  repo: https://github.com/kieranrcampbell/phenopath
- name: dplyr
  role: Merging pseudotimes with TCGA-BRCA survival and clinical covariate tables
- name: tibble
  role: Structuring the merged analysis data frame and result record
- name: ggplot2
  role: Visualising the survival association (e.g. Supplementary Fig 17)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:e57dc506e6c79242e12b3fd9781dace4f2b52f005c29395e693e30298d181c53
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PhenoPath
  - R
  - dplyr
  - ggplot2
  - tibble
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# tcga-survival-clinical-data-harmonization

## Summary

Merges TCGA-BRCA overall survival times/status and clinical covariates with sample-level pseudotimes (e.g. PhenoPath angiogenesis trajectory scores) and fits an ER-status-stratified Cox proportional hazards model to test whether pseudotemporal progression predicts survival (reported p = 0.0032, Supplementary Fig 17).

## When to use

Use when sample-level pseudotimes or trajectory scores must be tested against TCGA overall survival outcomes, requiring clinical covariate adjustment and stratification (e.g. by ER status) via Cox regression.

## When NOT to use

- Input is per-cell single-cell expression data without matched bulk survival/clinical records — pseudotime–survival association cannot be modelled
- The covariate (e.g. ER status) violates proportional hazards itself and should be modelled as a time-varying effect rather than a strata factor
- The outcome of interest is not time-to-event (e.g. binary response) — logistic regression, not Cox modelling, is required

## Inputs

- ER-stratified PhenoPath pseudotime scores per TCGA-BRCA tumour sample (from task_006)
- TCGA-BRCA overall survival time and event status per sample
- TCGA-BRCA clinical covariates (e.g. stage, age, ER status)

## Outputs

- tcga_brca_cox_results.csv containing pseudotime coefficient, hazard ratio, confidence interval, and Wald p-value
- Fitted coxph model summary object

## How to apply

Load the ER-stratified PhenoPath pseudotimes from the upstream trajectory-inference task and merge them with TCGA-BRCA overall survival time/status and clinical covariates (stage, age) using dplyr and tibble. Fit a Cox proportional hazards model with survival::coxph, using pseudotime as the primary predictor, survival time/status as the response, clinical covariates as adjustments, and ER status as a stratification factor (strata argument) rather than a covariate. Extract the pseudotime coefficient, hazard ratio, confidence interval, and Wald p-value from the model summary into a result data frame and write it to tcga_brca_cox_results.csv. The stratification choice follows the paper's design of encoding ER status as a heterogeneous background factor so each ER subgroup develops its own trajectory-baseline hazard relationship.

## Related tools

- **R** (Implementation environment for the survival analysis pipeline) — https://bioconductor.org/packages/release/bioc/html/phenopath.html
- **PhenoPath** (Source of the ER-stratified pseudotimes tested for survival association) — https://github.com/kieranrcampbell/phenopath
- **dplyr** (Merging pseudotimes with TCGA-BRCA survival and clinical covariate tables)
- **tibble** (Structuring the merged analysis data frame and result record)
- **ggplot2** (Visualising the survival association (e.g. Supplementary Fig 17))

## Examples

```
fit <- coxph(Surv(OS_time, OS_status) ~ pseudotime + stage + age + strata(ER_status), data = tcga_merged); summary(fit)
```

## Evaluation signals

- Output file tcga_brca_cox_results.csv exists and contains coefficient, hazard-ratio, and p-value fields
- The pseudotime Wald p-value reproduces the reported significant association (p = 0.0032, Supplementary Fig 17)
- Hazard ratio for pseudotime exceeds 1, consistent with greater angiogenic progression conferring reduced overall survival
- ER status is encoded as a strata (stratification) factor, not as an adjustment covariate, in the coxph call
- Merged analysis table row count matches the number of samples with both pseudotime and survival data (no silent NA drops)

## Limitations

- PhenoPath assumes features evolve linearly with pseudotime; nonlinear effects may occur and model fits should be verified against the data before survival testing
- PhenoPath cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings, which constrains which covariates can enter the trajectory model upstream
- Pseudotimes are inferred from bulk TCGA expression and are relative, not absolute time, so hazard ratios describe relative progression only

## Evidence

- [discussion] We also demonstrated applications to The Cancer Genome Atlas using PhenoPath to model disease trajectories in colorectal and breast cancer.: "We also demonstrated applications to The Cancer Genome Atlas using PhenoPath to model disease trajectories in colorectal and breast cancer."
- [abstract] We describe a novel statistical framework that learns how pseudotime trajectories can be modulated through covariates that encode such factors.: "We describe a novel statistical framework that learns how pseudotime trajectories can be modulated through covariates that encode such factors."
- [readme] Inference is performed using co-ordinate ascent variational inference (CAVI) implemented using the Rcpp library for increased performance.: [evidence span withheld — anchor status: external_doc]
- [discussion] checks should be conducted to verify that PhenoPath model fits are consistent with the data: "checks should be conducted to verify that PhenoPath model fits are consistent with the data"
