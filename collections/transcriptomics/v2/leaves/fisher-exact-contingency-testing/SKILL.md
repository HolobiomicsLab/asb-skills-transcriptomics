---
name: fisher-exact-contingency-testing
description: Use when testing a categorical association between CMScaller CMS class
  assignments (e.g., CMS2 vs non-CMS2, CMS1 vs non-CMS1) and a binary phenotype (drug
  responder status, MSI status, BRAF mutation) across PDX, organoid, or cell line
  cohorts.
when_to_use_negative:
- Comparing continuous measurements across CMS groups (e.g., total mutation counts
  or CNA burden in Gao PDXs, reported p=9.9e-4 and p=5.9e-3) — use a two-sided Mann-Whitney
  U test (wilcox.test) instead
- The exposure variable is already continuous or ordered (e.g., drug IC50 values)
  rather than a pre-binarized responder/non-responder label
- The sample-level class labels have not been generated yet — CMScaller classification
  must be run first on centered and scaled gene expression data
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: CMScaller
  role: Supplies the CMS class predictions (CMS1-4) for each pre-clinical model that
    form one margin of the contingency table
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Runtime environment; fisher.test() (stats package) performs the exact test,
    table() builds the 2x2 contingency table, dplyr joins sample IDs to phenotypes
- name: Biobase
  role: Loads the ExpressionSet objects holding expression data and sample metadata
    from which CMS labels and phenotypes are extracted
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:ff8a8ff6df3b42284b888635055e91b47c7a1d01ae068731aa8e7310de2719dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fisher-exact-contingency-testing

## Summary

Apply R's fisher.test() to 2x2 contingency tables to test categorical associations in CMScaller-based pre-clinical CRC analyses — e.g., CMS2 versus non-CMS2 classification against cetuximab responder status, or CMS1-immune prediction against MSI status — reporting exact p-values and odds ratios.

## When to use

Use when testing a categorical association between CMScaller CMS class assignments (e.g., CMS2 vs non-CMS2, CMS1 vs non-CMS1) and a binary phenotype (drug responder status, MSI status, BRAF mutation) across PDX, organoid, or cell line cohorts.

## When NOT to use

- Comparing continuous measurements across CMS groups (e.g., total mutation counts or CNA burden in Gao PDXs, reported p=9.9e-4 and p=5.9e-3) — use a two-sided Mann-Whitney U test (wilcox.test) instead
- The exposure variable is already continuous or ordered (e.g., drug IC50 values) rather than a pre-binarized responder/non-responder label
- The sample-level class labels have not been generated yet — CMScaller classification must be run first on centered and scaled gene expression data

## Inputs

- CMScaller class label assignments (e.g., CMS2 vs non-CMS2 predictions) for a pre-clinical cohort (Gao PDX, Isella GSE76402, Medle/Binter cell lines, Fujii organoids) as R vectors or Biobase-compatible ExpressionSet metadata
- Binary phenotype annotations keyed by sample/model ID (cetuximab responder status from Gao Supplementary Table 1, MSI status for organoids, BRAF mutation status)
- Optional subset filters (triple-WT PDXs, metastatic PDXs) as sample ID vectors

## Outputs

- Per-cohort/per-subset Fisher's exact test results: contingency table, p-value, odds ratio
- Combined results table (cohort, subset, test, group sizes, p-value, OR) saved as a CSV such as cms2_cetuximab_association.csv or statistical_association_tests.csv

## How to apply

Build a 2x2 contingency table with table() joining each model's CMScaller label (e.g., CMS2 vs non-CMS2) to its binary annotation (responder vs non-responder, MSI-positive vs MSI-negative) on sample IDs. Run fisher.test() in R (two-sided) per cohort and per restricted subset — e.g., Gao PDXs overall, triple-WT PDXs only, and metastatic PDXs for the cetuximab analysis. Extract both the p-value and the odds ratio from the test object and target the reported statistics: p=0.024 (Gao PDXs), p=0.016 (triple-WT only), p=0.01/OR=2.9 (metastatic PDXs), p=3.4e-3/OR=8.3 (Medle/Binter cell lines), and p=1.6e-4 (Fujii organoid CMS1-MSI association, 5/6 MSI organoids predicted CMS1). Use Fisher's exact test rather than chi-square because group sizes are small (tens of models per cohort). Compile cohort, subset, test statistic, and p-value into a results CSV and verify each reproduced statistic falls within tolerance of the reported value.

## Related tools

- **CMScaller** (Supplies the CMS class predictions (CMS1-4) for each pre-clinical model that form one margin of the contingency table) — https://github.com/Lothelab/CMScaller
- **R** (Runtime environment; fisher.test() (stats package) performs the exact test, table() builds the 2x2 contingency table, dplyr joins sample IDs to phenotypes)
- **Biobase** (Loads the ExpressionSet objects holding expression data and sample metadata from which CMS labels and phenotypes are extracted)

## Examples

```
fisher.test(table(cms2 = model_cms == "CMS2", responder = model_response == "responder"))
```

## Evaluation signals

- Reproduced p-values fall within tolerance of the reported values: p=0.024 (Gao PDXs), p=0.016 (triple-WT), p=0.01 with OR=2.9 (metastatic PDXs), p=3.4e-3 with OR=8.3 (cell lines)
- CMS1-MSI test reproduces 5 of 6 MSI organoids predicted CMS1-immune with p=1.6e-4
- Each 2x2 contingency table has row and column sums matching the cohort's class counts after the sample-ID join (no dropped or duplicated models)
- Direction of association is consistent: CMS2 models enrich among cetuximab responders (OR > 1 in every cohort)
- Output CSV contains one row per cohort/subset with non-missing p-value and odds-ratio columns

## Limitations

- CMScaller itself is not recommended for samples with a different human stromal component than primary CRCs, including metastatic CRC biopsies — caution applies to the metastatic-PDX subset analysis
- CMScaller input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples), affecting downstream contingency-table cell counts
- Caution is warranted in highly selected datasets where molecular distributions deviate from primary CRC cohorts, which can bias class proportions in the contingency table

## Evidence

- [results] CMS2-classified models respond more strongly to cetuximab than non-CMS2 models, confirmed in PDXs (p = 0.024; p = 0.016 in triple-WT only), metastatic PDXs (p = 0.01, OR = 2.9), and cell lines (p = 3.4e-3, OR = 8.3).: [evidence span withheld — anchor status: not_found]
- [discussion] The PDX and cell line drug response data presented here suggest that the CMS-groups add additional predictive information, indicating a poor response in CMS4.: "The PDX and cell line drug response data presented here suggest that the CMS-groups add additional predictive information, indicating a poor response in CMS4."
- [methods] Apply Fisher's exact test (fisher.test) per cohort to estimate the CMS2-response association, extracting p-values and odds ratios.: [evidence span withheld — anchor status: not_found]
- [results] In the Fujii et al. organoid dataset, 5 of 6 MSI samples were predicted to be CMS1-immune by CMScaller, a significant association (Fisher's exact test, p = 1.6 × 10−4).: [evidence span withheld — anchor status: not_found]
- [readme] res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE): "res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)"
