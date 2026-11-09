---
name: cox-proportional-hazards-regression
description: Use when you have per-sample survival time/status merged with continuous
  molecular scores (e.g. GSVA) or clinical covariates and need to determine which
  factors independently predict survival.
when_to_use_negative:
- When only one variable is of interest and no adjustment for confounders is needed
  — Kaplan-Meier curves with top/bottom 50% splitting may suffice
- When the input is raw counts or a TPM matrix without per-sample survival time/status
  — no Cox model can be fit until clinical data are merged via common samples
- When survival times are heavily tied or non-proportional hazards are evident — the
  proportional hazards assumption underlying the Cox model would be violated
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: ARMT
  role: Provides the shiny GUI (Integration&Analysis page) to integrate clinical,
    GSVA and TPM data and run single- or multi-variable Cox proportional hazards regression,
    rendering results for forest plotting
  repo: https://github.com/Dulab2020/ARMT
- name: survival
  role: R package used for Cox proportional hazards regression and Kaplan-Meier survival
    analysis underlying the ARMT survival/Cox functions
- name: GSVA
  role: Produces the per-sample GSVA scores (e.g. 14-gene hypoxia and 22-gene glycolysis
    signatures) that serve as continuous covariates in the Cox model
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:922a99e676484d84cef3b0be33a9c9807ee78bb144e5b6a8b0ca189d9da1e46f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/ARMT
  repo_url: https://github.com/Dulab2020/ARMT
  tools:
  - ARMT
  - GSVA
  - survival
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cox-proportional-hazards-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits univariate and multivariate Cox proportional hazards models to assess whether continuous factors (e.g. GSVA hypoxia and glycolysis scores) and clinical covariates (gender, stage, age) are independent prognostic factors, as implemented in the ARMT workflow using the R 'survival' package.

## When to use

Use when you have per-sample survival time/status merged with continuous molecular scores (e.g. GSVA) or clinical covariates and need to determine which factors independently predict survival.

## When NOT to use

- When only one variable is of interest and no adjustment for confounders is needed — Kaplan-Meier curves with top/bottom 50% splitting may suffice
- When the input is raw counts or a TPM matrix without per-sample survival time/status — no Cox model can be fit until clinical data are merged via common samples
- When survival times are heavily tied or non-proportional hazards are evident — the proportional hazards assumption underlying the Cox model would be violated

## Inputs

- Integrated data table (.csv) merging clinical data (survival time, status, gender, stage, age) with per-sample GSVA score matrix via common TCGA sample IDs
- Survival time column and status column identifiers
- Covariate columns (e.g. hypoxia GSVA score, glycolysis GSVA score, gender, stage, age)

## Outputs

- Univariate and multivariate Cox regression coefficient tables with p-values identifying independent prognostic factors
- Forest plots of Cox model coefficients
- Comparison against Kaplan-Meier high/low survival curves for each continuous factor

## How to apply

First integrate the clinical data (survival time, status, gender, stage, age) with the per-sample GSVA score matrix through common sample IDs. Select the survival time and status columns from the integrated data table, then choose the factors to test — in ARMT every column of the integrated table can serve as a factor. Run univariate Cox models for each factor (e.g. hypoxia score, glycolysis score), then a multivariate model combining all covariates to identify independent prognostic factors; for non-numeric factors, one value must be designated as the reference level. Render hazard ratios and coefficients as forest plots and compare against Kaplan-Meier results from top/bottom 50% group splits of continuous factors.

## Related tools

- **ARMT** (Provides the shiny GUI (Integration&Analysis page) to integrate clinical, GSVA and TPM data and run single- or multi-variable Cox proportional hazards regression, rendering results for forest plotting) — https://github.com/Dulab2020/ARMT
- **survival** (R package used for Cox proportional hazards regression and Kaplan-Meier survival analysis underlying the ARMT survival/Cox functions)
- **GSVA** (Produces the per-sample GSVA scores (e.g. 14-gene hypoxia and 22-gene glycolysis signatures) that serve as continuous covariates in the Cox model)

## Examples

```
ARMT::run_app()  # then in the GUI: Integration&Analysis -> choose survival time/status columns -> Cox regression with hypoxia score, glycolysis score, gender, stage, age
```

## Evaluation signals

- In the LUAD reference result, hypoxia GSVA score must emerge as an independent factor in the multivariate Cox model while glycolysis does not remain independent
- Univariate associations should be consistent with the KM curves: both higher hypoxia and glycolysis scores correspond to significantly poorer survival probability
- Non-numeric covariates (e.g. gender, stage) must have one level explicitly set as the reference before fitting
- The integrated table must contain only common samples shared across clinical, GSVA and TPM data — sample-ID mismatch should fail or reduce the cohort visibly
- Forest plots of coefficients should render with the covariates entered in the model formula (glycolysis score, hypoxia score, gender, stage, age)

## Limitations

- If a Cox factor is not numeric, ARMT requires the user to manually designate a reference value — no automatic reference selection
- ARMT groups continuous factors by top/bottom 50% only for KM curves; no built-in alternative cutoff optimization (e.g. maxstat) is described
- No software or ARMT version numbers are stated, so exact package versions affecting results cannot be pinned

## Evidence
