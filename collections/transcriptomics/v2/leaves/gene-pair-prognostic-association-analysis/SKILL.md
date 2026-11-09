---
name: gene-pair-prognostic-association-analysis
description: Use when bulk RNA-seq/microarray with matched survival data must be tested
  for whether immune cell types or immune marker genes appear spuriously favorable
  due to co-infiltration, as in TCGA SKCM06 melanoma.
when_to_use_negative:
- Input lacks matched survival information or event/time-to-event fields — TimiGP
  requires survival data to fit Cox models
- 'The cohort is too small: power is determined by sample size, number of events,
  and quality of survival information'
- The goal is cell-cell interaction inference or immunotherapy response prediction
  (TimiGP-Response modules) rather than quantifying prognostic bias of cell types/marker
  genes
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: TimiGP
  role: R package providing TimiCheckEvent, TimiPrePropress, and TimiCOX used for
    filtering, preprocessing, and Cox regression of immune markers
  repo: https://github.com/CSkylarL/TimiGP
- name: survival
  role: R package underlying the univariate Cox regression fitted for each cell type
    score and immune marker gene
- name: CIBERSORTx
  role: Deconvolution approach used with the LM22 signature to estimate per-sample
    immune cell relative abundance
- name: xCell
  role: Marker-expression approach used as an alternative cell-type abundance estimator
- name: devtools
  role: Used to install the TimiGP package from GitHub (devtools::install_github)
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:69cd4149752c163bc07bd48e6b83013d1b7dab863ed012065b47f12b0978f7e1
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
  - survival
  - xCell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# gene-pair-prognostic-association-analysis

## Summary

Quantifies prognostic bias caused by immune co-infiltration by fitting univariate Cox regressions (survival/TimiCOX, BH-adjusted) on immune cell type scores and immune marker genes in the TCGA SKCM06 metastatic melanoma cohort, then computing the fraction of significant associations (p<0.05) that are favorable (HR<1).

## When to use

Use when bulk RNA-seq/microarray with matched survival data must be tested for whether immune cell types or immune marker genes appear spuriously favorable due to co-infiltration, as in TCGA SKCM06 melanoma.

## When NOT to use

- Input lacks matched survival information or event/time-to-event fields — TimiGP requires survival data to fit Cox models
- The cohort is too small: power is determined by sample size, number of events, and quality of survival information
- The goal is cell-cell interaction inference or immunotherapy response prediction (TimiGP-Response modules) rather than quantifying prognostic bias of cell types/marker genes

## Inputs

- TCGA SKCM06 metastatic melanoma clinical information (SKCM06info) with event and time-to-event fields
- TCGA SKCM06 bulk transcriptome profile (SKCM06rna)
- CIBERSORT LM22 cell-type marker matrix (CellType_Newman2015_LM22)
- Immune marker gene sets (e.g., Immune_Marker_n1293 from the TimiGP package)

## Outputs

- Cell-type-level Cox regression table (HR, CI, p-value, BH-adjusted QV)
- Gene-level Cox regression table for immune marker genes
- Proportion of significant (p<0.05) associations with favorable HR<1 among cell types and marker genes
- Single combined results CSV of cell-type and gene Cox tables

## How to apply

Load the SKCM06 clinical info and transcriptome profile from the TimiGP package and filter low-quality survival records (event and time-to-event) with TimiCheckEvent. Preprocess the expression matrix for the LM22 immune markers with TimiPrePropress (log=TRUE, GMNorm=TRUE for gene-wise median normalization). Estimate per-sample immune cell relative abundance by deconvolving against the CIBERSORT LM22 signature (CellType_Newman2015_LM22) and via the xCell marker-expression approach. Fit univariate Cox regression on each cell type score and each immune marker gene (TimiCOX interface to the survival package, p.adj='BH'), remembering that in the TimiCOX convention reverse marker pairs with HR>1 indicate favorable prognosis (e.g., the CXCL10-over-PD-L1 direction). Finally compute the proportion of significant associations (p<0.05) with HR<1 among cell types and among marker genes, and write HR, CI, p-value, and BH-adjusted QV tables to a combined results CSV. Correct application reproduces the paper's bias statistics: 44 significant cell types with 93.2% favorable, and 90.5% of prognostic immune marker genes (including PD-L1 and IDO1) associated with improved survival.

## Related tools

- **TimiGP** (R package providing TimiCheckEvent, TimiPrePropress, and TimiCOX used for filtering, preprocessing, and Cox regression of immune markers) — https://github.com/CSkylarL/TimiGP
- **survival** (R package underlying the univariate Cox regression fitted for each cell type score and immune marker gene)
- **CIBERSORTx** (Deconvolution approach used with the LM22 signature to estimate per-sample immune cell relative abundance)
- **xCell** (Marker-expression approach used as an alternative cell-type abundance estimator)
- **devtools** (Used to install the TimiGP package from GitHub (devtools::install_github)) — https://github.com/r-lib/devtools

## Examples

```
Rscript -e 'library(TimiGP); info<-TimiCheckEvent(SKCM06info); rna<-TimiPrePropress(SKCM06rna, markers=Immune_Marker_n1293, log=TRUE, GMNorm=TRUE); cox<-TimiCOX(rna, info, p.adj="BH")'
```

## Evaluation signals

- 44 cell types are significantly associated with prognosis (p<0.05) in TCGA SKCM06, and 93.2% of them have HR<1 (favorable)
- 90.5% of prognostic immune marker genes — including checkpoints PD-L1 and IDO1 — show HR<1 (improved survival)
- TimiCheckEvent removes all records lacking valid event and time-to-event values before modeling
- TimiPrePropress output is log-transformed and gene-wise median normalized (log=TRUE, GMNorm=TRUE)
- Marker-pair directionality check: reverse pairs such as CXCL10-over-PD-L1 must show HR>1 under the TimiCOX convention

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- Power of the survival analysis is determined by sample size, number of events, and quality of survival information
- The bias estimate is a co-infiltration effect, so pro-tumor cells (e.g., via LM22 deconvolution) can appear prognostically favorable without implying causal benefit

## Evidence

- [readme] Dataset availability: [evidence span withheld — anchor status: external_doc]
