---
name: subtype-marker-selection
description: Use when building subtype classification templates for pre-clinical CRC
  models (cell lines, organoids, PDXs) where microenvironment-derived gene signals
  must be excluded from the marker set.
when_to_use_negative:
- Samples with a different human stromal component than primary CRCs, e.g. patient
  biopsies of metastatic CRC — CMScaller is explicitly not recommended in its current
  implementation for such samples.
- Single-cell RNA-sequencing data, which is exceedingly noisy and for which a dedicated
  classifier would be required.
- Highly selected datasets whose molecular distributions severely deviate from pCRC
  cohorts (e.g. metastatic samples), where caution is warranted.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: CMScaller
  role: R package whose templates.CMS gene sets are assembled from the filtered markers
    and which performs the downstream NTP-based subtype prediction
  repo: github.com/Lothelab/CMScaller
- name: R
  role: Implementation language for the differential expression, cell line IPR, and
    pCRC-PDX filtering steps
  repo: http://www.R-project.org/
- name: Biobase
  role: Bioconductor data structures used to hold expression data and template gene
    sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:4e430252a3cce1efc2594a79a4fbae723ea7ffedf2018bf00966399de91e4e9b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# subtype-marker-selection

## Summary

Reconstructs CMScaller's cancer cell-intrinsic CMS marker filtering pipeline, which successively narrows primary CRC (pCRC) subtype-vs-rest differential expression hits by cell line expression variation and by expression loss upon xenotransplantation, so the final template reflects cancer-cell rather than stromal/immune signals.

## When to use

Use when building subtype classification templates for pre-clinical CRC models (cell lines, organoids, PDXs) where microenvironment-derived gene signals must be excluded from the marker set.

## When NOT to use

- Samples with a different human stromal component than primary CRCs, e.g. patient biopsies of metastatic CRC — CMScaller is explicitly not recommended in its current implementation for such samples.
- Single-cell RNA-sequencing data, which is exceedingly noisy and for which a dedicated classifier would be required.
- Highly selected datasets whose molecular distributions severely deviate from pCRC cohorts (e.g. metastatic samples), where caution is warranted.

## Inputs

- Primary CRC (pCRC) subtype-vs-rest differential expression results (log2 fold change and adjusted p-value per gene)
- CRC cell line expression matrix (for IPR and detection filtering)
- Matched pCRC-PDX paired expression data (for xenotransplantation LFC filtering)

## Outputs

- Four subtype-specific template gene sets (CMS1-CMS4) of cancer cell-intrinsic markers stored in templates.CMS
- Expression template usable by the CMScaller/NTP classifier on pre-clinical model expression data

## How to apply

Start from pCRC subtype-vs-rest differential expression results and retain candidate marker genes with log2 fold change > 1 and adjusted p-value < 0.1. Next, using a CRC cell line expression matrix, keep only genes in the top 25% inter-percentile range (IPR) of expression variation that are detected as expressed in at least 3 cell lines, removing genes not variable or not detectable in vitro. Then apply the xenotransplantation filter: remove candidates whose expression changes with LFC >= 2 between matched pCRC and patient-derived xenograft (PDX) samples (no p-value threshold), which eliminates microenvironment-derived signals absent from xenografts. Assemble the surviving genes into four subtype-specific template gene sets (CMS1-CMS4) and store them in templates.CMS using Biobase data structures for use with the Nearest Template Prediction (NTP) core algorithm. The rationale is that the original CMS classifier fails on pre-clinical models because it depends on immune and stromal expression signals, so filtering for cancer cell-intrinsic markers restores CMS1 and CMS4 detection in cell lines, organoids and PDXs.

## Related tools

- **CMScaller** (R package whose templates.CMS gene sets are assembled from the filtered markers and which performs the downstream NTP-based subtype prediction) — github.com/Lothelab/CMScaller
- **R** (Implementation language for the differential expression, cell line IPR, and pCRC-PDX filtering steps) — http://www.R-project.org/
- **Biobase** (Bioconductor data structures used to hold expression data and template gene sets)

## Examples

```
devtools::install_github("Lothelab/CMScaller"); library(Biobase); library(CMScaller); res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- Each retained candidate gene must have pCRC subtype-vs-rest log2 fold change > 1 and adjusted p-value < 0.1.
- Genes failing the cell line filter (outside top 25% IPR or not expressed in at least 3 cell lines) must be dropped.
- Any candidate with matched pCRC-PDX log2 fold change >= 2 must be removed; the final template consists only of genes not lost upon xenotransplantation.
- The resulting templates.CMS object must contain four subtype-specific gene sets (CMS1-CMS4) as Biobase-compatible structures.
- Downstream NTP classification should recapitulate CMS subtypes, including poor-prognostic CMS4-mesenchymal and CMS1-immune groups, in in vitro and in vivo models (551 models in the paper).

## Limitations

- CMScaller is not recommended for samples with a different human stromal component than primary CRCs, including metastatic CRC biopsies.
- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples).
- Caution is warranted in highly selected datasets where molecular distributions deviate from pCRC cohorts, including metastatic samples.
- The classifier is not optimized for single-cell RNA-sequencing data, which is exceedingly noisy.
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic factors) may explain prediction errors at class boundaries; the template gene set is planned to be improved with larger, higher-quality datasets.

## Evidence

- [methods] The final filtering criterion retained only genes not lost upon xenotransplantation (pCRC-PDX log2 fold-change < 2, no p-value threshold): [evidence span withheld — anchor status: not_found]
- [abstract] The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group: "The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group"
- [abstract] This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total).: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs, including e.g. patient biopsies of metastatic CRC."
- [readme] The core algorithm is Nearest Template Prediction (NTP) algorithm proposed by Yujin Hoshida: "The core algorithm is *[Nearest Template Prediction (NTP)](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0015543)* algorithm proposed by Yujin Hoshida"
