---
name: fishers-exact-receptor-association-testing
description: Use when you have predicted molecular subtype labels (e.g., from a DeepCC
  model via get_DeepCC_label) plus cohort clinical metadata with ER/PR/HER2 receptor
  status and survival outcomes, and need to demonstrate clinical relevance.
when_to_use_negative:
- Your classification scheme cannot resolve Luminal A vs Luminal B (e.g., BluePrint,
  which 'can only predict Luminal, HER2, and Basal subtypes') — contingency structure
  will differ from the DeepCC/PAM50 comparison
- You are using prognostic risk tools like MammaPrint or Oncotype DX, which predict
  recurrence risk categories rather than biological subtypes, so receptor-association
  contingency tests are not the appropriate validation
- Your goal is unsupervised clustering of patients — DeepCC is 'specifically developed
  for addressing current challenges in classification but not clustering of cancer
  patients'
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3418
tools:
- name: DeepCC
  role: Generates the Luminal/HER2 subtype labels (via get_DeepCC_label on functional
    spectra) whose association with receptor status is tested
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: R
  role: Environment for Fisher's exact tests (fisher.test), Kaplan-Meier estimation,
    and log-rank tests (survdiff); DeepCC runs as an R package
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: PAM50
  role: Widely used breast cancer intrinsic subtyping system used as the benchmark
    label set for the Fisher's exact and DFS log-rank comparisons
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:2223747b9732a30248a2cc6dc9e534395c596399f0a02abd431b330a26f495b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - PAM50
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3418
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# fishers-exact-receptor-association-testing

## Summary

Validates that DeepCC-predicted breast cancer subtypes are biologically meaningful by testing associations between predicted Luminal/HER2 calls and ER/PR/HER2 receptor status via Fisher's exact tests in R, benchmarked against PAM50 with Kaplan-Meier disease-free survival and log-rank tests.

## When to use

Use when you have predicted molecular subtype labels (e.g., from a DeepCC model via get_DeepCC_label) plus cohort clinical metadata with ER/PR/HER2 receptor status and survival outcomes, and need to demonstrate clinical relevance.

## When NOT to use

- Your classification scheme cannot resolve Luminal A vs Luminal B (e.g., BluePrint, which 'can only predict Luminal, HER2, and Basal subtypes') — contingency structure will differ from the DeepCC/PAM50 comparison
- You are using prognostic risk tools like MammaPrint or Oncotype DX, which predict recurrence risk categories rather than biological subtypes, so receptor-association contingency tests are not the appropriate validation
- Your goal is unsupervised clustering of patients — DeepCC is 'specifically developed for addressing current challenges in classification but not clustering of cancer patients'

## Inputs

- DeepCC-predicted subtype labels (output of get_DeepCC_label on the trained breast cancer model)
- Cohort clinical metadata: ER/PR receptor status, HER2 status, and disease-free survival outcomes for TANSBIG, UNT, UPP, and NKI validation cohorts
- PAM50 subtype labels for the same cohorts (benchmark)

## Outputs

- Contingency tables of subtype calls versus ER/PR and HER2 receptor status per cohort
- Fisher's exact test P values for subtype-receptor associations (expected all < 10^-12)
- Kaplan-Meier DFS curves and log-rank (survdiff) test statistics for DeepCC and PAM50 subtypes
- deepcc_validation_stats.csv summarizing P values and survival statistics

## How to apply

Load DeepCC-predicted subtype labels for validation cohorts (TANSBIG, UNT, UPP, NKI) alongside clinical metadata containing ER/PR status, HER2 status, and disease-free survival. Build contingency tables of Luminal/HER2 subtype calls versus receptor status per cohort and run Fisher's exact tests with fisher.test in R; in the DeepCC breast cancer case all association P values fell below 10^-12. In parallel, estimate Kaplan-Meier DFS curves for DeepCC-defined subtypes and run log-rank tests (survdiff) within each cohort and across the four cohorts jointly, repeating the identical analysis with PAM50 labels as the benchmark. Accept the validation when all Fisher P values are < 10^-12 and DeepCC shows equal or stronger DFS separation than PAM50; write contingency tables, P values, and survival statistics to a results file such as deepcc_validation_stats.csv.

## Related tools

- **DeepCC** (Generates the Luminal/HER2 subtype labels (via get_DeepCC_label on functional spectra) whose association with receptor status is tested) — https://github.com/CityUHK-CompBio/DeepCC
- **R** (Environment for Fisher's exact tests (fisher.test), Kaplan-Meier estimation, and log-rank tests (survdiff); DeepCC runs as an R package) — https://github.com/CityUHK-CompBio/DeepCC
- **PAM50** (Widely used breast cancer intrinsic subtyping system used as the benchmark label set for the Fisher's exact and DFS log-rank comparisons)

## Examples

```
fisher.test(table(pred_labels, clinical$HER2_status))  # in R, after pred_labels <- get_DeepCC_label(deepcc_model, new_fs)
```

## Evaluation signals

- All Fisher's exact test P values for DeepCC Luminal/HER2 vs ER/PR and HER2 receptor associations are < 10^-12
- DeepCC-defined subtypes show equal or stronger DFS separation than PAM50 by log-rank test in each of TANSBIG, UNT, UPP, and NKI and across the four cohorts jointly
- deepcc_validation_stats.csv contains non-empty contingency tables, P values, and survival statistics for every validation cohort
- Predicted subtype label counts match the cohort sample sizes and the label levels returned by the DeepCC model (load_DeepCC_model returns classifier plus levels)

## Limitations

- BluePrint, an alternative breast classifier, cannot distinguish between Luminal A and Luminal B subtypes, so receptor-association results are not directly comparable across classification systems
- DeepCC depends on comprehensive gene set prior knowledge (e.g., MSigDB v7 with 22,596 gene sets by default) to avoid potential bias, which affects the subtype labels being validated
- DeepCC is designed for classification, not clustering of cancer patients, so the validation workflow presumes a trained model or pre-existing label set

## Evidence

- [abstract] DeepCC classifiers and DeepCC single sample predictors both achieved overall higher sensitivity, specificity, and accuracy compared with other widely used classification methods: "DeepCC classifiers and DeepCC single sample predictors both achieved overall higher sensitivity, specificity, and accuracy compared with other widely used classification methods"
- [discussion] we effectively applied DeepCC to breast cancer, and demonstrated a better performance over PAM50: "we effectively applied DeepCC to breast cancer, and demonstrated a better performance over PAM50, which is a widely used classification system in breast cancer"
- [methods] get_DeepCC_label(DeepCCModel, newData, cutoff, prob_mode, prob_raw): "get_DeepCC_label(DeepCCModel, newData, cutoff, prob_mode, prob_raw)"
- [readme] pred_labels <- get_DeepCC_label(deepcc_model, new_fs): "new_fs <- getFunctionalSpectra(new_eps)
pred_labels <- get_DeepCC_label(deepcc_model, new_fs)"
