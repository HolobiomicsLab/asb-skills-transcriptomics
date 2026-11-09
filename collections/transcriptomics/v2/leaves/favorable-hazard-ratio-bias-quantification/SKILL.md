---
name: favorable-hazard-ratio-bias-quantification
description: Use when bulk transcriptomic profiles with matched survival data are
  available and you need to test whether immune co-infiltration makes most cell types
  and immune marker genes appear prognostically favorable regardless of their anti-/pro-tumor
  function.
when_to_use_negative:
- The cohort lacks matched survival information or has too few events — TimiGP requires
  a large expression dataset with matched survival data to identify prognostic gene
  pairs
- The goal is cell-type deconvolution alone without survival endpoints, since no Cox
  associations can be fit
- Experimental validation of a specific cell-cell interaction is required — the inference
  is enrichment-analysis based and no wet-lab validation exists
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2640
tools:
- name: TimiGP
  role: R package providing TimiCheckEvent, TimiPrePropress, and TimiCOX used to filter
    records, normalize marker expression, and fit univariate Cox regressions on TCGA
    SKCM06
  repo: https://github.com/CSkylarL/TimiGP
- name: CIBERSORTx
  role: Deconvolution of bulk SKCM06 profiles against the LM22 signature (CellType_Newman2015_LM22)
    to estimate per-sample immune cell relative abundance
- name: xCell
  role: Marker-expression-based estimation of immune cell-type scores as a second
    deconvolution approach
- name: survival
  role: R package underlying the univariate Cox regression fits for each cell-type
    score and immune marker gene
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:fdee745fad153a1df8c4de5a06505ec63673c1a26fc5bd7dd0f11fb2cad277d0
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
  - survival
  - xCell
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Favorable hazard-ratio bias quantification

## Summary

Quantifies systematic prognostic bias in the tumor immune microenvironment by fitting univariate Cox regressions (TimiCOX, survival package, BH adjustment) on immune cell-type scores and marker genes in TCGA SKCM06, then computing the fraction of significant (p < 0.05) associations with HR < 1, including checkpoint genes like PD-L1 and IDO1.

## When to use

Use when bulk transcriptomic profiles with matched survival data are available and you need to test whether immune co-infiltration makes most cell types and immune marker genes appear prognostically favorable regardless of their anti-/pro-tumor function.

## When NOT to use

- The cohort lacks matched survival information or has too few events — TimiGP requires a large expression dataset with matched survival data to identify prognostic gene pairs
- The goal is cell-type deconvolution alone without survival endpoints, since no Cox associations can be fit
- Experimental validation of a specific cell-cell interaction is required — the inference is enrichment-analysis based and no wet-lab validation exists

## Inputs

- TCGA SKCM06 clinical info (SKCM06info: event and time-to-event survival records)
- TCGA SKCM06 bulk transcriptome profile (SKCM06rna, RNA-seq expression matrix)
- CIBERSORT LM22 cell-type marker matrix (CellType_Newman2015_LM22)
- Immune marker gene set (Immune_Marker_n1293)

## Outputs

- Cell-type-level Cox regression table (HR, confidence interval, p-value, BH-adjusted QV)
- Gene-level Cox regression table for immune marker genes (HR, CI, p-value, BH-adjusted QV)
- Proportion of significant associations with favorable hazard ratios among cell types and marker genes
- Single combined CSV of cell-type and gene-level Cox results

## How to apply

Load the SKCM06 clinical info and transcriptome profile from the TimiGP package and filter low-quality survival records (event and time-to-event) with TimiCheckEvent. Preprocess the expression matrix for LM22 immune markers with TimiPrePropress using log=TRUE and GMNorm=TRUE (log transformation plus gene-wise median normalization). Estimate per-sample relative abundance of the 22 LM22 cell types by deconvolution against CellType_Newman2015_LM22 (CIBERSORT-style) and via the xCell marker-expression approach. Fit a univariate Cox model on each cell-type score and each of the 1293 immune marker genes via TimiCOX with p.adj='BH', then compute the proportion of significant (p < 0.05) associations with HR < 1 among cell types and among marker genes, and check the CXCL10-over-PD-L1 marker-pair direction (reverse pairs with HR > 1 per TimiCOX convention). The expected bias signature is that even established pro-tumor cells and checkpoints appear favorable: 93.2% of significant cell types and 90.5% of prognostic marker genes with HR < 1.

## Related tools

- **TimiGP** (R package providing TimiCheckEvent, TimiPrePropress, and TimiCOX used to filter records, normalize marker expression, and fit univariate Cox regressions on TCGA SKCM06) — https://github.com/CSkylarL/TimiGP
- **CIBERSORTx** (Deconvolution of bulk SKCM06 profiles against the LM22 signature (CellType_Newman2015_LM22) to estimate per-sample immune cell relative abundance)
- **xCell** (Marker-expression-based estimation of immune cell-type scores as a second deconvolution approach)
- **survival** (R package underlying the univariate Cox regression fits for each cell-type score and immune marker gene)

## Examples

```
res <- TimiCOX(expr = SKCM06rna_norm, info = SKCM06info, p.adj = "BH") # univariate Cox per cell type / marker gene, BH-adjusted QV
```

## Evaluation signals

- 44 cell types significant at p < 0.05, with ~93.2% of significant cell types showing HR < 1 — including known pro-tumor cells
- ~90.5% of prognostic immune marker genes associated with improved survival, including checkpoints PD-L1 and IDO1
- Every Cox table row carries HR, confidence interval, raw p-value, and BH-adjusted QV column
- CXCL10-over-PD-L1 reverse marker pair shows HR > 1 per the TimiCOX convention on prognostic direction
- Filtered cohort after TimiCheckEvent retains only records with valid event and time-to-event fields

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- Power of the survival analysis is determined by sample size, number of events, and quality of the survival information
- No wet-lab experiments were performed to validate the important findings, raising uncertainty about the reliability of cell-cell interaction inference

## Evidence
