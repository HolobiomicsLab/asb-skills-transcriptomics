---
name: kaplan-meier-cohort-survival-estimation
description: Use when DeepCC subtype labels exist for validation cohorts (TANSBIG,
  UNT, UPP, NKI) with clinical metadata containing disease-free survival outcomes,
  and you need to show the classification has prognostic value.
when_to_use_negative:
- You want to discover patient groups by clustering — DeepCC is a supervised classifier,
  not a clustering method ('not clustering of cancer patients').
- The cohorts lack time-to-event or censoring information, so KM estimation and log-rank
  tests are impossible.
- Only receptor-status (ER/PR/HER2) association is needed without survival outcomes
  — the Fisher's exact contingency workflow is the right path instead.
tools:
- name: DeepCC
  role: Supplies the DeepCC-predicted subtype labels (get_DeepCC_label) whose prognostic
    value is evaluated by KM/log-rank analysis
  repo: github.com/CityUHK-CompBio/DeepCC
- name: R
  role: Environment for survival analysis; log-rank tests performed with survdiff
    (survival package) on DFS outcomes
- name: PAM50
  role: Widely used breast intrinsic-subtyping benchmark; its labels are run through
    the identical KM/log-rank analysis to compare significance with DeepCC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:4ddd12b32625b50d8716895e99b8e5a3565b8fb5223956b80e3dd10ed4dc1800
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepCC
  - PAM50
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# kaplan-meier-cohort-survival-estimation

## Summary

Validates DeepCC-predicted breast cancer molecular subtypes (Luminal, HER2, Basal) by estimating Kaplan-Meier disease-free survival (DFS) curves per cohort and testing separation with log-rank tests, benchmarked against PAM50 labels.

## When to use

Use when DeepCC subtype labels exist for validation cohorts (TANSBIG, UNT, UPP, NKI) with clinical metadata containing disease-free survival outcomes, and you need to show the classification has prognostic value.

## When NOT to use

- You want to discover patient groups by clustering — DeepCC is a supervised classifier, not a clustering method ('not clustering of cancer patients').
- The cohorts lack time-to-event or censoring information, so KM estimation and log-rank tests are impossible.
- Only receptor-status (ER/PR/HER2) association is needed without survival outcomes — the Fisher's exact contingency workflow is the right path instead.

## Inputs

- DeepCC-predicted subtype labels per validation cohort (from get_DeepCC_label on functional spectra of cohort expression data)
- Cohort clinical metadata tables with disease-free survival time, DFS event indicator, and cohort identifiers (TANSBIG, UNT, UPP, NKI)
- PAM50 intrinsic subtype labels for the same samples (benchmark)

## Outputs

- Kaplan-Meier DFS curves per DeepCC subtype per cohort
- Log-rank test statistics and P values (per cohort and pooled across cohorts), for both DeepCC and PAM50 labels
- Comparison table of DeepCC vs PAM50 survival separation written to deepcc_validation_stats.csv

## How to apply

Attach the DeepCC-predicted subtype labels (from get_DeepCC_label) to each cohort's clinical table of DFS time/event. In R, estimate Kaplan-Meier DFS curves stratified by DeepCC subtype for each of TANSBIG, UNT, UPP, and NKI separately. Run log-rank tests with survdiff() within each cohort and across the four cohorts jointly. Repeat the identical analysis with PAM50 labels as the benchmark, then compare DeepCC versus PAM50 log-rank significance and separation strength — accept validation when DeepCC shows equal or stronger DFS separation than PAM50 across the four cohorts.

## Related tools

- **DeepCC** (Supplies the DeepCC-predicted subtype labels (get_DeepCC_label) whose prognostic value is evaluated by KM/log-rank analysis) — github.com/CityUHK-CompBio/DeepCC
- **R** (Environment for survival analysis; log-rank tests performed with survdiff (survival package) on DFS outcomes)
- **PAM50** (Widely used breast intrinsic-subtyping benchmark; its labels are run through the identical KM/log-rank analysis to compare significance with DeepCC)

## Examples

```
survdiff(Surv(DFS_time, DFS_event) ~ deepcc_subtype, data = cohort_df)
```

## Evaluation signals

- A KM curve is produced for each DeepCC subtype in each of the four cohorts (TANSBIG, UNT, UPP, NKI), with at-risk groups matching the contingency-table sample counts.
- survdiff log-rank P values are computed both per cohort and across the four cohorts jointly.
- The same analysis is repeated with PAM50 labels, and DeepCC demonstrates equal or stronger DFS separation than PAM50 across the cohorts.
- Statistics (contingency tables, P values, survival statistics) are persisted to deepcc_validation_stats.csv.
- Companion receptor-status Fisher's exact test P values for Luminal/HER2 subtype vs ER/PR and HER2 are all below 10^-12, corroborating the subtype calls used in the survival analysis.

## Limitations

- DeepCC was developed for classification, not clustering of cancer patients; survival validation presumes trustworthy pre-defined subtype labels.
- DeepCC depends on comprehensive gene set prior knowledge — the major characteristic signaling pathways for each subtype should be in the gene set database (MSigDB v7, 22,596 gene sets by default).
- For microarray single samples, the inverseRescale parameter is suggested because TCGA reference data are RNA-Seq; a customized reference expression profile is preferable.
- BluePrint-style labels cannot distinguish Luminal A from Luminal B, so subtype granularity limits survival stratification to Luminal/HER2/Basal when only such predictors are available.

## Evidence

- [discussion] we effectively applied DeepCC to breast cancer, and demonstrated a better performance over PAM50, which is a widely used classification system in breast cancer: "we effectively applied DeepCC to breast cancer, and demonstrated a better performance over PAM50, which is a widely used classification system in breast cancer"
- [abstract] DeepCC provides a novel cancer classification framework that is platform independent, robust to missing data, and can be used for single sample prediction: "DeepCC provides a novel cancer classification framework that is platform independent, robust to missing data, and can be used for single sample prediction"
- [methods] classify new samples with trained DeepCC model: [evidence span withheld — anchor status: not_found]
- [readme] Note: if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data: "Note: if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data"
