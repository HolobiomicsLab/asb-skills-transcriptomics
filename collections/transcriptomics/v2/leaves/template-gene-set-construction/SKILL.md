---
name: template-gene-set-construction
description: Use when constructing a nearest-template-prediction gene set for classifying
  pre-clinical models (cell lines, organoids, PDXs) and you must purge markers driven
  by immune/stromal tumor compartments.
when_to_use_negative:
- The input samples have a different human stromal component than primary CRCs, e.g.
  patient biopsies of metastatic CRC — CMScaller is explicitly not recommended there.
- The goal is single-cell RNA-sequencing classification, which the authors note is
  exceedingly noisy and not what this template was optimized for.
- You already have a validated, cancer cell-intrinsic template and only need to run
  prediction (use CMScaller directly rather than rebuilding the template).
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: CMScaller
  role: R package in which the filtered template gene sets are assembled and used
    for Nearest Template Prediction CMS classification of pre-clinical models
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Implementation environment for the differential expression, IPR, and xenotransplantation
    filtering steps and template assembly
- name: Biobase
  role: Bioconductor data structures (ExpressionSet) used to handle expression matrices
    during filtering and template construction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:4512fee86a3e099fed6a94e4e902ec3da95d93bf51af9ef98f936fbc849cc5ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# template-gene-set-construction

## Summary

Builds cancer cell-intrinsic subtype marker templates (e.g. CMScaller's CMS1-CMS4 gene sets) by successively filtering primary-CRC differential expression candidates through expression-variation and xenotransplantation (pCRC-vs-PDX) filters so the template reflects cancer-cell rather than stromal/immune signals.

## When to use

Use when constructing a nearest-template-prediction gene set for classifying pre-clinical models (cell lines, organoids, PDXs) and you must purge markers driven by immune/stromal tumor compartments.

## When NOT to use

- The input samples have a different human stromal component than primary CRCs, e.g. patient biopsies of metastatic CRC — CMScaller is explicitly not recommended there.
- The goal is single-cell RNA-sequencing classification, which the authors note is exceedingly noisy and not what this template was optimized for.
- You already have a validated, cancer cell-intrinsic template and only need to run prediction (use CMScaller directly rather than rebuilding the template).

## Inputs

- Primary CRC (pCRC) subtype-vs-rest differential expression results table (gene, log2 fold change, adjusted p-value)
- CRC cell line gene expression matrix (for inter-percentile range variation and detection filtering)
- Matched pCRC and patient-derived xenograft (PDX) expression data for xenotransplantation filtering

## Outputs

- Four subtype-specific template gene sets (CMS1-CMS4) stored as the CMScaller prediction template (templates.CMS)
- Cancer cell-intrinsic marker gene lists suitable for Nearest Template Prediction classification

## How to apply

Start from primary CRC (pCRC) subtype-vs-rest differential expression results and retain candidate genes with log2 fold change > 1 and adjusted p-value < 0.1. Next, using a CRC cell line expression matrix, keep only genes in the top 25% inter-percentile range (IPR) that are detected as expressed in at least 3 cell lines. Then apply the xenotransplantation filter: remove candidates whose expression changes with log2 fold-change >= 2 between matched pCRC and patient-derived xenograft (PDX) samples (no p-value threshold), so that microenvironment-derived signals absent in pre-clinical models are excluded. Assemble the surviving genes into four subtype-specific template gene sets (CMS1-CMS4) and store them as the prediction template used by CMScaller's Nearest Template Prediction algorithm, leveraging Biobase/Bioconductor data structures. Correctness is judged by verifying each filter's thresholds were applied in order and that the final template classifies 551 in vitro/in vivo models consistent with expected CMS proportions.

## Related tools

- **CMScaller** (R package in which the filtered template gene sets are assembled and used for Nearest Template Prediction CMS classification of pre-clinical models) — https://github.com/Lothelab/CMScaller
- **R** (Implementation environment for the differential expression, IPR, and xenotransplantation filtering steps and template assembly)
- **Biobase** (Bioconductor data structures (ExpressionSet) used to handle expression matrices during filtering and template construction)

## Examples

```
library(Biobase); library(CMScaller); res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- Candidate list after step 1 contains only genes with log2FC > 1 and adjusted p-value < 0.1 from pCRC subtype-vs-rest comparisons.
- Expression-variation filter retains only genes in the top 25% inter-percentile range detected in at least 3 cell lines.
- No retained gene shows a pCRC-to-PDX log2 fold-change >= 2 (xenotransplantation filter passed with no p-value threshold).
- Final template contains exactly four subtype-specific gene sets (CMS1-CMS4) formatted as templates.CMS within the CMScaller package.
- Template recapitulates CMS subtypes across the 551 in vitro and in vivo models, including the CMS1-immune and CMS4-mesenchymal groups that the stromal-dependent classifier misses.

## Limitations

- CMScaller is not recommended for samples with a different human stromal component than primary CRCs, including metastatic CRC biopsies.
- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples).
- Caution is warranted in highly selected datasets where molecular distributions deviate from pCRC cohorts; the template is planned to be improved with larger, higher-quality datasets.

## Evidence

- [abstract] Original classifier dependence on stroma: "The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group"
- [abstract] Cancer cell-intrinsic markers recapitulate subtypes: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] Metastatic CRC caution: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs, including e.g. patient biopsies of metastatic CRC."
- [discussion] Small-sample scaling uncertainty: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] NTP core algorithm and install path: "The core algorithm is *[Nearest Template Prediction (NTP)](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0015543)* algorithm proposed by Yujin Hoshida"
- [readme] Package build environment: [evidence span withheld — anchor status: external_doc]
