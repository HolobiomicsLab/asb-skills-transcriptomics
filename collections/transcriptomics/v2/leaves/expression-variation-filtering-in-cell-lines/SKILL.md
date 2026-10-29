---
name: expression-variation-filtering-in-cell-lines
description: Use when building a subtype/classification gene template from bulk tumor
  data that must generalize to cell lines, organoids or PDXs lacking human stromal/immune
  compartments.
when_to_use_negative:
- The target samples have a human stromal component different from primary CRCs, e.g.
  patient biopsies of metastatic CRC — CMScaller is explicitly not recommended there
- Input is single-cell RNA-sequencing data, which the authors call exceedingly noisy
  and for which a dedicated classifier is needed
- You only need to classify already-subtyped bulk primary tumors with full stromal
  content — the original CMS classifier applies
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CMScaller
  role: R package whose templates.CMS holds the filtered, cell-intrinsic marker gene
    sets used for nearest-template-prediction subtyping
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Implementation language for the filtering workflow and the CMScaller package
- name: Biobase
  role: Bioconductor data structures (ExpressionSet) used to assemble and store the
    template gene sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:d4cf0ec6718db81bc0422f628e03230a49eed5ce7d6a75d024477c88bdb5dd43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# Expression-variation filtering in cell lines

## Summary

A marker-gene filtering strategy from CMScaller (Eide 2017) that restricts the CMS subtype template to cancer cell-intrinsic genes by successively applying primary-CRC differential-expression, cell-line expression-variation (top 25% inter-percentile range, expressed in ≥3 lines), and pCRC-vs-PDX xenotransplantation (LFC < 2) filters.

## When to use

Use when building a subtype/classification gene template from bulk tumor data that must generalize to cell lines, organoids or PDXs lacking human stromal/immune compartments.

## When NOT to use

- The target samples have a human stromal component different from primary CRCs, e.g. patient biopsies of metastatic CRC — CMScaller is explicitly not recommended there
- Input is single-cell RNA-sequencing data, which the authors call exceedingly noisy and for which a dedicated classifier is needed
- You only need to classify already-subtyped bulk primary tumors with full stromal content — the original CMS classifier applies

## Inputs

- pCRC subtype-vs-rest differential expression results (gene symbols with log2 fold change and adjusted p-value)
- CRC cell line gene expression matrix (e.g. from CCLE/linCS-type data)
- Matched pCRC–PDX paired expression data for xenotransplantation filtering

## Outputs

- Four subtype-specific template gene sets (CMS1–CMS4) of cancer cell-intrinsic marker genes
- templates.CMS object usable by the CMScaller R package for NTP-based subtype prediction

## How to apply

Start from primary CRC (pCRC) subtype-vs-rest differential expression results and retain candidate markers with log2 fold change > 1 and adjusted p-value < 0.1. On a CRC cell line expression matrix, keep only genes in the top 25% inter-percentile range (IPR) that are detected as expressed in at least 3 cell lines, ensuring markers actually vary in the pre-clinical models. Remove any candidate whose expression shifts by LFC >= 2 between matched pCRC and patient-derived xenograft (PDX) samples, dropping genes lost upon xenotransplantation that likely reflect microenvironment signals. Assemble surviving genes into four subtype-specific template gene sets (CMS1–CMS4) stored in templates.CMS within the CMScaller R package using Biobase data structures. The rationale is that the original CMS classifier depends on immune and stromal signals and fails on pre-clinical models, so the template must be cancer cell-intrinsic.

## Related tools

- **CMScaller** (R package whose templates.CMS holds the filtered, cell-intrinsic marker gene sets used for nearest-template-prediction subtyping) — https://github.com/Lothelab/CMScaller
- **R** (Implementation language for the filtering workflow and the CMScaller package)
- **Biobase** (Bioconductor data structures (ExpressionSet) used to assemble and store the template gene sets)

## Examples

```
library(CMScaller); res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- Every retained template gene has pCRC subtype-vs-rest log2 fold change > 1 and adjusted p-value < 0.1
- All retained genes fall in the top 25% inter-percentile range of the CRC cell line expression matrix and are detected in at least 3 cell lines
- No retained gene shows pCRC-to-PDX log2 fold change >= 2 (xenotransplantation-loss filter applied with no p-value threshold)
- Templates resolve into four non-empty CMS1–CMS4 gene sets loadable from templates.CMS
- The resulting classifier recapitulates CMS subtypes in in vitro and in vivo models (551 models in the published validation)

## Limitations

- Not recommended for samples with a different human stromal component than primary CRCs, including metastatic CRC biopsies
- Input expression data must be centered and scaled, introducing prediction uncertainty for small datasets (fewer than ~40 samples)
- Not optimized for single-cell RNA-sequencing data; caution in highly selected datasets whose molecular distributions deviate from pCRC cohorts

## Evidence

- [abstract] The original classifier depends on stromal signals and fails on pre-clinical models: "The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group"
- [abstract] The filter yields a classifier that works across 551 in vitro and in vivo models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [readme] The core algorithm is Nearest Template Prediction (Hoshida 2010): [evidence span withheld — anchor status: not_found]
- [readme] Purpose is CMS classification of pre-clinical models: "The `CMScaller` package provides Consensus Molecular Subtype (CMS) classification of colorectal cancer pre-clinical models"
- [discussion] Explicit scope statement about metastatic samples: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs, including e.g. patient biopsies of metastatic CRC."
