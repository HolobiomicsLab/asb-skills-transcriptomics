---
name: differential-expression-threshold-filtering
description: Use when building cancer cell-intrinsic subtype marker gene sets for
  pre-clinical model subtyping, i.e. when DE results from tumor cohorts must be stripped
  of stromal/immune signals absent from cell lines, organoids and PDXs.
when_to_use_negative:
- Samples with a different human stromal component than primary CRCs, e.g. patient
  biopsies of metastatic CRC
- Single-cell RNA-sequencing data, which is exceedingly noisy and not what the template
  gene set was optimized for
- Very small datasets (fewer than approximately forty samples), where required centering
  and scaling inescapably introduces prediction uncertainty
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CMScaller
  role: R package implementing the NTP-based classifier whose template gene set is
    built by these threshold filters; provides templates.CMS and prediction functions
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Runtime environment in which the DE threshold filtering workflow and CMScaller
    are executed
- name: Biobase
  role: Bioconductor data structures (ExpressionSet) used to hold expression matrices
    and template gene sets during filtering and prediction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:c8feecedf7142d5a4b1e4081cbc85133c5ebb5b63433cc8088f87ca5f33d3f62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# differential-expression-threshold-filtering

## Summary

Constructs cancer cell-intrinsic CMS subtype marker templates for CMScaller by successively filtering primary CRC subtype-vs-rest differential expression results with log2 fold-change and adjusted p-value cutoffs, an expression-variation filter on CRC cell lines, and a pCRC-vs-PDX xenotransplantation filter to remove stromal/immune-derived signals.

## When to use

Use when building cancer cell-intrinsic subtype marker gene sets for pre-clinical model subtyping, i.e. when DE results from tumor cohorts must be stripped of stromal/immune signals absent from cell lines, organoids and PDXs.

## When NOT to use

- Samples with a different human stromal component than primary CRCs, e.g. patient biopsies of metastatic CRC
- Single-cell RNA-sequencing data, which is exceedingly noisy and not what the template gene set was optimized for
- Very small datasets (fewer than approximately forty samples), where required centering and scaling inescapably introduces prediction uncertainty

## Inputs

- Primary CRC (pCRC) subtype-vs-rest differential expression results (log2 fold change, adjusted p-value per gene)
- CRC cell line gene expression matrix
- Matched pCRC and patient-derived xenograft (PDX) sample expression data
- Gene expression matrices of pre-clinical models (RNA-seq or microarray) for downstream CMScaller classification

## Outputs

- Four subtype-specific template gene sets (CMS1-CMS4) stored in templates.CMS
- CMScaller predictions per sample with confidence scores, plus optional plots (doPlot=TRUE)
- CMS-informative gene set analysis results (CMSgsa) and subtype differential expression results (subDEG)

## How to apply

Start from primary CRC (pCRC) subtype-vs-rest differential expression results and retain candidate genes with log2 fold change > 1 and adjusted p-value < 0.1. Apply the expression-variation filter on the CRC cell line expression matrix, keeping genes in the top 25% inter-percentile range (IPR) that are detected as expressed in at least 3 cell lines. Then apply the xenotransplantation filter, removing candidate genes whose expression changes with LFC >= 2 between matched pCRC and patient-derived xenograft (PDX) samples; the final criterion retains only genes not lost upon xenotransplantation (pCRC-PDX log2 fold-change < 2, no p-value threshold), so the template consists of cancer cell-intrinsic markers rather than microenvironment-derived signals. Assemble surviving genes into four subtype-specific template gene sets (CMS1-CMS4) and store them in templates.CMS using the CMScaller R package with Biobase data structures. Classification of new samples then proceeds via the Nearest Template Prediction (NTP) algorithm implemented in the CMScaller function.

## Related tools

- **CMScaller** (R package implementing the NTP-based classifier whose template gene set is built by these threshold filters; provides templates.CMS and prediction functions) — https://github.com/Lothelab/CMScaller
- **R** (Runtime environment in which the DE threshold filtering workflow and CMScaller are executed)
- **Biobase** (Bioconductor data structures (ExpressionSet) used to hold expression matrices and template gene sets during filtering and prediction)

## Examples

```
res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- All retained candidate genes from the DE step satisfy log2 fold change > 1 and adjusted p-value < 0.1
- Surviving template genes fall in the top 25% inter-percentile range (IPR) of the CRC cell line matrix and are detected as expressed in at least 3 cell lines
- No template gene shows pCRC-to-PDX log2 fold-change >= 2 (xenotransplantation filter passes, i.e. genes not lost upon xenotransplantation)
- Template gene sets are organized as four CMS1-CMS4 sets stored in templates.CMS
- The resulting classifier recapitulates CMS subtypes in the in vitro/in vivo model set (551 models in the published evaluation)

## Limitations

- CMScaller in its current implementation is not recommended for samples with a different human stromal component than primary CRCs, including metastatic CRC biopsies; caution is warranted in highly selected datasets where molecular distributions deviate from pCRC cohorts
- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Not optimized for single-cell RNA-sequencing data, which is exceedingly noisy; the prediction template gene set is planned to be improved with larger, higher-quality datasets
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries

## Evidence

- [abstract] The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group: "The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group"
- [abstract] CMScaller recapitulated subtypes in pre-clinical models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] Not recommended for metastatic CRC biopsies: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs, including e.g. patient biopsies of metastatic CRC."
- [discussion] Input must be centered and scaled; small datasets introduce uncertainty: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] Microenvironment-independent CMScaller classification of pre-clinical models: "CMScaller: microenvironment-independent colorectal cancer consensus molecular subtyping"
- [readme] Core algorithm is NTP by Hoshida: "The core algorithm is *[Nearest Template Prediction (NTP)](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0015543)* algorithm proposed by Yujin Hoshida"
