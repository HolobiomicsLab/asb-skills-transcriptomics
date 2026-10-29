---
name: cetuximab-response-stratification
description: Use when assigning CMS labels to CRC PDX/cell-line expression data with
  CMScaller and testing whether CMS2-classified models respond more to cetuximab than
  non-CMS2 models.
when_to_use_negative:
- Input samples are metastatic CRC biopsies or otherwise have a human stromal component
  different from primary CRCs — CMScaller's current implementation is not recommended
  for such samples
- Datasets with fewer than approximately forty samples, where centering/scaling inescapably
  introduces prediction uncertainty
- Single-cell RNA-seq data, which is exceedingly noisy and for which CMScaller is
  not optimized
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CMScaller
  role: NTP-based, microenvironment-independent CMS classification of the pre-clinical
    models (PDXs and cell lines) prior to response testing
  repo: github.com/Lothelab/CMScaller
- name: R
  role: Host language for CMScaller, the dplyr/table contingency-table construction,
    and fisher.test() per cohort
  repo: http://www.R-project.org/
- name: Biobase
  role: Loading and handling ExpressionSet gene expression objects for the Gao PDX
    and cell line cohorts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:1b5eef39c6178730dffd8d8311326fa9034b70bb2c313621e59f45c12623c0c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cetuximab-response-stratification

## Summary

A workflow that classifies colorectal cancer pre-clinical models (PDXs, cell lines) into Consensus Molecular Subtypes with CMScaller and statistically tests the CMS2-vs-non-CMS2 association with cetuximab (anti-EGFR) response using Fisher's exact tests per cohort.

## When to use

Use when assigning CMS labels to CRC PDX/cell-line expression data with CMScaller and testing whether CMS2-classified models respond more to cetuximab than non-CMS2 models.

## When NOT to use

- Input samples are metastatic CRC biopsies or otherwise have a human stromal component different from primary CRCs — CMScaller's current implementation is not recommended for such samples
- Datasets with fewer than approximately forty samples, where centering/scaling inescapably introduces prediction uncertainty
- Single-cell RNA-seq data, which is exceedingly noisy and for which CMScaller is not optimized

## Inputs

- Gene expression matrices (Biobase-compatible ExpressionSet objects) for the Gao PDX, Isella GSE76402, and Medle/Binter cell line cohorts
- CMScaller-predicted CMS class labels per model/sample
- Cetuximab response annotations from Gao Supplementary Table 1 and Medle/Binter cell line response-group tables

## Outputs

- cms2_cetuximab_association.csv containing cohort, subset, p-value, and odds ratio columns
- Per-cohort CMS2 vs non-CMS2 by responder vs non-responder contingency tables

## How to apply

Install CMScaller from GitHub (devtools::install_github("Lothelab/CMScaller") with Biobase and limma dependencies), then run CMScaller() on centered and scaled expression matrices (RNAseq=TRUE for RNA-seq data) to obtain NTP-based CMS predictions for the Gao PDX, Isella GSE76402, and Medle/Binter cell line cohorts. Join the predicted CMS labels to cetuximab response annotations from Gao Supplementary Table 1 and the Medle/Binter response-group tables on sample/model identifiers using dplyr. For each cohort — and for restricted subsets such as triple-WT PDXs and metastatic PDXs — build CMS2 vs non-CMS2 by responder vs non-responder contingency tables with table(), then apply fisher.test() to extract p-values and odds ratios. Target the reported statistics: p=0.024 (Gao PDXs), p=0.016 (triple-WT only), p=0.01 with OR=2.9 (metastatic PDXs), and p=3.4e-3 with OR=8.3 (cell lines). Write the results to cms2_cetuximab_association.csv with cohort, subset, p-value, and odds ratio columns and verify each statistic falls within tolerance of the reported values.

## Related tools

- **CMScaller** (NTP-based, microenvironment-independent CMS classification of the pre-clinical models (PDXs and cell lines) prior to response testing) — github.com/Lothelab/CMScaller
- **R** (Host language for CMScaller, the dplyr/table contingency-table construction, and fisher.test() per cohort) — http://www.R-project.org/
- **Biobase** (Loading and handling ExpressionSet gene expression objects for the Gao PDX and cell line cohorts)

## Examples

```
res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- Reproduced Fisher's exact p-values match reported values within tolerance: p=0.024 (Gao PDXs), p=0.016 (triple-WT only), p=0.01 (metastatic PDXs), p=3.4e-3 (cell lines)
- Reproduced odds ratios match reported values: OR=2.9 for metastatic PDXs and OR=8.3 for cell lines
- Contingency table margins sum to the correct sample counts after joining CMS labels to response annotations on sample IDs
- CMScaller predictions are produced for all models after expression data are centered and scaled (RNAseq=TRUE for RNA-seq inputs)
- Output file cms2_cetuximab_association.csv contains cohort, subset, p-value, and odds ratio columns and all cohorts show CMS2 favoring cetuximab response

## Limitations

- CMScaller is not recommended for samples with a human stromal component different from primary CRCs, including patient biopsies of metastatic CRC
- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Caution is warranted for highly selected datasets where molecular distributions deviate from primary CRC cohorts, including metastatic samples

## Evidence

- [abstract] CMScaller recapitulates CMS subtypes in pre-clinical models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [abstract] CMS subtypes predict response to standard CRC drugs: "by analyzing public drug response data from patient-derived xenografts and cell lines, we show that the subtypes are predictive of response to standard CRC drugs"
- [discussion] Not recommended for metastatic biopsies: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs, including e.g. patient biopsies"
- [discussion] Centering/scaling requirement for small datasets: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] CMScaller core algorithm is Nearest Template Prediction: [evidence span withheld — anchor status: not_found]
