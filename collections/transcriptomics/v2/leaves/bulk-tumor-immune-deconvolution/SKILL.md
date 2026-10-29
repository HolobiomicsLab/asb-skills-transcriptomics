---
name: bulk-tumor-immune-deconvolution
description: Use when a bulk RNA-seq or microarray tumor cohort with matched survival
  data (event + time-to-event) should be scored for immune cell infiltration and linked
  to Cox-model hazard ratios to detect co-infiltration-driven prognostic bias.
when_to_use_negative:
- The cohort lacks matched survival (event + time-to-event) information — TimiGP requires
  a large gene expression dataset with matched survival data to identify prognostic
  associations
- Input is already single-cell RNA-seq resolved per cell type rather than a bulk profile
  needing deconvolution
- The goal is experimental validation of cell-cell interactions rather than bulk-level
  inference — the article notes no wet-lab validation was performed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2648
- http://edamontology.org/topic_2269
tools:
- name: TimiGP
  role: R package providing TimiCheckEvent (survival record filtering), TimiPrePropress
    (log transform + gene-wise median normalization), and TimiCOX (univariate Cox
    regression with BH adjustment) used to link immune cell abundance to prognosis
  repo: https://github.com/CSkylarL/TimiGP
- name: CIBERSORTx
  role: Deconvolution of bulk SKCM06 profiles against the CIBERSORT LM22 signature
    (CellType_Newman2015_LM22) to estimate per-sample immune cell relative abundance
- name: xCell
  role: Marker-expression-based alternative approach for estimating per-sample immune
    cell-type scores from the same bulk profiles
- name: survival
  role: R package underlying TimiCOX for fitting univariate Cox proportional hazards
    models on each cell-type score and marker gene
- name: R
  role: Execution environment; TimiGP is installed via devtools::install_github('CSkylarL/TimiGP')
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:7aac4217f1e105dbdd41d1ed3c0faa6b2435f7a3d01e15c7bfe05852957c8658
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2648
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bulk-tumor-immune-deconvolution

## Summary

A workflow for estimating per-sample relative abundance of immune cell types from bulk tumor transcriptomes and testing whether each cell type or immune marker gene is prognostically associated, in order to quantify systematic prognostic bias caused by immune co-infiltration (e.g., 93.2% of significant cell types in TCGA_SKCM06 showing HR < 1).

## When to use

Use when a bulk RNA-seq or microarray tumor cohort with matched survival data (event + time-to-event) should be scored for immune cell infiltration and linked to Cox-model hazard ratios to detect co-infiltration-driven prognostic bias.

## When NOT to use

- The cohort lacks matched survival (event + time-to-event) information — TimiGP requires a large gene expression dataset with matched survival data to identify prognostic associations
- Input is already single-cell RNA-seq resolved per cell type rather than a bulk profile needing deconvolution
- The goal is experimental validation of cell-cell interactions rather than bulk-level inference — the article notes no wet-lab validation was performed

## Inputs

- Bulk tumor transcriptome profile (e.g., TCGA SKCM06 SKCM06rna RNA-seq expression matrix)
- Matched clinical information with event and time-to-event fields (e.g., SKCM06info)
- Cell-type marker matrix, e.g., CIBERSORT LM22 signature (CellType_Newman2015_LM22) or xCell marker sets
- Immune marker gene list (e.g., Immune_Marker_n1293, 1293 immune cell markers)

## Outputs

- Per-sample relative abundance scores for immune cell types (CIBERSORTx and xCell)
- Cell-type-level Cox regression table (HR, confidence interval, p-value, BH-adjusted QV)
- Gene-level Cox regression table (HR, CI, p-value, QV) for immune marker genes
- Proportion of significant (p<0.05) associations with favorable HR (HR<1) among cell types and among marker genes
- Single combined results CSV of cell-type and gene Cox tables

## How to apply

