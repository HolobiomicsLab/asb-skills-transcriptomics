---
name: forest-plot-visualization
description: Use when Cox regression results (single- or multi-variable, e.g., GSVA
  scores with clinical covariates) need to be visualized as a forest plot to judge
  which factors are independent prognostic factors.
when_to_use_negative:
- The goal is Kaplan-Meier survival curves for a single factor rather than multi-factor
  Cox regression effects
- The factor list contains only non-numeric variables without a defined reference
  level, making hazard-ratio interpretation ambiguous
- The data have not been integrated (clinical, GSVA, TPM merged by common samples),
  so covariates cannot be jointly modeled
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: ARMT
  role: Runs Cox proportional-hazards regression on the integrated clinical/GSVA table
    and renders coefficients as forest plots via its shiny GUI
  repo: https://github.com/Dulab2020/ARMT
- name: survival
  role: R package performing the underlying Kaplan-Meier and Cox proportional-hazards
    regression computations feeding the forest plot
- name: GSVA
  role: Produces the per-sample hypoxia/glycolysis scores used as continuous covariates
    in the Cox model
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:a1879572c699a36efcf476d55c5738d97a9e7279ed81a1dc0af773e26a7a371c
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# forest-plot-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Render Cox proportional-hazards regression coefficients (e.g., GSVA hypoxia and glycolysis scores, gender, stage, age in TCGA-LUAD) as forest plots so that hazard ratios, confidence intervals and independence of prognostic factors can be inspected at a glance.

## When to use

Use when Cox regression results (single- or multi-variable, e.g., GSVA scores with clinical covariates) need to be visualized as a forest plot to judge which factors are independent prognostic factors.

## When NOT to use

- The goal is Kaplan-Meier survival curves for a single factor rather than multi-factor Cox regression effects
- The factor list contains only non-numeric variables without a defined reference level, making hazard-ratio interpretation ambiguous
- The data have not been integrated (clinical, GSVA, TPM merged by common samples), so covariates cannot be jointly modeled

## Inputs

- Integrated data table (.csv) merging clinical data (survival time, status, gender, stage, age), GSVA score matrix, TPM expression and mutation data by common sample IDs
- Selection of one column as survival time and one as survival status
- Cox model factor list (numeric and categorical variables, e.g., hypoxia score, glycolysis score, gender, stage, age)

## Outputs

- Forest plot of Cox regression coefficients (hazard ratios with confidence intervals) for each covariate
- Identification of independent prognostic factors from the multivariate model

## How to apply

Fit a Cox proportional-hazards model with the R 'survival' package (integrated into ARMT) using the merged clinical + GSVA score table, selecting survival time and status columns from the integrated data. Include both continuous factors (e.g., hypoxia and glycolysis GSVA scores, age) and categorical factors (gender, stage); for any non-numeric factor one value must be set as the reference level. Run univariate and multivariate models — univariate to screen associations, multivariate to test independence of each covariate. Render the resulting coefficients (hazard ratios with confidence intervals) as a forest plot; ARMT's GUI supports selecting multiple factors and returns the forest plot in its main panel. In the LUAD example, the forest plot showed hypoxia remained an independent factor while glycolysis did not.

## Related tools

- **ARMT** (Runs Cox proportional-hazards regression on the integrated clinical/GSVA table and renders coefficients as forest plots via its shiny GUI) — https://github.com/Dulab2020/ARMT
- **survival** (R package performing the underlying Kaplan-Meier and Cox proportional-hazards regression computations feeding the forest plot)
- **GSVA** (Produces the per-sample hypoxia/glycolysis scores used as continuous covariates in the Cox model)

## Examples

```
devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()  # then in Integration&Analysis page select survival time/status columns and covariates for Cox forest plot
```

## Evaluation signals

- The forest plot contains one row per selected covariate with an estimated hazard ratio and confidence interval
- For non-numeric covariates (e.g., stage, gender), a reference level is defined and relative coefficients are interpretable
- In the TCGA-LUAD validation, the multivariate forest plot shows hypoxia score as an independent factor while glycolysis is not independent
- Univariate forest plots reproduce the significant association of both higher hypoxia and glycolysis scores with poorer survival
- All covariates come from the same integrated data table merged by common sample IDs

## Limitations

- If a factor in the Cox analysis is not a numeric variable, one value must be designated as the reference, which affects coefficient interpretation
- ARMT does not state version numbers or detail handling of missing clinical covariates in the Cox analysis
- Dichotomizing continuous GSVA factors (top/bottom 50%) applies to KM curves; Cox regression uses the continuous scores, and choices here affect the reported independence of factors

## Evidence
