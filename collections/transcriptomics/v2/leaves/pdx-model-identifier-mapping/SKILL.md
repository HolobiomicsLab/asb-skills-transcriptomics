---
name: pdx-model-identifier-mapping
description: Use when pre-clinical CRC cohorts (PDX or cell line) have CMS predictions
  from CMScaller and independently annotated drug-response labels that must be joined
  on sample/model identifiers before testing subtype-response associations.
when_to_use_negative:
- Sample is a metastatic CRC biopsy or other tissue with a different human stromal
  component than primary CRCs — CMScaller predictions feeding the mapping are not
  recommended for such samples
- Gene expression dataset is very small (fewer than ~40 samples), where centering/scaling
  of inputs introduces prediction uncertainty in the CMS labels being mapped
- The data are single-cell RNA-sequencing, which is exceedingly noisy and outside
  CMScaller's intended scope
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: CMScaller
  role: Produces the microenvironment-independent CMS class labels per pre-clinical
    model that are joined to response annotations via sample IDs
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Runtime environment for loading labels, performing dplyr ID joins, building
    contingency tables, and running fisher.test
- name: Biobase
  role: Provides ExpressionSet-style container objects in which CMScaller predictions
    and cohort expression data are loaded
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:21ac7a7c09c7fa85c1f0a4788c1dfa1684a802af08e638fb9e90721efb15f4c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pdx-model-identifier-mapping

## Summary

Maps pre-clinical colorectal cancer model identifiers (PDX and cell line sample IDs) to their CMScaller CMS class labels and external drug-response annotations via ID joins, enabling contingency-table analyses of CMS2 vs non-CMS2 response to cetuximab across cohorts such as Gao PDXs, Isella GSE76402, and Medle/Binter cell lines.

## When to use

Use when pre-clinical CRC cohorts (PDX or cell line) have CMS predictions from CMScaller and independently annotated drug-response labels that must be joined on sample/model identifiers before testing subtype-response associations.

## When NOT to use

- Sample is a metastatic CRC biopsy or other tissue with a different human stromal component than primary CRCs — CMScaller predictions feeding the mapping are not recommended for such samples
- Gene expression dataset is very small (fewer than ~40 samples), where centering/scaling of inputs introduces prediction uncertainty in the CMS labels being mapped
- The data are single-cell RNA-sequencing, which is exceedingly noisy and outside CMScaller's intended scope

## Inputs

- CMScaller predicted CMS class labels for Gao PDX, Isella GSE76402, and Medle/Binter cell line cohorts (Biobase ExpressionSet / predicted-label objects)
- Cetuximab (anti-EGFR) response annotations from Gao Supplementary Table 1
- Medle/Binter cell line response group tables
- Sample/model identifier keys for joining CMS labels to response annotations

## Outputs

- Per-cohort joined tables of model ID, CMS class, and cetuximab responder status
- CMS2 vs non-CMS2 by responder vs non-responder contingency tables (full cohorts and restricted subsets)
- cms2_cetuximab_association.csv with cohort, subset, p-value, and odds ratio columns

## How to apply

Load the CMScaller class label assignments for each cohort (Gao PDX, Isella GSE76402, Medle/Binter cell lines) as Biobase-compatible ExpressionSet/predicted-label objects. Import external response annotations — Gao Supplementary Table 1 for PDXs and the Medle/Binter cell line response group tables — and join them to the CMS predictions on sample/model IDs using dplyr joins, respecting cohort-specific ID conventions. Build CMS2 vs non-CMS2 by responder vs non-responder contingency tables per cohort and per restricted subset (triple-WT PDXs, metastatic PDXs) with dplyr/table. Apply fisher.test per contingency table and verify p-values and odds ratios against reported targets (p=0.024 Gao PDXs; p=0.016 triple-WT; p=0.01, OR=2.9 metastatic PDXs; p=3.4e-3, OR=8.3 cell lines).

## Related tools

- **CMScaller** (Produces the microenvironment-independent CMS class labels per pre-clinical model that are joined to response annotations via sample IDs) — https://github.com/Lothelab/CMScaller
- **R** (Runtime environment for loading labels, performing dplyr ID joins, building contingency tables, and running fisher.test)
- **Biobase** (Provides ExpressionSet-style container objects in which CMScaller predictions and cohort expression data are loaded)

## Examples

```
res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE); merged <- dplyr::left_join(res, response_table, by = c("sample" = "model_id"))
```

## Evaluation signals

- Every joined row has a valid CMS class label (CMS1–CMS4) and a defined responder status after the ID join; unmapped IDs are logged and excluded explicitly
- Contingency tables are correctly stratified for each restricted subset (triple-WT PDXs, metastatic PDXs) before testing
- Reproduced Fisher's exact test statistics fall within tolerance of reported values: p=0.024 (Gao PDXs), p=0.016 (triple-WT only), p=0.01 with OR=2.9 (metastatic PDXs), p=3.4e-3 with OR=8.3 (cell lines)
- cms2_cetuximab_association.csv contains cohort, subset, p-value, and odds ratio columns with one row per tested cohort/subset
- Overall pattern shows CMS2-classified models responding more strongly to cetuximab than non-CMS2 models in every cohort

## Limitations

- CMScaller in its current implementation is not recommended for samples with a different human stromal component than primary CRCs, e.g. metastatic CRC biopsies, so mapped metastatic-PDX labels warrant caution
- Input gene expression data must be centered and scaled, introducing prediction uncertainty for small datasets (fewer than approximately forty samples)
- Caution is warranted in highly selected datasets where molecular distributions deviate from pCRC cohorts, including metastatic samples

## Evidence

- [abstract] CMS classification of PDX/cell line models: "by analyzing public drug response data from patient-derived xenografts and cell lines, we show that the subtypes are predictive of response to standard CRC drugs"
- [abstract] CMScaller recapitulates subtypes in pre-clinical models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] Metastatic sample caution: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs"
- [readme] Package purpose: "The `CMScaller` package provides Consensus Molecular Subtype (CMS) classification of colorectal cancer pre-clinical models"
