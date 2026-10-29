---
name: clinical-covariate-adjustment-regression
description: Use when associating continuous molecular scores (e.g., PROGENy pathway
  activity) with patient overall survival time and event status, and confounders such
  as cohort/study of origin or age must be adjusted for.
when_to_use_negative:
- The goal is inferring pathway activity scores from expression rather than testing
  survival associations — use PROGENy/decoupleR scoring instead; this skill consumes
  already-computed scores.
- Survival data are missing or the endpoint is not a time-to-event with censoring
  (e.g., only binary response), where Cox regression is inappropriate.
- Analysis targets non-clinical associations such as driver-mutation recovery or drug
  response in GDSC cell lines, where this Cox-survival workflow does not apply.
edam_operation: http://edamontology.org/operation_3657
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0625
tools:
- name: PROGENy
  role: Provides the pathway-responsive gene signatures from which the pathway activity
    scores fed into the Cox regression are computed
  repo: https://github.com/saezlab/progeny
- name: R survival package (coxph)
  role: Fits the covariate-adjusted Cox proportional hazards models of pathway score
    vs. overall survival
- name: decoupler
  role: Recommended method for inferring pathway activity scores from transcriptomics
    using PROGENy signatures
  repo: https://github.com/scverse/decoupler
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:201e82fc1b6b68817049a290d2e88ae3c4d91b07ac13782cfe7089d74315026a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - PROGENy
  - R survival package (coxph)
  - decoupler
  edam_operation: http://edamontology.org/operation_3657
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# clinical-covariate-adjustment-regression

## Summary

Fit covariate-adjusted Cox proportional hazards models (R survival package, coxph) linking pathway activity scores such as PROGENy signaling footprints to patient overall survival, regressing out confounders like study of origin and age in pan-cancer and per-cancer-type cohorts.

## When to use

Use when associating continuous molecular scores (e.g., PROGENy pathway activity) with patient overall survival time and event status, and confounders such as cohort/study of origin or age must be adjusted for.

## When NOT to use

- The goal is inferring pathway activity scores from expression rather than testing survival associations — use PROGENy/decoupleR scoring instead; this skill consumes already-computed scores.
- Survival data are missing or the endpoint is not a time-to-event with censoring (e.g., only binary response), where Cox regression is inappropriate.
- Analysis targets non-clinical associations such as driver-mutation recovery or drug response in GDSC cell lines, where this Cox-survival workflow does not apply.

## Inputs

- Matrix of PROGENy pathway activity scores per tumor sample (e.g., 7254 TCGA tumors x 11 pathways)
- Clinical survival metadata: overall survival time and event status per patient
- Per-patient covariates: cancer type, study of origin, age

## Outputs

- Tidy CSV table of survival associations with columns (pathway, cohort, HR, FDR), e.g. survival_associations.csv
- Benjamini-Hochberg FDR-adjusted p-values per pathway within each cohort

## How to apply

Load a matrix of PROGENy pathway activity scores (e.g., for the 7254 TCGA tumors) together with clinical survival metadata: overall survival time, event status, cancer type, study of origin, and age. For each pathway, fit a Cox proportional hazards model with the R survival package's coxph on the pan-cancer cohort, including study of origin and age as covariates to regress them out. Repeat the covariate-adjusted Cox regression per pathway within each individual cancer type (34 cohorts in the TCGA analysis). Apply Benjamini-Hochberg FDR correction across pathways within each cohort, and collect hazard ratios and FDR values into a tidy table (pathway, cohort, HR, FDR). Interpret HR > 1 as the pathway activation associating with decreased survival (oncogenic behavior) and HR < 1 as increased survival (tumor-suppressive behavior).

## Related tools

- **PROGENy** (Provides the pathway-responsive gene signatures from which the pathway activity scores fed into the Cox regression are computed) — https://github.com/saezlab/progeny
- **R survival package (coxph)** (Fits the covariate-adjusted Cox proportional hazards models of pathway score vs. overall survival)
- **decoupler** (Recommended method for inferring pathway activity scores from transcriptomics using PROGENy signatures) — https://github.com/scverse/decoupler

## Examples

```
library(survival); fit <- coxph(Surv(OS_time, OS_event) ~ EGFR_score + MAPK_score + study_origin + age, data=tcga_pancontrols)
```

## Evaluation signals

- Pan-cancer models show significantly increased hazard (decreased survival) for EGFR, MAPK, PI3K, and Hypoxia pathway activation
- Trail/apoptosis pathway activation shows significantly decreased hazard (increased survival) pan-cancer; PROGENy is the only method to find this
- p53 shows a significant association in adrenocortical carcinoma
- All key reported associations pass FDR < 10^-3 after Benjamini-Hochberg correction within each cohort
- Output table has the tidy schema (pathway, cohort, HR, FDR) covering the pan-cancer cohort plus 34 individual cancer types

## Limitations

- Individual perturbation-expression signatures are heterogeneous; the analysis relies on PROGENy consensus signatures rather than tissue-specific ones, which may more closely reflect activation in a specific context
- Pathway scores from gene expression do not account for post-translational modifications, so they reflect inferred signaling activity rather than direct measurement
- Consensus PROGENy signatures may not capture additional transcriptional modules that are not always activated; more studies are needed to fully elucidate differences between the common response and such modules

## Evidence

- [abstract] PROGENy can distinguish between oncogenic and tumor suppressor pathways for patient survival: "and (iii) distinguish between oncogenic and tumor suppressor pathways for patient survival"
- [other] Cox model covariates: study of origin and age are regressed out per pathway: [evidence span withheld — anchor status: not_found]
- [readme] Analysis scripts for the survival effect (Fig. 5) live in the repo: "Effect on patient survival (Fig. 5)

Analyses available in [analyses/tcga_survival](analyses/tcga_survival)."
- [readme] TCGA pathway scores computed on primary tumors with tissue-matched normals from firebrowse: "We took TCGA data from [firehose.org](http://firebrowse.org/) and computed all
pathway scores for primary tumors (`01A` in barcode)"
- [readme] PROGENy leverages perturbation compendium to define consensus pathway activity signatures: "We overcome both limitations by
leveraging a large compendium of publicly available perturbation experiments to
define consensus signatures for pathway activity."
