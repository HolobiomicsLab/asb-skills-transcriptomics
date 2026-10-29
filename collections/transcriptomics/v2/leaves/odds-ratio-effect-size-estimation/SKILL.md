---
name: odds-ratio-effect-size-estimation
description: Use when comparing a binary biomarker/class label (e.g., CMS2 vs non-CMS2
  from CMScaller) against a binary drug-response annotation in PDX or cell line cohorts,
  and you need per-cohort p-values and odds ratios to quantify the association.
when_to_use_negative:
- The outcome or the class variable is not binary (e.g., continuous drug-response
  metrics such as IC50) — Fisher's exact test and odds ratios require 2x2 contingency
  tables
- Expected cell counts are large enough that a chi-square test is appropriate and
  the rarity assumption underlying odds ratios does not hold
- The class labels come from samples with a human stromal component unlike primary
  CRCs (e.g., metastatic biopsies), where CMScaller predictions are unreliable and
  any downstream association is confounded
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: R
  role: Statistical environment in which contingency tables are built (dplyr/table)
    and fisher.test is run to extract p-values and odds ratios
- name: CMScaller
  role: Provides the microenvironment-independent CMS class labels (CMS2 vs non-CMS2)
    for the pre-clinical models that form one margin of the contingency tables
  repo: https://github.com/Lothelab/CMScaller
- name: Biobase
  role: Used to load ExpressionSet-based gene expression / prediction objects in R
    for the model cohorts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:6afc55d96b6c5e8744b5e3dee7916d28e0fcb3c58fff145dc3d7fe37eae8e29c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# odds-ratio-effect-size-estimation

## Summary

Estimates the strength of association between a binary classification (e.g., CMS2 vs non-CMS2 in pre-clinical colorectal cancer models) and a binary outcome (e.g., cetuximab responder vs non-responder) by building 2x2 contingency tables and applying Fisher's exact test in R, reporting p-values and odds ratios per cohort and subset.

## When to use

Use when comparing a binary biomarker/class label (e.g., CMS2 vs non-CMS2 from CMScaller) against a binary drug-response annotation in PDX or cell line cohorts, and you need per-cohort p-values and odds ratios to quantify the association.

## When NOT to use

- The outcome or the class variable is not binary (e.g., continuous drug-response metrics such as IC50) — Fisher's exact test and odds ratios require 2x2 contingency tables
- Expected cell counts are large enough that a chi-square test is appropriate and the rarity assumption underlying odds ratios does not hold
- The class labels come from samples with a human stromal component unlike primary CRCs (e.g., metastatic biopsies), where CMScaller predictions are unreliable and any downstream association is confounded

## Inputs

- CMScaller class label assignments (CMS2 vs non-CMS2) for PDX cohorts (Gao) and cell line cohorts (Isella GSE76402, Medle/Binter) as R objects (ExpressionSet / predicted-label tables)
- Binary drug response annotations, e.g., cetuximab responder status from Gao Supplementary Table 1 and Medle/Binter cell line response group tables
- Sample/model identifier mapping for joining labels to responses
- Optional subset definitions (triple-WT PDXs, metastatic PDXs)

## Outputs

- Per-cohort/per-subset 2x2 contingency tables (CMS2 vs non-CMS2 by responder vs non-responder)
- Fisher's exact test p-values and odds ratios per cohort and subset
- Combined results table cms2_cetuximab_association.csv with cohort, subset, p-value, and odds ratio columns

## How to apply

Join the model-level class labels (e.g., CMScaller predictions for the Gao PDX, Isella GSE76402, and Medle/Binter cell line cohorts) to binary response annotations (e.g., cetuximab response from Gao Supplementary Table 1) on sample/model identifiers. For each cohort — and for biologically motivated restricted subsets such as triple-WT PDXs and metastatic PDXs — construct CMS2-vs-non-CMS2 by responder-vs-non-responder contingency tables using dplyr/table. Apply fisher.test per table, extracting the p-value and odds ratio; in the source analysis this reproduced p=0.024 (Gao PDXs), p=0.016 (triple-WT only), p=0.01 with OR=2.9 (metastatic PDXs), and p=3.4e-3 with OR=8.3 (cell lines). Write cohort, subset, p-value, and odds ratio columns to a combined results table (cms2_cetuximab_association.csv) and verify each statistic against reported values within tolerance.

## Related tools

- **R** (Statistical environment in which contingency tables are built (dplyr/table) and fisher.test is run to extract p-values and odds ratios)
- **CMScaller** (Provides the microenvironment-independent CMS class labels (CMS2 vs non-CMS2) for the pre-clinical models that form one margin of the contingency tables) — https://github.com/Lothelab/CMScaller
- **Biobase** (Used to load ExpressionSet-based gene expression / prediction objects in R for the model cohorts)

## Examples

```
fisher.test(table(cms_class = labels$CMS2, response = resp$responder))  # per cohort; extract p.value and estimate (odds ratio)
```

## Evaluation signals

- Each reproduced p-value falls within a stated tolerance of the reported values: p=0.024 (Gao PDXs), p=0.016 (triple-WT only), p=0.01 with OR=2.9 (metastatic PDXs), p=3.4e-3 with OR=8.3 (cell lines)
- Contingency tables for each cohort/subset are 2x2 with margins matching the number of joined samples (no samples lost or duplicated in the ID join)
- Results table cms2_cetuximab_association.csv contains exactly the columns cohort, subset, p-value, odds ratio with one row per tested cohort/subset
- All reported odds ratios are >1 for CMS2 responders, consistent with the direction of the finding that CMS2 models respond more strongly to cetuximab
- fisher.test is applied without continuity corrections or approximations that would alter exact p-values for small PDX cohorts

## Limitations

- CMScaller labels are not recommended for samples with a different human stromal component than primary CRCs, e.g., metastatic CRC biopsies, so associations in metastatic subsets carry classification uncertainty
- Input gene expression must be centered and scaled, and small datasets (fewer than ~40 samples) inescapably introduce prediction uncertainty that propagates into the association statistics
- Caution is warranted in highly selected datasets where molecular distributions deviate from primary CRC cohorts, including metastatic samples
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries, affecting contingency table assignment

## Evidence

- [other] CMS2-classified models respond more strongly to cetuximab than non-CMS2 models, confirmed in PDXs (p = 0.024; p = 0.016 in triple-WT only), metastatic PDXs (p = 0.01, OR = 2.9), and cell lines (p = 3.4e-3, OR = 8.3).: [evidence span withheld — anchor status: not_found]
- [readme] The core algorithm is Nearest Template Prediction (NTP) as documented in the CMScaller README.: [evidence span withheld — anchor status: not_found]
- [abstract] CMS subtypes are predictive of response to standard CRC drugs based on pre-clinical drug response data.: "by analyzing public drug response data from patient-derived xenografts and cell lines, we show that the subtypes are predictive of response to standard CRC drugs"
- [discussion] CMScaller should not be used on samples with a different stromal component, such as metastatic biopsies.: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs"
- [discussion] Input gene expression data should be centered and scaled, introducing prediction uncertainty in small datasets.: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
