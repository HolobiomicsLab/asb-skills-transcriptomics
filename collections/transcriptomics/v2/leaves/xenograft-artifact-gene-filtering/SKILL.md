---
name: xenograft-artifact-gene-filtering
description: Use when building CMS subtype prediction templates for cell lines, organoids
  or PDXs, where original CMS markers driven by immune/stromal expression fail to
  classify CMS1/CMS4 groups.
when_to_use_negative:
- Classifying primary CRC (pCRC) samples themselves, where the original stroma-dependent
  CMS classifier is appropriate
- Samples with a different human stromal component than primary CRCs, e.g. patient
  biopsies of metastatic CRC
- Single-cell RNA-sequencing data, which is exceedingly noisy and not covered by this
  classifier
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CMScaller
  role: R package that hosts the filtered subtype templates and performs Nearest Template
    Prediction classification of pre-clinical models
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Implementation language for the differential-expression, IPR and pCRC-PDX
    filtering workflow
- name: Biobase
  role: Bioconductor data structures used to assemble and store the template gene
    sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:535a60e11bb0f53a61ebd7b0ccc84c83f9e43a81cc3dbe32c3abddf1904108d6
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# xenograft-artifact-gene-filtering

## Summary

A filtering procedure from CMScaller that removes genes whose expression changes upon xenotransplantation (pCRC-PDX log2 fold-change >= 2) so that CMS subtype marker templates consist of cancer cell-intrinsic signals rather than stromal/immune artifacts absent in pre-clinical models.

## When to use

Use when building CMS subtype prediction templates for cell lines, organoids or PDXs, where original CMS markers driven by immune/stromal expression fail to classify CMS1/CMS4 groups.

## When NOT to use

- Classifying primary CRC (pCRC) samples themselves, where the original stroma-dependent CMS classifier is appropriate
- Samples with a different human stromal component than primary CRCs, e.g. patient biopsies of metastatic CRC
- Single-cell RNA-sequencing data, which is exceedingly noisy and not covered by this classifier

## Inputs

- pCRC subtype-vs-rest differential expression results table (gene, log2FC, adjusted p-value)
- CRC cell line gene expression matrix (for IPR variation and detection filtering)
- Matched pCRC and patient-derived xenograft (PDX) gene expression samples

## Outputs

- Four subtype-specific, cancer cell-intrinsic template gene sets (CMS1-CMS4) written to templates.CMS
- CMScaller R package template data structures (Biobase-based)

## How to apply

Start from pCRC subtype-vs-rest differential expression results and keep candidate genes with log2 fold change > 1 and adjusted p-value < 0.1. Apply the cell-line variation filter, retaining only genes in the top 25% inter-percentile range (IPR) detected as expressed in at least 3 CRC cell lines, to ensure measurable variation in pre-clinical models. Then apply the xenotransplantation filter: compute matched pCRC-PDX log2 fold-changes and discard candidates with LFC >= 2, deliberately without a p-value threshold, because such genes reflect microenvironment loss upon xenografting. Assemble the surviving genes into four subtype-specific template gene sets (CMS1-CMS4) and package them into templates.CMS using the CMScaller R package with Biobase data structures, so that Nearest Template Prediction (NTP) operates on cancer cell-intrinsic markers.

## Related tools

- **CMScaller** (R package that hosts the filtered subtype templates and performs Nearest Template Prediction classification of pre-clinical models) — https://github.com/Lothelab/CMScaller
- **R** (Implementation language for the differential-expression, IPR and pCRC-PDX filtering workflow)
- **Biobase** (Bioconductor data structures used to assemble and store the template gene sets)

## Examples

```
library(Biobase); library(CMScaller); res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- Surviving template genes must have pCRC-PDX log2 fold-change < 2 (no p-value threshold applied to this filter)
- Candidate genes entering the workflow must pass log2FC > 1 and adjusted p-value < 0.1 in pCRC subtype-vs-rest DE
- Cell-line filter must retain only genes in the top 25% inter-percentile range expressed in at least 3 cell lines
- Output must be four subtype-specific template sets (CMS1-CMS4) loadable via templates.CMS in the CMScaller package
- Reclassified pre-clinical models should recover CMS1-immune and CMS4-mesenchymal groups that the original classifier missed

## Limitations

- Input gene expression data must be centered and scaled; small datasets (fewer than ~40 samples) inescapably introduce prediction uncertainty
- CMScaller in its current implementation is not recommended for samples with a different human stromal component than primary CRCs, e.g. metastatic CRC biopsies
- Caution is warranted in highly selected datasets where molecular distributions deviate from pCRC cohorts, including metastatic samples
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries

## Evidence

- [abstract] original classifier misses stromal groups: "The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group"
- [abstract] CMScaller recapitulates subtypes in pre-clinical models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] not for metastatic biopsies: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs, including e.g. patient biopsies of metastatic CRC."
- [discussion] scaling and small-dataset uncertainty: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] NTP core algorithm: [evidence span withheld — anchor status: not_found]
