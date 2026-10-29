---
name: subtype-classifier-comparative-evaluation
description: Use when a trained subtype classifier (e.g., DeepCC labels from get_DeepCC_label)
  has been applied to validation cohorts with clinical metadata and you must show
  its clinical relevance equals or exceeds a benchmark system like PAM50.
when_to_use_negative:
- The goal is de novo discovery of subtypes — DeepCC is a supervised classification
  framework 'not clustering of cancer patients', so a classifier-evaluation skill
  on predefined subtypes is inappropriate for unsupervised discovery
- The input is raw expression data without predicted subtype labels or clinical receptor/DFS
  metadata — receptor association and survival testing cannot be performed and the
  classifier must first be run (getFunctionalSpectra then get_DeepCC_label)
- There is no established benchmark (e.g., PAM50) available for the same samples,
  making a comparative evaluation impossible
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DeepCC
  role: Supervised deep-learning classifier producing the subtype labels under evaluation;
    pretrained breast cancer model trained on TCGA data available in the deepcc_model
    repository
  repo: https://github.com/zero19970/deepcc_model
- name: PAM50
  role: Established breast cancer intrinsic subtyping system used as the benchmark
    for receptor association and DFS log-rank comparison
- name: R
  role: 'Analysis environment: fisher.test for subtype–receptor contingency tables
    and survdiff/Kaplan-Meier for DFS log-rank tests; DeepCC itself is an R/keras
    package installed via devtools from GitHub'
  repo: https://github.com/CityUHK-CompBio/DeepCC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:10ef421d0ec3a82b08ee7eaf9373a169c99d3ab6df5864e91409fd91078701d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zero19970/deepcc_model
  tools:
  - DeepCC
  - PAM50
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# subtype-classifier-comparative-evaluation

## Summary

Benchmark a newly predicted molecular subtype classification (e.g., DeepCC-derived labels) against an established standard (PAM50) by testing concordance with known receptor biology (ER/PR/HER2 via Fisher's exact tests) and clinical outcome separation (Kaplan-Meier DFS with log-rank tests) across independent validation cohorts.

## When to use

Use when a trained subtype classifier (e.g., DeepCC labels from get_DeepCC_label) has been applied to validation cohorts with clinical metadata and you must show its clinical relevance equals or exceeds a benchmark system like PAM50.

## When NOT to use

- The goal is de novo discovery of subtypes — DeepCC is a supervised classification framework 'not clustering of cancer patients', so a classifier-evaluation skill on predefined subtypes is inappropriate for unsupervised discovery
- The input is raw expression data without predicted subtype labels or clinical receptor/DFS metadata — receptor association and survival testing cannot be performed and the classifier must first be run (getFunctionalSpectra then get_DeepCC_label)
- There is no established benchmark (e.g., PAM50) available for the same samples, making a comparative evaluation impossible

## Inputs

- DeepCC-predicted subtype labels for validation cohorts (from get_DeepCC_label on new functional spectra)
- Clinical metadata per cohort: ER/PR status, HER2 status, disease-free survival outcomes
- PAM50 benchmark labels for the same samples
- Cohorts: TANSBIG, UNT, UPP, NKI breast cancer expression sets

## Outputs

- Per-cohort Fisher's exact test contingency tables and P values for subtype vs ER/PR and HER2 receptor status
- Kaplan-Meier DFS curves with log-rank test statistics for DeepCC and PAM50 labels
- deepcc_validation_stats.csv containing contingency tables, P values, and survival statistics

## How to apply

Load the DeepCC-predicted subtype labels for each validation cohort (TANSBIG, UNT, UPP, NKI) together with ER/PR status, HER2 status, and disease-free survival outcomes. Build contingency tables of predicted Luminal/HER2 calls versus ER/PR and HER2 receptor status per cohort and run Fisher's exact tests (fisher.test in R), requiring all association P values below 10^-12 as evidence the predicted subtypes track their characteristic markers. Estimate Kaplan-Meier DFS curves per cohort and run log-rank tests (survdiff) within each cohort and jointly across the four cohorts, then repeat the identical analysis with PAM50 labels as the benchmark. Accept the evaluation only if the new classifier shows receptor associations below the P < 10^-12 threshold and equal or stronger DFS separation than PAM50; write contingency tables, P values, and survival statistics to deepcc_validation_stats.csv.

## Related tools

- **DeepCC** (Supervised deep-learning classifier producing the subtype labels under evaluation; pretrained breast cancer model trained on TCGA data available in the deepcc_model repository) — https://github.com/zero19970/deepcc_model
- **PAM50** (Established breast cancer intrinsic subtyping system used as the benchmark for receptor association and DFS log-rank comparison)
- **R** (Analysis environment: fisher.test for subtype–receptor contingency tables and survdiff/Kaplan-Meier for DFS log-rank tests; DeepCC itself is an R/keras package installed via devtools from GitHub) — https://github.com/CityUHK-CompBio/DeepCC

## Examples

```
library(DeepCC); fs <- getFunctionalSpectra(new_eps); pred_labels <- get_DeepCC_label(deepcc_model, new_fs); fisher.test(table(pred_labels, receptor_status))
```

## Evaluation signals

- All Fisher's exact test P values for DeepCC Luminal/HER2 vs ER/PR and HER2 receptor status are < 10^-12
- Log-rank DFS separation by DeepCC labels is equal to or stronger than PAM50 log-rank significance across TANSBIG, UNT, UPP, and NKI cohorts (individually and jointly)
- deepcc_validation_stats.csv is populated with contingency tables, P values, and survival statistics for both classifiers
- Predicted label levels match the trained classifier's stored levels (the model object returns list(classifier, levels)) — labels map onto Luminal/HER2/Basal categories cleanly
- Single-sample predictions use a reference expression profile (e.g., refExp = 'BRCA' for TCGA reference) and, for microarray input, the inverseRescale flag is applied so labels are not confounded by platform distribution differences

## Limitations

- DeepCC is developed for classification, not clustering of cancer patients — evaluation assumes labels from a trained supervised model, not de novo subtypes
- Breast DeepCC evaluation cannot distinguish Luminal A from Luminal B in the way expected of full intrinsic subtyping; comparisons against PAM50 should account for the predicted subtype granularity
- For single-sample microarray input, the TCGA-derived reference is RNA-Seq; the README warns distribution differences mean 'it's better to generate your customized reference', and a mismatched reference degrades label quality feeding the evaluation
- DeepCC's prior knowledge depends on comprehensive gene sets (MSigDB v7, 22,596 sets by default); biased gene sets can bias the labels being evaluated

## Evidence

- [discussion] DeepCC better than PAM50 benchmark in breast cancer: "we effectively applied DeepCC to breast cancer, and demonstrated a better performance over PAM50, which is a widely used classification system in breast cancer"
- [discussion] Trained model is used to classify new samples (source of labels under evaluation): "The trained DeepCC model can be subsequently used for classification of new samples"
- [readme] Single-sample label prediction requires a reference expression profile: "for a given single sample, you have to provide a reference expression profile. new_fs_single <- getFunctionalSpectrum(new_ep, refExp = "COADREAD")"
- [discussion] DeepCC is for classification, not clustering: "specifically developed for addressing current challenges in classification but not clustering of cancer patients"
- [discussion] BluePrint benchmark cannot distinguish Luminal A from Luminal B: "can only predict Luminal, HER2, and Basal subtypes, but cannot distinguish between Luminal A and Luminal B subtypes"
