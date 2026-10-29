---
name: kaplan-meier-survival-analysis
description: Use when TCGA clinical data (survival time, status) has been merged with
  a continuous sample-level factor such as GSVA scores, and you need to test whether
  high vs low factor levels associate with poorer survival probability.
when_to_use_negative:
- The survival factor is already a categorical variable with its own group labels
  — no top/bottom 50% dichotomization is needed or appropriate
- The goal is to assess the combined/independent effect of multiple covariates rather
  than univariate survival curves — use Cox proportional hazards regression instead
- Input lacks merged clinical survival data (time and status); K-M analysis cannot
  run on GSVA scores alone
edam_operation: http://edamontology.org/operation_3503
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0625
tools:
- name: ARMT
  role: Provides TCGA clinical data, integrates it with GSVA scores on common samples,
    dichotomizes continuous factors into top/bottom 50% groups, and drives K-M analysis
    through its shiny GUI
  repo: https://github.com/Dulab2020/ARMT
- name: survival
  role: R package underlying ARMT's survival analysis that computes Kaplan-Meier curves
    and log-rank tests
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Produces the per-sample gene-set (hypoxia/glycolysis) scores used as the continuous
    survival factor
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:99820164ef27419fe9cc0815f8bdb59f06b9e5c9623bb9c62203d69c0cd5a2a4
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
  - R
  - survival
  - survminer
  edam_operation: http://edamontology.org/operation_3503
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# kaplan-meier-survival-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Kaplan-Meier survival analysis of patient cohorts dichotomized by a continuous factor (e.g., GSVA hypoxia/glycolysis scores), implemented in ARMT via the R 'survival' package and visualized as K-M curves with log-rank testing.

## When to use

Use when TCGA clinical data (survival time, status) has been merged with a continuous sample-level factor such as GSVA scores, and you need to test whether high vs low factor levels associate with poorer survival probability.

## When NOT to use

- The survival factor is already a categorical variable with its own group labels — no top/bottom 50% dichotomization is needed or appropriate
- The goal is to assess the combined/independent effect of multiple covariates rather than univariate survival curves — use Cox proportional hazards regression instead
- Input lacks merged clinical survival data (time and status); K-M analysis cannot run on GSVA scores alone

## Inputs

- Integrated data table (.csv) containing clinical survival time and status columns plus continuous factors (GSVA score, TPM expression)
- GSVA score matrix produced by ARMT's Normalization&GSVA page
- Built-in TCGA clinical data for 33 cancer types obtained via ARMT's Data page

## Outputs

- Kaplan-Meier survival curves (one per selected factor, comparing high vs low groups)
- Log-rank p-values for each dichotomized factor

## How to apply

First integrate clinical data with GSVA scores (e.g., 14-gene hypoxia or 22-gene glycolysis signatures) on common sample IDs using ARMT's Integration&Analysis page, selecting columns for survival time and status. Because GSVA scores are continuous variables, ARMT automatically groups samples into high and low parts by value level (default top and bottom 50%). Run Kaplan-Meier analysis with the R 'survival' package; multiple factors can be selected to obtain multiple K-M curves in one run. Judge success by log-rank significance (p < 0.05) between high and low groups, e.g., in TCGA-LUAD both higher hypoxia and glycolysis scores showed significantly poorer survival probability.

## Related tools

- **ARMT** (Provides TCGA clinical data, integrates it with GSVA scores on common samples, dichotomizes continuous factors into top/bottom 50% groups, and drives K-M analysis through its shiny GUI) — https://github.com/Dulab2020/ARMT
- **survival** (R package underlying ARMT's survival analysis that computes Kaplan-Meier curves and log-rank tests) — https://github.com/Dulab2020/ARMT
- **GSVA** (Produces the per-sample gene-set (hypoxia/glycolysis) scores used as the continuous survival factor)

## Examples

```
ARMT::run_app()  # then in Integration&Analysis: select survival time/status columns and GSVA score factors to render K-M curves
```

## Evaluation signals

- Continuous factors are split exactly into top and bottom 50% high/low groups, matching ARMT's default grouping rule
- K-M curve for the high group falls below the low group (poorer survival) when the factor is adverse, e.g., hypoxia/glycolysis score in LUAD
- Log-rank p-value < 0.05 for both high hypoxia and high glycolysis groups in TCGA-LUAD, reproducing the paper's finding
- Curves are generated for every factor selected, and sample counts per group are consistent with the merged clinical+GSVA dataset
- The analysis used the same common-sample integration of clinical and GSVA data, so every curve is based on samples present in both sources

## Limitations

- Only a top/bottom 50% cut is described for continuous factors; alternative cutoffs (e.g., optimal maxstat) are not supported in the documented workflow
- K-M analysis is univariate — it cannot establish independence of a factor, as shown when hypoxia was independent in Cox regression but glycolysis was not despite both being significant in K-M
- The README truncates before describing the GSVA score matrix input format in full, so exact file-format details for this input are incomplete

## Evidence
