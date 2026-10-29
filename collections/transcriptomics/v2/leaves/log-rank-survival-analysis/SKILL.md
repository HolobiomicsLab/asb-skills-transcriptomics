---
name: log-rank-survival-analysis
description: Use when validating that predicted molecular subtype labels (e.g., DeepCC
  or PAM50 Luminal/HER2/Basal calls) are clinically meaningful by testing survival
  separation in cohorts with disease-free survival outcomes.
when_to_use_negative:
- Input samples lack clinical outcome data or disease-free survival follow-up, so
  no survival stratification is possible
- The goal is unsupervised grouping of patients — DeepCC and this validation workflow
  are designed for classification, not clustering of cancer patients
edam_operation: http://edamontology.org/operation_0218
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0625
tools:
- name: DeepCC
  role: Produces the predicted Luminal/HER2 subtype labels whose clinical relevance
    is tested by the survival and receptor-status association analyses
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: R
  role: Implementation language for Fisher's exact tests (fisher.test), Kaplan-Meier
    DFS estimation, and log-rank tests (survdiff)
- name: PAM50
  role: Widely used breast cancer intrinsic subtyping system used as the benchmark
    to compare log-rank significance and separation strength against
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:561847f55e268dfce2018ba951249ab949a9dbd4c53b8832359a08ddd177752c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - PAM50
  - R
  edam_operation: http://edamontology.org/operation_0218
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# log-rank-survival-analysis

## Summary

Kaplan-Meier disease-free survival estimation with log-rank testing in R (survdiff) to validate that DeepCC-predicted molecular subtypes of breast cancer separate patient survival at least as strongly as the PAM50 benchmark across TANSBIG, UNT, UPP, and NKI cohorts.

## When to use

Use when validating that predicted molecular subtype labels (e.g., DeepCC or PAM50 Luminal/HER2/Basal calls) are clinically meaningful by testing survival separation in cohorts with disease-free survival outcomes.

## When NOT to use

- Input samples lack clinical outcome data or disease-free survival follow-up, so no survival stratification is possible
- The goal is unsupervised grouping of patients — DeepCC and this validation workflow are designed for classification, not clustering of cancer patients

## Inputs

- DeepCC-predicted subtype labels per sample (from get_DeepCC_label)
- PAM50 subtype labels per sample (benchmark)
- Clinical metadata per cohort (TANSBIG, UNT, UPP, NKI) with ER/PR status, HER2 status, and disease-free survival outcomes

## Outputs

- Kaplan-Meier DFS curves per DeepCC subtype per cohort
- Log-rank test statistics and P values (per cohort and joint across four cohorts)
- Fisher's exact test contingency tables and P values for subtype vs receptor status
- deepcc_validation_stats.csv with contingency tables, P values, and survival statistics

## How to apply

Join DeepCC-predicted subtype labels (from get_DeepCC_label) with per-cohort clinical metadata containing disease-free survival outcomes for TANSBIG, UNT, UPP, and NKI. Estimate Kaplan-Meier DFS curves stratified by DeepCC-defined subtype within each cohort, then run log-rank tests with R's survdiff, first per cohort and again jointly across the four cohorts. Repeat the identical workflow with PAM50 labels as the benchmark so log-rank significance and separation strength can be compared head-to-head. Accept the analysis when DeepCC shows equal or stronger DFS separation than PAM50; optionally complement the survival evidence with Fisher's exact tests of predicted Luminal/HER2 subtypes against ER/PR and HER2 receptor status (association P < 10^-12). Write contingency tables, P values, and survival statistics to a results file such as deepcc_validation_stats.csv.

## Related tools

- **DeepCC** (Produces the predicted Luminal/HER2 subtype labels whose clinical relevance is tested by the survival and receptor-status association analyses) — https://github.com/CityUHK-CompBio/DeepCC
- **R** (Implementation language for Fisher's exact tests (fisher.test), Kaplan-Meier DFS estimation, and log-rank tests (survdiff))
- **PAM50** (Widely used breast cancer intrinsic subtyping system used as the benchmark to compare log-rank significance and separation strength against)

## Examples

```
survdiff(Surv(DFS_time, DFS_event) ~ deepcc_subtype, data = nki_cohort)
```

## Evaluation signals

- All Fisher's exact test P values for predicted Luminal/HER2 subtype vs ER/PR and HER2 receptor status associations are < 10^-12
- DeepCC log-rank DFS significance is equal to or stronger than the PAM50 benchmark in each of TANSBIG, UNT, UPP, and NKI
- Log-rank tests are reported both per cohort and jointly across the four cohorts
- deepcc_validation_stats.csv contains contingency tables, P values, and survival statistics for every cohort
- Kaplan-Meier curves are stratified by the same subtype label set used in the Fisher's exact tests

## Limitations

- DeepCC classification performance depends on a comprehensive gene set prior; missing characteristic signaling pathways or biological processes for a subtype in the MSigDB database can bias predictions and hence downstream survival associations
- BluePrint-style labels predicted at the Luminal/HER2/Basal level cannot distinguish Luminal A from Luminal B, limiting survival comparisons at finer PAM50 granularity
- DeepCC-based subtyping is a supervised classification framework and cannot be used for clustering of cancer patients

## Evidence

- [results] Receptor-status association evidence: [evidence span withheld — anchor status: not_found]
- [discussion] Benchmark comparison evidence: "we effectively applied DeepCC to breast cancer, and demonstrated a better performance over PAM50, which is a widely used classification system in breast cancer"
- [methods] Label prediction step: "pred_labels <- get_DeepCC_label(deepcc_model, new_fs)"
- [discussion] Clustering-only limitation: "specifically developed for addressing current challenges in classification but not clustering of cancer patients"
