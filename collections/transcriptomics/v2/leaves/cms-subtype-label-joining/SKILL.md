---
name: cms-subtype-label-joining
description: Use when you have gene expression data from CRC pre-clinical models (PDXs,
  organoids, cell lines) plus drug-response annotations, and need to test whether
  a CMS subtype (e.g. CMS2) predicts response, as with cetuximab in Gao PDXs and Medle/Binter
  cell lines.
when_to_use_negative:
- Classifying samples whose human stromal component differs from primary CRCs, e.g.
  patient biopsies of metastatic CRC — the authors state CMScaller is not recommended
  for such samples
- Single-cell RNA-sequencing data, which is exceedingly noisy and not what CMScaller
  was optimized for
- Small datasets (fewer than ~40 samples), where centering/scaling of expression data
  inescapably introduces prediction uncertainty
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CMScaller
  role: Performs microenvironment-independent NTP-based CMS classification of pre-clinical
    models to produce the subtype labels being joined
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Runtime environment for CMScaller, contingency-table construction, fisher.test()
    association testing, and CSV export
- name: Biobase
  role: Provides ExpressionSet infrastructure for loading and handling cohort expression
    data as required by CMScaller
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:271e9ff1c202ad92aeaf1abee6057b3988e9380de4c68c4c51d41c1a6b863e10
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cms-subtype-label-joining

## Summary

Classify colorectal cancer pre-clinical models (PDXs, cell lines) into Consensus Molecular Subtypes using the CMScaller R package, join CMS labels to drug-response annotations by sample ID, and test CMS2 vs non-CMS2 response associations with Fisher's exact test.

## When to use

Use when you have gene expression data from CRC pre-clinical models (PDXs, organoids, cell lines) plus drug-response annotations, and need to test whether a CMS subtype (e.g. CMS2) predicts response, as with cetuximab in Gao PDXs and Medle/Binter cell lines.

## When NOT to use

- Classifying samples whose human stromal component differs from primary CRCs, e.g. patient biopsies of metastatic CRC — the authors state CMScaller is not recommended for such samples
- Single-cell RNA-sequencing data, which is exceedingly noisy and not what CMScaller was optimized for
- Small datasets (fewer than ~40 samples), where centering/scaling of expression data inescapably introduces prediction uncertainty

## Inputs

- Gene expression matrices or ExpressionSet objects for CRC pre-clinical cohorts (Gao PDXs, Isella GSE76402, Medle/Binter cell lines)
- CMScaller-predicted CMS class labels per sample/model
- Cetuximab (anti-EGFR) response annotations, e.g. Gao Supplementary Table 1 and Medle/Binter response group tables
- Sample/model identifier keys for joining labels to responses

## Outputs

- Per-cohort CMS2 vs non-CMS2 by responder vs non-responder contingency tables
- Fisher's exact test p-values and odds ratios per cohort and subset
- Combined results table (cohort, subset, p-value, odds ratio) written to cms2_cetuximab_association.csv

## How to apply

Install CMScaller from GitHub (devtools::install_github("Lothelab/CMScaller"), which requires Biobase and limma from Bioconductor) and load expression data as Biobase-compatible ExpressionSet or matrix objects. Run CMScaller, whose core algorithm is Nearest Template Prediction (NTP), on each cohort (Gao PDX, Isella GSE76402, Medle/Binter cell lines), setting RNAseq=TRUE for RNA-seq input; the caller is microenvironment-independent so it can classify CMS4-mesenchymal models that the original pCRC classifier misses. Join predicted CMS labels to response annotations (e.g. cetuximab response groups from Gao Supplementary Table 1) by sample identifier using dplyr, then build CMS2 vs non-CMS2 by responder vs non-responder contingency tables per cohort and subset (triple-WT PDXs, metastatic PDXs). Apply fisher.test() per table to extract p-values and odds ratios, targeting reported statistics such as p=0.024 (Gao PDXs), p=0.016 (triple-WT only), p=0.01/OR=2.9 (metastatic PDXs), and p=3.4e-3/OR=8.3 (cell lines), and write results to a CSV with cohort, subset, p-value, and odds ratio columns.

## Related tools

- **CMScaller** (Performs microenvironment-independent NTP-based CMS classification of pre-clinical models to produce the subtype labels being joined) — https://github.com/Lothelab/CMScaller
- **R** (Runtime environment for CMScaller, contingency-table construction, fisher.test() association testing, and CSV export)
- **Biobase** (Provides ExpressionSet infrastructure for loading and handling cohort expression data as required by CMScaller)

## Examples

```
res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- Gao PDX cohort reproduces the reported Fisher's exact p=0.024 for CMS2 vs non-CMS2 cetuximab response
- Triple-WT PDX subset reproduces p=0.016; metastatic PDX subset reproduces p=0.01 with OR=2.9
- Cell line cohort reproduces p=3.4e-3 with OR=8.3
- Each contingency table's margins sum to the correct cohort/subset sample counts after the label-to-response join
- Output cms2_cetuximab_association.csv contains cohort, subset, p-value, and odds ratio columns with one row per tested cohort/subset

## Limitations

- CMScaller in its current implementation is not recommended for samples with a different human stromal component than primary CRCs, including metastatic CRC biopsies; caution is warranted for highly selected datasets such as metastatic samples
- Input gene expression data must be centered and scaled, introducing prediction uncertainty for small datasets (fewer than ~40 samples)
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries

## Evidence

- [abstract] This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total).: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [readme] The `CMScaller` package provides Consensus Molecular Subtype (CMS) classification of colorectal cancer pre-clinical models: "The `CMScaller` package provides Consensus Molecular Subtype (CMS) classification of colorectal cancer pre-clinical models"
- [discussion] input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [discussion] CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs"
