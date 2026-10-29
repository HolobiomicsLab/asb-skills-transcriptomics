---
name: continuous-variable-median-split-grouping
description: Use when a survival factor (e.g. a continuous GSVA score) must be converted
  into high/low groups to draw K-M curves or compare survival between extreme halves
  of samples.
when_to_use_negative:
- The factor is already categorical or non-numeric — ARMT treats one value as a reference
  instead of splitting
- A biologically meaningful cutoff exists (e.g. known expression threshold); a fixed
  threshold is preferable to a data-driven 50% split
- The goal is multivariate Cox regression, where ARMT uses the numeric factor directly
  without grouping
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0625
tools:
- name: ARMT
  role: Performs the top/bottom 50% grouping of continuous factors and renders K-M
    curves in its shiny GUI
  repo: https://github.com/Dulab2020/ARMT
- name: survival
  role: R package underlying ARMT's Kaplan-Meier survival analysis on the grouped
    samples
- name: GSVA
  role: Produces the continuous per-sample score matrix (e.g. hypoxia/glycolysis)
    that is dichotomized by this skill
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:549b9e0bdc0de26c7899c852021eddc9a05b50ba9f927b63d5008445ab7d1192
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Continuous-variable median-split grouping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Dichotomizes continuous factors such as per-sample GSVA hypoxia/glycolysis scores into high and low groups by splitting at the top and bottom 50% of value ranks, enabling Kaplan-Meier survival comparison in ARMT.

## When to use

Use when a survival factor (e.g. a continuous GSVA score) must be converted into high/low groups to draw K-M curves or compare survival between extreme halves of samples.

## When NOT to use

- The factor is already categorical or non-numeric — ARMT treats one value as a reference instead of splitting
- A biologically meaningful cutoff exists (e.g. known expression threshold); a fixed threshold is preferable to a data-driven 50% split
- The goal is multivariate Cox regression, where ARMT uses the numeric factor directly without grouping

## Inputs

- Integrated data table (.csv) merging TCGA-LUAD clinical data (survival time, status) with continuous per-sample GSVA score matrix
- GSVA scores for the 14-gene hypoxia signature and 22-gene glycolysis signature computed from log2(TPM+1) data

## Outputs

- Kaplan-Meier survival curves comparing high vs low (top/bottom 50%) sample groups per continuous factor
- High/low group assignment for each continuous factor used in downstream survival comparison

## How to apply

In the ARMT Integration&Analysis page, select survival time and status columns plus the continuous factor; ARMT groups samples by value level into top and bottom 50% and computes a K-M curve via the survival package. Apply the same split per factor, e.g. hypoxia (14-gene) and glycolysis (22-gene) GSVA scores in TCGA-LUAD, so each factor gets its own high/low comparison. If the factor is non-numeric, use its existing category values rather than splitting. The rationale is that Cox models treat the score continuously, but K-M visualization requires discrete groups, and the 50/50 split maximizes group balance without an arbitrary biological cutoff.

## Related tools

- **ARMT** (Performs the top/bottom 50% grouping of continuous factors and renders K-M curves in its shiny GUI) — https://github.com/Dulab2020/ARMT
- **survival** (R package underlying ARMT's Kaplan-Meier survival analysis on the grouped samples)
- **GSVA** (Produces the continuous per-sample score matrix (e.g. hypoxia/glycolysis) that is dichotomized by this skill)

## Examples

```
ARMT::run_app()  # then in Integration&Analysis choose survival time/status columns; continuous GSVA factors are auto-split at top/bottom 50%
```

## Evaluation signals

- Each high/low group contains approximately half of the samples (top and bottom 50% of values)
- Grouping is applied per continuous factor independently (separate splits for hypoxia and glycolysis scores)
- Resulting K-M curves reproduce the reported finding: higher hypoxia and glycolysis scores both show significantly poorer survival probability in LUAD patients
- Non-numeric factors are handled by reference-level coding, not by 50% splitting

## Limitations

- The 50% split is a default with no biological basis; different cutoffs can change survival significance
- Only two groups are produced — no tertile/quantile grouping is described in ARMT

## Evidence

- [methods] Grouping of samples by top and bottom 50% for continuous factors in survival analysis: "If the factor is a continuous variable, the samples will be grouped according to the level of values (top and bottom 50%, respectively)"
- [readme] README on survival factor grouping: "If the survival factor  is a continuous variable, the samples will be grouped into two parts(high&low)."
- [methods] LUAD hypoxia/glycolysis survival finding: "both higher hypoxia and glycolysis score were significantly associated with poorer survival probability in LUAD patients"
- [methods] GSVA scores as the continuous factors: "These two gene sets were used to calculated GSVA score by ARMT in the following study"
