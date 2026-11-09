---
name: marker-gene-pair-cox-survival-regression
description: Use when you have bulk transcriptomic profiles (RNA-seq or microarray)
  with matched survival information and a cell-type marker set, and need to identify
  prognostic marker gene pairs as the basis for inferring cell-cell interactions in
  the tumor immune microenvironment.
when_to_use_negative:
- The dataset is small or lacks matched survival information — TimiGP requires a large
  gene expression dataset with matched survival information to effectively identify
  prognostic gene pairs
- The clinical survival information is low quality (few events or poor time-to-event
  data), since the power of the survival analysis is determined by sample size, number
  of events, and quality of survival information
- The goal is immunotherapy response rather than prognosis — use the TimiGP-Response
  module (e.g. TimiFisher) instead of the Cox-based Prognosis Module
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2229
tools:
- name: TimiGP
  role: R package providing TimiCheckEvent, TimiPrePropress, TimiGenePair, TimiCOX,
    TimiBG, TimiCellPair, TimiEnrich, and TimiPermFDR for the marker-pair Cox survival
    pipeline
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Runtime environment; TimiGP is installed from GitHub via devtools in R
  repo: https://github.com/r-lib/devtools
- name: devtools
  role: Used to install the TimiGP R package from GitHub (devtools::install_github)
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:6f431a413fb5a77413c6ec73858cbcce899b431cc0b8d62be694448e74d8ba58
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - devtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# marker-gene-pair-cox-survival-regression

## Summary

A TimiGP workflow step that converts cell-type marker expression into a binary Marker Pair Score (MPS) matrix, fits univariate Cox regression on each marker pair against survival data, and selects directed pairs associated with favorable prognosis for downstream cell-cell interaction network inference.

## When to use

Use when you have bulk transcriptomic profiles (RNA-seq or microarray) with matched survival information and a cell-type marker set, and need to identify prognostic marker gene pairs as the basis for inferring cell-cell interactions in the tumor immune microenvironment.

## When NOT to use

- The dataset is small or lacks matched survival information — TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- The clinical survival information is low quality (few events or poor time-to-event data), since the power of the survival analysis is determined by sample size, number of events, and quality of survival information
- The goal is immunotherapy response rather than prognosis — use the TimiGP-Response module (e.g. TimiFisher) instead of the Cox-based Prognosis Module

## Inputs

- Bulk transcriptomic profile (RNA-seq or microarray expression matrix, e.g. SKCM06rna from TCGA SKCM06 metastatic melanoma)
- Clinical information with event and time-to-event fields (e.g. SKCM06info)
- Cell-type marker gene list (e.g. Immune_Marker_n1293, CellType_Charoentong2017_Bindea2013_Xu2018_Immune, CellType_Newman2015_LM22)

## Outputs

- Marker Pair Score (MPS) matrix of binary pairwise comparisons (A>B=1, A<B=0)
- Cox regression results per marker pair with hazard ratios and BH-adjusted p-values
- Set of directed marker gene pairs (IMGP) associated with favorable prognosis
- Cell-cell interaction network with per-cell favorable/unfavorable favorability scores (via TimiCellNetwork and TimiFS)

## How to apply

First preprocess the cohort with TimiCheckEvent (which filters out low-quality event and time-to-event clinical info) and TimiPrePropress (which extracts the marker expression profile and performs log transformation and gene-wise median normalization). Then run TimiGenePair to generate a binary Marker Pair Score matrix where each pair is scored A>B=1, A<B=0, capturing the logical relation of any two markers. Fit univariate Cox regression per marker pair using TimiCOX with Benjamini-Hochberg multiple-testing adjustment (p.adj='BH'). For each retained pair, reverse the direction when the hazard ratio is greater than 1 so that all retained directed pairs associate with favorable prognosis; this directionality convention encodes the TimiGP rationale that anti-tumor cell dominance corresponds to favorable outcome. The resulting directed marker gene pairs (IMGPs) are then used with TimiBG, TimiCellPair, and TimiEnrichment (QV<0.05, then TimiPermFDR with niter=100) to identify cell-cell interactions.

## Related tools

- **TimiGP** (R package providing TimiCheckEvent, TimiPrePropress, TimiGenePair, TimiCOX, TimiBG, TimiCellPair, TimiEnrich, and TimiPermFDR for the marker-pair Cox survival pipeline) — https://github.com/CSkylarL/TimiGP
- **R** (Runtime environment; TimiGP is installed from GitHub via devtools in R) — https://github.com/r-lib/devtools
- **devtools** (Used to install the TimiGP R package from GitHub (devtools::install_github)) — https://github.com/r-lib/devtools

## Examples

```
devtools::install_github("CSkylarL/TimiGP"); mps <- TimiGenePair(rna, markers); res <- TimiCOX(mps, info, p.adj = "BH")
```

## Evaluation signals

- All retained directed marker pairs have hazard ratio <= 1 after reversal, i.e. every kept pair associates with favorable prognosis
- Cell-cell interactions derived from the Cox-selected pairs reach significance at BH-adjusted p (QV) < 0.05, confirmed with TimiPermFDR (niter=100) FDR estimation
- In metastatic melanoma validation, the cytotoxic cells → tumor cells control interaction is significant (adjusted p < 0.05) across all eight datasets, and cross-dataset network similarity reaches p < 0.0002
- Preprocessing invariants hold: marker expression is log-transformed and gene-wise median normalized; low-quality event/time-to-event records were removed by TimiCheckEvent before Cox fitting
- Favorability scores (TimiFS) reproduce the expected pattern: activated T cell / NK cell cell types score favorable and neutrophils unfavorable

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- The power of the survival analysis is determined by the sample size, the number of events, and the quality of the survival information
- Because cell-cell interactions are identified via enrichment analysis, sensitivity to a specific interaction depends on the number of corresponding marker genes; cell types with fewer marker genes are less likely to be identified
- No wet-lab experiments were performed to validate the important findings, raising uncertainty about the reliability of the inferred cell-cell interactions

## Evidence
