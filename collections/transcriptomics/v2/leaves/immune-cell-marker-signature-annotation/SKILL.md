---
name: immune-cell-marker-signature-annotation
description: Use when bulk RNA-seq or microarray tumor profiles need per-sample immune
  cell-type abundance estimates from a published marker signature (LM22 or similar),
  e.g. to link co-infiltration to survival outcomes.
when_to_use_negative:
- Input is already single-cell RNA-seq resolved to individual cells — cell-type deconvolution
  of bulk profiles is unnecessary; use scRNA-seq-derived markers for high-resolution
  annotation instead
- No suitable marker signature exists for the tissue or cancer type in question —
  TimiGP notes that cell types with fewer marker genes are less likely to be identified
  reliably
- 'The cohort lacks sufficient samples/events: TimiGP requires a large gene expression
  dataset with matched survival information to yield powered downstream associations'
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: TimiGP
  role: R package providing marker sets (CellType_Newman2015_LM22, Immune_Marker_n1293)
    and preprocessing/analysis functions for cell-type annotation and prognostic association
  repo: https://github.com/CSkylarL/TimiGP
- name: CIBERSORTx
  role: Deconvolution of bulk SKCM06 profiles against the LM22 signature to estimate
    per-sample immune cell relative abundance
- name: xCell
  role: Marker-expression-based alternative for estimating immune cell-type abundance
    from bulk profiles
- name: TimiPrePropress
  role: Extracts the transcription profile of selected markers and performs log transformation
    and gene-wise median normalization before deconvolution
  repo: https://github.com/CSkylarL/TimiGP
- name: devtools
  role: Installs the TimiGP package from GitHub
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:054d8207318d31032b2d346e19bd5f0a2301407fe19cbbb58e2f76158e898f0c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - CIBERSORTx
  - R
  - TimiCOX
  - TimiCheckEvent
  - TimiGP
  - TimiPrePropress
  - devtools
  - xCell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# immune-cell-marker-signature-annotation

## Summary

Annotate bulk tumor transcriptomes with immune cell identity using curated marker signatures (e.g., the CIBERSORT LM22 signature in TimiGP), estimating per-sample relative abundance via deconvolution (CIBERSORTx) or marker-expression approaches (xCell) to support downstream prognostic modeling.

## When to use

Use when bulk RNA-seq or microarray tumor profiles need per-sample immune cell-type abundance estimates from a published marker signature (LM22 or similar), e.g. to link co-infiltration to survival outcomes.

## When NOT to use

- Input is already single-cell RNA-seq resolved to individual cells — cell-type deconvolution of bulk profiles is unnecessary; use scRNA-seq-derived markers for high-resolution annotation instead
- No suitable marker signature exists for the tissue or cancer type in question — TimiGP notes that cell types with fewer marker genes are less likely to be identified reliably
- The cohort lacks sufficient samples/events: TimiGP requires a large gene expression dataset with matched survival information to yield powered downstream associations

## Inputs

- Bulk transcriptomic profile (RNA-seq or microarray), e.g. TCGA SKCM06rna
- Curated immune cell-type marker matrix, e.g. CellType_Newman2015_LM22 or Immune_Marker_n1293 from the TimiGP package
- Optional matched clinical info table (event, time-to-event) for downstream survival association, e.g. SKCM06info

## Outputs

- Per-sample relative abundance / score matrix for immune cell types
- Log-transformed, gene-wise median-normalized marker expression matrix from TimiPrePropress
- Cell-type-level Cox regression table (HR, CI, p-value, BH-adjusted QV) when combined with survival data

## How to apply

First select a curated cell-type marker set appropriate to the tissue — TimiGP ships CellType_Newman2015_LM22 (LM22 signature from CIBERSORT), CellType_Charoentong2017_Bindea2013_Xu2018_Immune, and Immune_Marker_n1293 (1293 immune cell markers). Preprocess the bulk expression matrix with TimiPrePropress using log=TRUE and GMNorm=TRUE to extract the marker genes and apply log transformation plus gene-wise median normalization, which makes marker expression comparable across genes and samples. Estimate relative abundance of each cell type by deconvolving the profiles against the LM22 signature with CIBERSORTx, or with the xCell marker-expression approach; TimiGP shows both marker-expression- and deconvolution-based strategies are acceptable inputs. The resulting per-sample cell-type score matrix feeds univariate Cox regression (TimiCOX, BH p.adj) or gene-pair network construction (TimiGenePair) in the TCGA SKCM06 melanoma example.

## Related tools

- **TimiGP** (R package providing marker sets (CellType_Newman2015_LM22, Immune_Marker_n1293) and preprocessing/analysis functions for cell-type annotation and prognostic association) — https://github.com/CSkylarL/TimiGP
- **CIBERSORTx** (Deconvolution of bulk SKCM06 profiles against the LM22 signature to estimate per-sample immune cell relative abundance)
- **xCell** (Marker-expression-based alternative for estimating immune cell-type abundance from bulk profiles)
- **TimiPrePropress** (Extracts the transcription profile of selected markers and performs log transformation and gene-wise median normalization before deconvolution) — https://github.com/CSkylarL/TimiGP
- **devtools** (Installs the TimiGP package from GitHub) — https://github.com/r-lib/devtools

## Examples

```
devtools::install_github("CSkylarL/TimiGP"); data(CellType_Newman2015_LM22); expr <- TimiPrePropress(rna = SKCM06rna, geneset = CellType_Newman2015_LM22, log = TRUE, GMNorm = TRUE)
```

## Evaluation signals

- The preprocessed expression matrix contains only the marker genes of the chosen signature (LM22) and is log-transformed with gene-wise median normalization applied
- Per-sample abundance estimates cover the expected cell types of the signature (e.g., 22 LM22 immune cell types) with plausible relative-abundance ranges
- Downstream Cox results should reproduce the reported bias pattern: 44 cell types significant at p < 0.05 in SKCM06, with 93.2% HR < 1 (favorable)
- Results should be robust when a different marker set (e.g., Charoentong2017/Bindea2013/Xu2018 vs LM22) is used, since TimiGP reports highly consistent results across marker-set inputs
- Filtering with TimiCheckEvent removes low-quality survival records before any Cox model is fit

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic associations
- Power of the survival analysis is determined by sample size, number of events, and quality of survival information
- Cell types with fewer marker genes are less likely to be identified as having significant interactions, since identification is enrichment-analysis based
- No wet-lab experiments were performed to validate the important findings, raising uncertainty about inferred cell-cell interactions

## Evidence
