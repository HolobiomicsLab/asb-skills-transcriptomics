---
name: continuous-variable-sample-dichotomization
description: Use when a survival or group-comparison factor is continuous (e.g. per-sample
  GSVA scores) and a two-group (high vs low) split is needed for K-M curves or differential
  analysis in ARMT.
when_to_use_negative:
- The factor is already a categorical/numeric-discrete variable (e.g. mutation status,
  stage) — no dichotomization is needed; select values or groups directly
- The analysis goal is to model the continuous dose-response of the factor — use Cox
  proportional hazards regression with the continuous variable instead of dichotomizing
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0625
tools:
- name: ARMT
  role: Performs the automatic top/bottom 50% grouping of samples by continuous factor
    level within its survival and differential analysis modules
  repo: https://github.com/Dulab2020/ARMT
- name: survival
  role: R package used to run Kaplan-Meier survival analysis and log-rank tests on
    the dichotomized high/low groups
- name: survminer
  role: R package used to visualize the per-cancer-type K-M survival curves of high
    vs low groups
- name: GSVA
  role: Produces the continuous per-sample gene set scores (hypoxia, glycolysis) that
    are dichotomized
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:c385ed77aa798aac6c0d2bddad280b8905d0698def268d1d8dbe9450d4d7936a
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# continuous-variable-sample-dichotomization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Dichotomize samples by a continuous factor (e.g. GSVA scores for hypoxia/glycolysis gene sets) into top-50% (high) and bottom-50% (low) groups so that Kaplan-Meier survival analysis with log-rank testing can be performed, as ARMT does for TCGA cancer cohorts.

## When to use

Use when a survival or group-comparison factor is continuous (e.g. per-sample GSVA scores) and a two-group (high vs low) split is needed for K-M curves or differential analysis in ARMT.

## When NOT to use

- The factor is already a categorical/numeric-discrete variable (e.g. mutation status, stage) — no dichotomization is needed; select values or groups directly
- The analysis goal is to model the continuous dose-response of the factor — use Cox proportional hazards regression with the continuous variable instead of dichotomizing

## Inputs

- Integrated data table (.csv) merging clinical data (survival time/status) with continuous factor columns such as GSVA score matrices
- GSVA score matrix from the Normalization&GSVA module (e.g. 14-gene hypoxia and 22-gene glycolysis signatures)
- TCGA clinical data built into ARMT (33 cancer types)
- TPM/log2(TPM+1) expression matrix with gene symbol IDs

## Outputs

- High vs low sample group assignments (top 50% / bottom 50%) for each continuous factor
- Kaplan-Meier survival curves (via survival/survminer) per factor and per cancer type
- Table of log-rank p-values comparing high vs low groups
- High/low groups for differential analysis with low group as control

## How to apply

First integrate clinical data with the continuous factor (e.g. GSVA scores computed from the 14-gene hypoxia and 22-gene glycolysis signatures on log2(TPM+1)-normalized TCGA counts) on common samples. If the factor is a continuous variable, ARMT groups samples according to their value level, taking the top 50% as the high group and the bottom 50% as the low group. For survival analysis, select the survival time and status columns from the integrated table and treat each continuous factor column as a stratification variable; ARMT produces K-M curves (survival/survminer in R) and log-rank p-values per group. The same dichotomization applies to differential analysis, where the low group serves as the control and the cutoff 't' (upper and lower t*100%) is adjustable. Validate by checking that high hypoxia and high glycolysis groups show significantly poorer survival (log-rank p < 0.05), as replicated in the paper's LUAD analysis.

## Related tools

- **ARMT** (Performs the automatic top/bottom 50% grouping of samples by continuous factor level within its survival and differential analysis modules) — https://github.com/Dulab2020/ARMT
- **survival** (R package used to run Kaplan-Meier survival analysis and log-rank tests on the dichotomized high/low groups)
- **survminer** (R package used to visualize the per-cancer-type K-M survival curves of high vs low groups)
- **GSVA** (Produces the continuous per-sample gene set scores (hypoxia, glycolysis) that are dichotomized)

## Examples

```
ARMT::run_app()  # then in Integration&Analysis select survival time/status columns and a continuous GSVA factor; ARMT auto-splits top/bottom 50%
```

## Evaluation signals

- Each sample in the integrated table is assigned to exactly one of two groups, with group sizes approximately balanced at 50%/50% (within rounding of odd sample counts)
- High glycolysis and high hypoxia groups show significantly poorer survival with log-rank p < 0.05, reproducing the paper's LUAD finding and the pan-cancer Fig. 7C pattern in BLCA, CESC and HNSC
- The K-M curves and log-rank p-value table are produced per continuous factor and per cancer type (via the 'Group Column') without mixing cohorts
- In differential analysis, the low (bottom-50%) group is used as the control group and the adjustable cutoff 't' defaults to the upper/lower t*100% split

## Limitations

- Dichotomization discards continuous-value information; Cox regression on the paper's data showed hypoxia was an independent factor but glycolysis was not, so grouping results may not match multivariable conclusions
- The 50% cutoff is ARMT's default for continuous survival factors; the adjustable cutoff 't' only applies in the differential analysis module, not in K-M stratification
- Results depend on the input cohort composition — the grouping must be computed per cancer type via the 'Group Column' to avoid pan-cancer mixing

## Evidence

- [methods] ARMT groups samples by factor level for survival analysis: "If the factor is a continuous variable, the samples will be grouped according to the level of values (top and bottom 50%, respectively)"
