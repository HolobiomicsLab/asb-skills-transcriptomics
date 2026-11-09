---
name: kaplan-meier-survival-stratification
description: Use when a per-patient prognostic risk score (e.g., TimiRS from LASSO
  Cox-selected marker pairs) must be validated by showing overall survival separation
  between high- and low-risk groups in one or more cohorts with survival endpoints.
when_to_use_negative:
- Clinical data lack reliable event and time-to-event information (TimiCheckEvent
  filters out low-quality survival info, so stratification would be unpowered)
- The goal is response prediction to immunotherapy rather than overall-survival stratification
  from bulk transcriptomics plus survival data
- Cohorts are too small for adequate log-rank power — TimiGP requires a large gene
  expression dataset with matched survival information to identify prognostic pairs
  effectively
edam_operation: http://edamontology.org/operation_3503
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: TimiGP
  role: R package providing TimiCheckEvent (clinical filtering), TimiPrePropress,
    and TimiGenePair to build the MPS matrix from which TimiRS risk scores used for
    stratification are computed
  repo: https://github.com/CSkylarL/TimiGP
- name: survival
  role: R package used to fit Cox models for marker-pair selection and to perform
    Kaplan-Meier/log-rank survival analysis on risk-stratified groups
- name: R
  role: Execution environment for the TimiGP pipeline, glmnet LASSO Cox fitting, and
    survival analyses
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:0c70c723a32634313f12d6c316ce07a7e9dda23065bca697aec7100d081733c6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - survival
  edam_operation: http://edamontology.org/operation_3503
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Kaplan-Meier survival stratification

## Summary

Stratifies patients into high- and low-risk groups (median split of a risk score such as TimiRS) per cohort and assesses survival separation with Kaplan-Meier curves and log-rank tests, used in TimiGP to validate that inferred cell-cell interactions carry prognostic signal across independent melanoma cohorts.

## When to use

Use when a per-patient prognostic risk score (e.g., TimiRS from LASSO Cox-selected marker pairs) must be validated by showing overall survival separation between high- and low-risk groups in one or more cohorts with survival endpoints.

## When NOT to use

- Clinical data lack reliable event and time-to-event information (TimiCheckEvent filters out low-quality survival info, so stratification would be unpowered)
- The goal is response prediction to immunotherapy rather than overall-survival stratification from bulk transcriptomics plus survival data
- Cohorts are too small for adequate log-rank power — TimiGP requires a large gene expression dataset with matched survival information to identify prognostic pairs effectively

## Inputs

- Per-patient risk scores (e.g., TimiRS) computed for each validation cohort
- Clinical survival information (event and time-to-event) per cohort, filtered via TimiCheckEvent
- Marker Pair Score (MPS) matrices from TimiGenePair for the CD8-Tem marker list
- Independent melanoma cohorts: GSE65904, GSE22155, GSE54467, GSE53118, GSE8401

## Outputs

- Kaplan-Meier overall survival curves for high- vs low-risk groups per cohort
- Log-rank test p-values for survival separation per cohort and per cell-cell interaction
- Harrell's C-index values per cohort (target range 0.621-0.659 for TimiRS)
- Summary table (e.g., timirs_validation_results.csv) of C-index, control-model comparison, and interaction-level stratification results

## How to apply

First compute a per-patient risk score in each validation cohort — in the TimiGP workflow this is the TimiRS score obtained by applying LASSO Cox coefficients (glmnet, alpha = 1) fitted on the training cohort (TCGA SKCM06) to each validation cohort's Marker Pair Score (MPS) matrix built with TimiGenePair. Split patients at the median risk score into high- and low-risk groups per cohort. Plot Kaplan-Meier overall survival curves for the two groups and test separation with the log-rank test using the survival R package; repeat per cohort (GSE65904, GSE22155, GSE54467, GSE53118, GSE8401) and per feature of interest (here, the 19 identified cell-cell interactions). Before analysis, filter clinical data with TimiCheckEvent to remove low-quality event/time-to-event information, since survival-analysis power depends on sample size, event count, and survival-data quality. Assemble C-index values and stratification results (e.g., timirs_validation_results.csv) so performance can be compared against a control model (TimiRS-control).

## Related tools

- **TimiGP** (R package providing TimiCheckEvent (clinical filtering), TimiPrePropress, and TimiGenePair to build the MPS matrix from which TimiRS risk scores used for stratification are computed) — https://github.com/CSkylarL/TimiGP
- **survival** (R package used to fit Cox models for marker-pair selection and to perform Kaplan-Meier/log-rank survival analysis on risk-stratified groups)
- **R** (Execution environment for the TimiGP pipeline, glmnet LASSO Cox fitting, and survival analyses)

## Examples

```
library(TimiGP); library(survival); info <- TimiCheckEvent(info); mps <- TimiGenePair(rna, markers); fit <- glmnet(t(mps[, cox_pairs]), Surv(info$time, info$event), alpha=1); risk <- as.vector(predict(fit, t(mps[, cox_pairs]), s='lambda.min', type='response')); survdiff(Surv(time, event) ~ (risk > median(risk)), data=info)
```

## Evaluation signals

- TimiRS C-index falls within the reported 0.621-0.659 range across all five validation cohorts (GSE65904, GSE22155, GSE54467, GSE53118, GSE8401)
- TimiRS C-index exceeds the TimiRS-control model C-index built from marker pairs not associated with the 19 enriched cell-cell interactions
- High- vs low-risk groups (median risk-score split) show significant survival separation by Kaplan-Meier/log-rank in all five datasets
- Each of the 19 identified cell-cell interactions stratifies overall survival in all five validation datasets
- Clinical inputs pass TimiCheckEvent filtering before survival analysis (no low-quality event/time-to-event records remain)

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs; survival-analysis power is determined by sample size, number of events, and quality of survival information
- No wet-lab experiments were performed to validate the important findings, raising uncertainty about the reliability of the inferred cell-cell interactions
- Because cell-cell interactions are identified via enrichment analysis, sensitivity depends on the number of corresponding marker genes; cell types with fewer markers are less likely to be detected

## Evidence

- [methods] median risk score split with KM/log-rank stratification: [evidence span withheld — anchor status: not_found]
- [methods] C-index validation range: [evidence span withheld — anchor status: not_found]
