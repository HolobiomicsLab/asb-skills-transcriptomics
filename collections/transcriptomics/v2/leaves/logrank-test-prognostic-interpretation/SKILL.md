---
name: logrank-test-prognostic-interpretation
description: Use when integrating patient clinical survival data with continuous molecular
  scores (GSVA, expression TPM) and you need to test whether high vs low score groups
  differ in survival probability via Kaplan-Meier/log-rank.
when_to_use_negative:
- The survival factor is already a categorical variable (e.g. stage or mutation status)
  — ARMT's top/bottom 50% dichotomization applies only to continuous numeric factors
- Clinical survival time/status data are unavailable or cannot be merged with GSVA
  scores on common samples
- The goal is differential expression or mutation analysis rather than prognostic
  survival association
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: ARMT
  role: Integrates clinical, GSVA, TPM and mutation data on common samples and performs
    survival analysis with top/bottom 50% grouping of continuous factors
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Computes the per-sample hypoxia/glycolysis gene set scores that are dichotomized
    for survival testing
- name: survival
  role: R package used to run Kaplan-Meier and Cox proportional hazards analyses
- name: survminer
  role: Visualization of Kaplan-Meier survival curves
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:40957934e5edaa610641c03be9604abfb2423116c17759377a787dcf0dd16dc0
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# logrank-test-prognostic-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Dichotomize continuous GSVA scores (e.g. hypoxia and glycolysis signatures) into top vs bottom 50% groups and run Kaplan-Meier survival analysis with the R 'survival'/'survminer' packages via ARMT, interpreting log-rank p < 0.05 as a significant survival association.

## When to use

Use when integrating patient clinical survival data with continuous molecular scores (GSVA, expression TPM) and you need to test whether high vs low score groups differ in survival probability via Kaplan-Meier/log-rank.

## When NOT to use

- The survival factor is already a categorical variable (e.g. stage or mutation status) — ARMT's top/bottom 50% dichotomization applies only to continuous numeric factors
- Clinical survival time/status data are unavailable or cannot be merged with GSVA scores on common samples
- The goal is differential expression or mutation analysis rather than prognostic survival association

## Inputs

- Clinical data (.csv) with survival time and status columns for TCGA cancer types (e.g. BLCA, CESC, HNSC, LUAD)
- GSVA score matrix (.csv) for the 14-gene hypoxia and 22-gene glycolysis signatures, computed from log2(TPM + 1) data
- TPM expression matrix (.csv, gene symbol rows, sample columns) for optional gene-level integration

## Outputs

- Kaplan-Meier survival curves (high vs low score groups) per cancer type
- Table of log-rank p-values comparing top vs bottom 50% score groups
- Cox proportional hazards regression results for single- and multi-variable models

## How to apply

First integrate clinical data with GSVA scores on common samples using ARMT's Integration&Analysis page (clinical + GSVA merged from .csv files produced by the Data and Normalization&GSVA pages). For each continuous factor, ARMT groups samples into high and low halves (top and bottom 50% of values). Select survival time and status columns from the integrated table, then run Kaplan-Meier analysis with the R 'survival' package (visualized via 'survminer'), producing per-cancer-type curves and a table of log-rank p-values; optionally follow with Cox proportional hazards regression to test independence of factors. Validate the result by checking that the high-score group shows significantly poorer survival (log-rank p < 0.05), as in the paper's hypoxia/glycolysis LUAD finding, and note that Cox regression may reveal only one factor (hypoxia) to be independent.

## Related tools

- **ARMT** (Integrates clinical, GSVA, TPM and mutation data on common samples and performs survival analysis with top/bottom 50% grouping of continuous factors) — https://github.com/Dulab2020/ARMT
- **GSVA** (Computes the per-sample hypoxia/glycolysis gene set scores that are dichotomized for survival testing)
- **survival** (R package used to run Kaplan-Meier and Cox proportional hazards analyses)
- **survminer** (Visualization of Kaplan-Meier survival curves)

## Examples

```
ARMT::run_app()  # use Data -> Normalization&GSVA -> Integration&Analysis > Survival analysis with GSVA score as continuous factor
```

## Evaluation signals

- High glycolysis and high hypoxia groups show significantly poorer survival with log-rank p < 0.05, reproducing the paper's LUAD/pan-cancer survival finding (Fig. 7C context)
- Samples are split exactly at the median: top 50% vs bottom 50% of the continuous GSVA score, per cancer type
- Integrated table contains only common samples across clinical, GSVA and TPM inputs
- Cox regression identifies hypoxia as an independent factor while glycolysis is not independent, consistent with the article's reported result
- Per-cancer-type K-M curves are produced alongside a log-rank p-value table

## Limitations

- Dichotomizing a continuous score into top/bottom 50% discards within-group information; a significant log-rank result does not guarantee independence in a multivariable Cox model (glycolysis was not independent in the article)
- ARMT's TCGA clinical data cover 33 cancer types; survival analysis is limited to cohorts with available clinical follow-up
- No software version numbers for survival/survminer are stated in the article or README

## Evidence
