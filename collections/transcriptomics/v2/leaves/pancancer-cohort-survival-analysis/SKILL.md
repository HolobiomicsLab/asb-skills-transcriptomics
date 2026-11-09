---
name: pancancer-cohort-survival-analysis
description: Use when a matrix of pathway activity scores (e.g. PROGENy footprints)
  over TCGA tumors must be associated with overall survival time and event status,
  pan-cancer or within individual cancer types.
when_to_use_negative:
- The goal is pathway activity inference from expression data rather than survival
  association — compute scores with PROGENy/decoupleR first, then apply this skill.
- No patient-level clinical survival metadata (OS time and event status) is available,
  so Cox regression cannot be fitted.
- The cohort is a single homogeneous tissue context where tissue-specific signatures,
  not the consensus PROGENy signatures, would better reflect activation.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PROGENy
  role: Provides the pathway activity scores (signaling footprints) for the 11 pathways
    (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF) that
    are regressed against survival
  repo: https://github.com/saezlab/progeny
- name: survival (R coxph)
  role: Fits the covariate-adjusted Cox proportional hazards models per pathway per
    cohort
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:09792d4d81b9d5c71598fa8ed8354de86a07981e9bc14f9139ed6d1d58f278cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - PROGENy
  - survival (R coxph)
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pancancer-cohort-survival-analysis

## Summary

Cox proportional-hazards survival analysis linking PROGENy pathway activity scores to overall patient survival across the pan-cancer TCGA cohort and per cancer type, used to separate oncogenic pathways from tumor-suppressive ones.

## When to use

Use when a matrix of pathway activity scores (e.g. PROGENy footprints) over TCGA tumors must be associated with overall survival time and event status, pan-cancer or within individual cancer types.

## When NOT to use

- The goal is pathway activity inference from expression data rather than survival association — compute scores with PROGENy/decoupleR first, then apply this skill.
- No patient-level clinical survival metadata (OS time and event status) is available, so Cox regression cannot be fitted.
- The cohort is a single homogeneous tissue context where tissue-specific signatures, not the consensus PROGENy signatures, would better reflect activation.

## Inputs

- Matrix of PROGENy pathway activity scores for 7254 TCGA primary tumors
- Clinical survival metadata: overall survival time, event status, cancer type, study of origin, age

## Outputs

- Tidy table (pathway, cohort, HR, FDR) written to survival_associations.csv

## How to apply

Load the matrix of PROGENy pathway activity scores for the 7254 TCGA tumors together with clinical survival metadata (overall survival time and event status, cancer type, study of origin, age). For each of the 11 PROGENy pathways, fit a Cox proportional hazards model with the R survival package (coxph) on the pan-cancer cohort, regressing out study of origin and age as covariates. Repeat the covariate-adjusted Cox regression per pathway within each of the 34 individual cancer types. Apply Benjamini-Hochberg FDR correction across pathways within each cohort and record hazard ratios and FDR values in a tidy table (pathway, cohort, HR, FDR) written to survival_associations.csv. Reproducible analysis scripts live in analyses/tcga_survival of the saezlab/footprints repository (Fig. 5).

## Related tools

- **PROGENy** (Provides the pathway activity scores (signaling footprints) for the 11 pathways (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF) that are regressed against survival) — https://github.com/saezlab/progeny
- **survival (R coxph)** (Fits the covariate-adjusted Cox proportional hazards models per pathway per cohort)

## Examples

```
Rscript -e 'library(survival); fit <- coxph(Surv(OS.time, OS.event) ~ EGFR + study + age, data = tcga_cohort)'
```

## Evaluation signals

- Pan-cancer: significantly increased hazard (decreased survival) for EGFR, MAPK, PI3K, and Hypoxia pathway activation at FDR < 10^-3
- Pan-cancer: significantly decreased hazard (increased survival) for Trail/apoptosis pathway activation at FDR < 10^-3
- Significant p53 association recovered in adrenocortical carcinoma at FDR < 10^-3
- Output table has schema pathway, cohort, HR, FDR with one row per pathway per cohort (pan-cancer plus 34 cancer types)

## Limitations

- Consensus PROGENy signatures estimate pathway activity in large heterogeneous data sets; tissue-specific signatures may more closely reflect activation status in a specific tissue context.
- Survival associations reflect pathway activity inferred from transcriptomes, which do not directly capture post-translational control of signal transduction.

## Evidence

- [abstract] PROGENy distinguishes oncogenic from tumor-suppressive pathways for patient survival: "and (iii) distinguish between oncogenic and tumor suppressor pathways for patient survival"
- [readme] Patient survival analysis (Fig. 5) in analyses/tcga_survival: "Effect on patient survival (Fig. 5)

Analyses available in [analyses/tcga_survival](analyses/tcga_survival)."
- [discussion] Tissue-specific signatures may outperform consensus PROGENy signatures in a single tissue: "While PROGENy provides a good estimate of pathway activity in large and heterogeneous data sets, signatures derived from, for instance, a specific tissue may still more closely reflect activation"