First filter low-quality survival records with TimiCheckEvent, then extract the LM22 immune marker expression profile with TimiPrePropress (log=TRUE plus GMNorm gene-wise median normalization). Estimate per-sample cell-type abundance by deconvolving the SKCM06 bulk profiles against the CIBERSORT LM22 signature (CellType_Newman2015_LM22) with CIBERSORTx and in parallel with the xCell marker-expression approach. Fit univariate Cox regressions on each cell-type score and each immune marker gene via TimiCOX with Benjamini-Hochberg (p.adj='BH') multiple-testing adjustment, treating reverse marker pairs with HR>1 as the unfavorable direction per the TimiCOX convention. Finally, compute the proportion of significant (p<0.05) associations with favorable HRs among cell types and among marker genes (e.g., PD-L1, IDO1) to quantify the co-infiltration prognostic bias, and export a combined CSV of HR, CI, p-value and BH-adjusted QV.

## Related tools

- **TimiGP** (R package providing TimiCheckEvent (survival record filtering), TimiPrePropress (log transform + gene-wise median normalization), and TimiCOX (univariate Cox regression with BH adjustment) used to link immune cell abundance to prognosis) — https://github.com/CSkylarL/TimiGP
- **CIBERSORTx** (Deconvolution of bulk SKCM06 profiles against the CIBERSORT LM22 signature (CellType_Newman2015_LM22) to estimate per-sample immune cell relative abundance)
- **xCell** (Marker-expression-based alternative approach for estimating per-sample immune cell-type scores from the same bulk profiles)
- **survival** (R package underlying TimiCOX for fitting univariate Cox proportional hazards models on each cell-type score and marker gene)
- **R** (Execution environment; TimiGP is installed via devtools::install_github('CSkylarL/TimiGP')) — https://github.com/r-lib/devtools

## Examples

```
data(CellType_Newman2015_LM22); rna <- TimiPrePropress(SKCM06rna, marker = names(CellType_Newman2015_LM22), log = TRUE, GMNorm = TRUE); cox <- TimiCOX(rna, info, p.adj = 'BH')
```

## Evaluation signals

- TimiCheckEvent removes all records lacking valid event and time-to-event before any modeling; downstream sample counts must reflect this filtering
- After TimiPrePropress with log=TRUE and GMNorm=TRUE, only the LM22-selected marker genes remain and their distributions are gene-wise median normalized
- In TCGA_SKCM06, ~44 cell types significant at p<0.05 with ~93.2% showing HR<1, including known pro-tumor cells, and ~90.5% of prognostic immune marker genes (including PD-L1 and IDO1) associated with improved survival — reproduce this favorable-prognosis dominance
- Marker-pair directionality follows the TimiCOX convention: reverse pairs carry HR>1 as unfavorable; the CXCL10-over-PD-L1 pair's prognostic direction should match this rule
- Exported CSV contains HR, CI, p-value, and BH-adjusted QV for both cell-type-level and gene-level entries, with QV from Benjamini-Hochberg (p.adj='BH')

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information; statistical power depends on sample size, number of events, and survival data quality
- The article notes no wet-lab experiments were performed to validate the important findings, raising uncertainty about the reliability of cell-cell interaction inference
- The inverse association found here reflects co-infiltration prognostic bias, not true causal protective roles for pro-tumor cells or checkpoint genes like PD-L1 and IDO1

## Evidence

- [methods] Preprocess with marker filtering and normalization: "`TimiPrePropress` will extract the transcription profile of selected markers and cohorts, then perform log transformation and gene-wise median normalization."
- [methods] Cox regression step: "`TimiCOX` will perform univariate Cox regression that fits each marker pair as a variable."
- [methods] Survival record quality filtering: "`TimiCheckEvent` will filter out the low-quality clinical info(event and time-to-event)."
- [readme] Rationale for TIME prognostic direction: "The tumor immune microenvironment(TIME) is a balance between anti-tumor and pro-tumor immune cells."
- [discussion] Power depends on survival data quality: "The power of the survival analysis is determined by the sample size, the number of events, and the quality of the survival information."
