---
name: breast-cancer-receptor-status-annotation
description: Use when DeepCC-predicted breast cancer subtype labels are available
  for validation cohorts with ER/PR and HER2 receptor-status metadata plus disease-free
  survival outcomes, and you must demonstrate clinical relevance against the PAM50
  benchmark.
when_to_use_negative:
- Input is a breast cancer cohort without receptor-status or survival clinical metadata
  — associations and DFS testing cannot be performed
- The goal is subtype discovery/clustering of patients rather than validating a trained
  classifier's labels — DeepCC is 'specifically developed for addressing current challenges
  in classification but not clustering of cancer patients'
- The cancer type is not breast cancer and no PAM50-style ground truth exists to benchmark
  against
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: DeepCC
  role: Provides the predicted Luminal/HER2/Basal subtype labels and deep features
    that are tested against receptor status and DFS; model invoked via get_DeepCC_label
  repo: https://github.com/zero19970/deepcc_model
- name: R
  role: Analysis environment for Fisher's exact tests (fisher.test), Kaplan-Meier
    DFS estimation, log-rank tests (survdiff), and writing deepcc_validation_stats.csv
- name: PAM50
  role: Widely used breast cancer intrinsic-subtyping system used as the benchmark
    classifier; its labels are re-analyzed identically to compare log-rank significance
    and association strength
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:3a0d22f8af68ab293946397401348a3532ed90079197f3d5db636402bb05b944
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zero19970/deepcc_model
  tools:
  - DeepCC
  - PAM50
  - R
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# breast-cancer-receptor-status-annotation

## Summary

Validates the clinical relevance of DeepCC-predicted breast cancer intrinsic subtypes by testing associations between predicted Luminal/HER2/Basal calls and ER/PR/HER2 receptor status, and by comparing disease-free survival separation of DeepCC versus PAM50 labels across validation cohorts (TANSBIG, UNT, UPP, NKI).

## When to use

Use when DeepCC-predicted breast cancer subtype labels are available for validation cohorts with ER/PR and HER2 receptor-status metadata plus disease-free survival outcomes, and you must demonstrate clinical relevance against the PAM50 benchmark.

## When NOT to use

- Input is a breast cancer cohort without receptor-status or survival clinical metadata — associations and DFS testing cannot be performed
- The goal is subtype discovery/clustering of patients rather than validating a trained classifier's labels — DeepCC is 'specifically developed for addressing current challenges in classification but not clustering of cancer patients'
- The cancer type is not breast cancer and no PAM50-style ground truth exists to benchmark against

## Inputs

- DeepCC-predicted subtype labels for validation cohorts (from get_DeepCC_label)
- Clinical metadata per cohort: ER/PR status, HER2 status, disease-free survival outcomes
- PAM50 subtype labels for the same cohorts (benchmark)
- Cohort expression profiles for TANSBIG, UNT, UPP, and NKI breast cancer validation sets

## Outputs

- Contingency tables of subtype vs ER/PR and HER2 receptor status per cohort
- Fisher's exact test P values for subtype–receptor associations
- Kaplan-Meier DFS curves and log-rank (survdiff) statistics for DeepCC and PAM50 subtypes
- Summary table, e.g. deepcc_validation_stats.csv

## How to apply

Load the DeepCC-predicted subtype labels (obtained via get_DeepCC_label on cohort expression data classified with the trained breast cancer model) and merge them with clinical metadata holding ER/PR status, HER2 status, and disease-free survival. Build contingency tables of Luminal/HER2 subtype calls versus receptor status per cohort and run Fisher's exact tests in R (fisher.test); in the DeepCC study all association P values fell below 10^-12. Estimate Kaplan-Meier disease-free survival curves for DeepCC-defined subtypes in each of TANSBIG, UNT, UPP, and NKI, then perform log-rank tests (survdiff) within each cohort and jointly across the four cohorts. Repeat the same survival analysis with PAM50 labels to benchmark, accepting the result when DeepCC shows equal or stronger DFS separation than PAM50. Write contingency tables, P values, and survival statistics to a summary file such as deepcc_validation_stats.csv.

## Related tools

- **DeepCC** (Provides the predicted Luminal/HER2/Basal subtype labels and deep features that are tested against receptor status and DFS; model invoked via get_DeepCC_label) — https://github.com/zero19970/deepcc_model
- **R** (Analysis environment for Fisher's exact tests (fisher.test), Kaplan-Meier DFS estimation, log-rank tests (survdiff), and writing deepcc_validation_stats.csv)
- **PAM50** (Widely used breast cancer intrinsic-subtyping system used as the benchmark classifier; its labels are re-analyzed identically to compare log-rank significance and association strength)

## Examples

```
fisher.test(table(pred_labels, clin$ER_PR_status)); survdiff(Surv(DFS_time, DFS_event) ~ pred_labels, data=clin)
```

## Evaluation signals

- All Fisher's exact test P values for subtype–receptor-status associations are < 10^-12 (as reported for DeepCC Luminal/HER2 vs ER/PR and HER2 status)
- Log-rank tests on DFS yield significant separation within each of TANSBIG, UNT, UPP, and NKI and across the four cohorts jointly
- DeepCC DFS log-rank significance is equal to or stronger than the same analysis run with PAM50 labels
- Contingency tables contain only the expected subtype categories (e.g., Luminal vs HER2 vs Basal calls) matched one-to-one with samples that have receptor-status metadata
- Output file (deepcc_validation_stats.csv) contains contingency tables, P values, and survival statistics with no missing cohort entries

## Limitations

- DeepCC requires comprehensive gene set prior knowledge to avoid potential bias — 'the prior knowledge of gene sets needs to be comprehensive to avoid potential bias'
- DeepCC supports classification but not clustering of cancer patients, so receptor-status validation presupposes pre-existing subtype labels
- For single-sample microarray input, distribution differences versus TCGA RNA-Seq reference profiles can degrade functional-spectrum computation; a customized reference profile is recommended

## Evidence

- [results] clinical relevance of Luminal/HER2 subtype associations with receptor status: [evidence span withheld — anchor status: not_found]
- [methods] log-rank DFS testing protocol: [evidence span withheld — anchor status: not_found]
- [discussion] classification-only scope: "specifically developed for addressing current challenges in classification but not clustering of cancer patients"
- [discussion] PAM50 benchmark rationale: "we effectively applied DeepCC to breast cancer, and demonstrated a better performance over PAM50, which is a widely used classification system in breast cancer"
- [readme] single-sample reference profile caveat: "Note: You can generate customized reference expression profile from your previous data or public data, which is the same(similar) cancer type and platform."
