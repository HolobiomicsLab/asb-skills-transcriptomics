---
name: cross-cohort-statistical-reproducibility
description: Use when a published finding links a categorical subtype/label (e.g.,
  CMS2 from CMScaller) to a binary drug-response outcome across multiple independent
  cohorts and the target is to reproduce the reported p-values and odds ratios per
  cohort.
when_to_use_negative:
- The subtype labels are not cancer-cell-intrinsic classifications (CMScaller was
  designed for pre-clinical models and is not recommended for metastatic CRC biopsies
  with different human stromal content)
- The outcome variable is continuous or multi-class rather than a binary responder/non-responder
  grouping suitable for a 2x2 Fisher's exact test
- Any single cohort contains fewer than approximately forty samples, where CMScaller's
  centered/scaled input introduces prediction uncertainty
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: CMScaller
  role: Provides the cancer-cell-intrinsic CMS2 vs non-CMS2 class labels for the pre-clinical
    models whose response is being tested
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Environment in which CMScaller labels, dplyr contingency tables, and fisher.test
    association statistics are computed
  repo: http://www.R-project.org/
- name: Biobase
  role: Loads the ExpressionSet/predicted-label objects holding the cohort expression
    data and class assignments
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:1419d7bb8ff465b4c2bf8d7ac702921a9683d349f313fa136f3cd65803218f35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cross-cohort-statistical-reproducibility

## Summary

Reproduces a reported biology-association finding (e.g., CMS2 classification predicting cetuximab response) by rebuilding cohort-specific contingency tables and applying per-cohort Fisher's exact tests, then verifying p-values and odds ratios against published values across PDX and cell line datasets.

## When to use

Use when a published finding links a categorical subtype/label (e.g., CMS2 from CMScaller) to a binary drug-response outcome across multiple independent cohorts and the target is to reproduce the reported p-values and odds ratios per cohort.

## When NOT to use

- The subtype labels are not cancer-cell-intrinsic classifications (CMScaller was designed for pre-clinical models and is not recommended for metastatic CRC biopsies with different human stromal content)
- The outcome variable is continuous or multi-class rather than a binary responder/non-responder grouping suitable for a 2x2 Fisher's exact test
- Any single cohort contains fewer than approximately forty samples, where CMScaller's centered/scaled input introduces prediction uncertainty

## Inputs

- CMScaller predicted CMS class labels for Gao PDX, Isella GSE76402, and Medle/Binter cell line cohorts (ExpressionSet/predicted-label objects)
- Cetuximab response annotations (e.g., Gao Supplementary Table 1; Medle/Binter response group tables) with sample/model identifiers
- Optional subset masks (triple-WT PDXs, metastatic PDXs)

## Outputs

- Per-cohort CMS2 vs non-CMS2 by responder vs non-responder contingency tables
- Fisher's exact test p-values and odds ratios per cohort/subset
- Combined results table cms2_cetuximab_association.csv with cohort, subset, p-value, and odds ratio columns

## How to apply

Load CMScaller class-label assignments for each cohort (Gao PDX, Isella GSE76402, Medle/Binter cell lines) as Biobase-compatible ExpressionSet/predicted-label objects. Import response annotations (e.g., Gao Supplementary Table 1), join sample IDs to CMS labels, and build CMS2 vs non-CMS2 by responder vs non-responder contingency tables per cohort and for restricted subsets (triple-WT PDXs, metastatic PDXs). Run fisher.test per table, extracting p-values and odds ratios. Judge success by comparing each reproduced statistic against reported values (p=0.024 Gao PDXs; p=0.016 triple-WT; p=0.01/OR=2.9 metastatic; p=3.4e-3/OR=8.3 cell lines) within tolerance, then write the combined results to cms2_cetuximab_association.csv.

## Related tools

- **CMScaller** (Provides the cancer-cell-intrinsic CMS2 vs non-CMS2 class labels for the pre-clinical models whose response is being tested) — https://github.com/Lothelab/CMScaller
- **R** (Environment in which CMScaller labels, dplyr contingency tables, and fisher.test association statistics are computed) — http://www.R-project.org/
- **Biobase** (Loads the ExpressionSet/predicted-label objects holding the cohort expression data and class assignments)

## Examples

```
res <- fisher.test(table(cms2=labels$prediction=="CMS2", responder=response$group)); write.csv(results, "cms2_cetuximab_association.csv")
```

## Evaluation signals

- Reproduced Gao PDX p-value falls within tolerance of 0.024 (all PDXs) and 0.016 (triple-WT only)
- Metastatic PDX result reproduces p=0.01 with odds ratio OR=2.9
- Cell line result reproduces p=3.4e-3 with odds ratio OR=8.3
- Each contingency table has the expected 2x2 structure (CMS2 vs non-CMS2 by responder vs non-responder) with no samples lost to failed ID joins
- cms2_cetuximab_association.csv contains one row per cohort/subset with non-missing cohort, subset, p-value, and odds ratio columns

## Limitations

- CMScaller is not recommended for samples with a different human stromal component than primary CRCs, e.g., metastatic CRC biopsies, so metastatic-subset classifications warrant caution
- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Caution is warranted for highly selected datasets where molecular distributions deviate from primary CRC cohorts, including metastatic samples
- Public pre-clinical drug screen data suffer from imprecise subtyping of models unless a pre-clinical-optimized classifier such as CMScaller is used

## Evidence

- [abstract] CMS subtypes are predictive of response to standard CRC drugs based on public drug response data from patient-derived xenografts and cell lines: "by analyzing public drug response data from patient-derived xenografts and cell lines, we show that the subtypes are predictive of response to standard CRC drugs"
- [discussion] CMScaller is platform independent and available as an R package: "CMScaller is platform independent and available as an R package."
- [intro] Public pre-clinical drug screen data hinder CMS-drug association analysis due to imprecise model subtyping: "analysis of CMS-drug associations is hindered by imprecise subtyping of the models."
- [discussion] Small datasets introduce prediction uncertainty after centering/scaling: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] CMScaller classifies pre-clinical models via the CMScaller function with Biobase loaded: "res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)"
