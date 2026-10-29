---
name: tumor-immune-microenvironment-cell-interaction-inference
description: Use when you have bulk RNA-seq or microarray data with matched overall
  survival (or immunotherapy response) information and cell-type marker lists, and
  you need to infer which immune cell-cell interactions drive favorable/unfavorable
  prognosis.
when_to_use_negative:
- The cohort lacks matched survival information or has too few samples/events — TimiGP
  requires a large gene expression dataset with matched survival information to identify
  prognostic gene pairs
- The goal is deconvolving absolute cell-type proportions from bulk data (deconvolution
  tools like CIBERSORTx or xCell are the appropriate class)
- The analysis requires experimentally validated interactions — the paper notes no
  wet-lab validation was performed, so inferred interactions carry uncertainty
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: TimiGP
  role: 'Core R package implementing the full pipeline: TimiCheckEvent, TimiPrePropress,
    TimiGenePair, TimiCOX, TimiBG, TimiCellPair, TimiEnrich, TimiPermFDR, TimiCellNetwork,
    TimiFS'
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Runtime language; TimiGP installed via devtools from GitHub
- name: devtools
  role: Used to install the TimiGP R package from GitHub (devtools::install_github)
  repo: https://github.com/r-lib/devtools
- name: MSofTimiGP
  role: Companion repository with codes and data reproducing manuscript figures and
    cross-cohort analyses
  repo: https://github.com/CSkylarL/MSofTimiGP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:b0b97e35375836baf8b7a8e63dff400f5fb4063e0227910e178fc493d2c703d2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - MSofTimiGP
  - R
  - TimiGP
  - devtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# tumor-immune-microenvironment-cell-interaction-inference

## Summary

A workflow, implemented in the TimiGP R package, that infers directed cell-cell interactions and prognostic associations in the tumor immune microenvironment from bulk transcriptomics plus survival data using marker gene pairs, Cox regression, and enrichment analysis.

## When to use

Use when you have bulk RNA-seq or microarray data with matched overall survival (or immunotherapy response) information and cell-type marker lists, and you need to infer which immune cell-cell interactions drive favorable/unfavorable prognosis.

## When NOT to use

- The cohort lacks matched survival information or has too few samples/events — TimiGP requires a large gene expression dataset with matched survival information to identify prognostic gene pairs
- The goal is deconvolving absolute cell-type proportions from bulk data (deconvolution tools like CIBERSORTx or xCell are the appropriate class)
- The analysis requires experimentally validated interactions — the paper notes no wet-lab validation was performed, so inferred interactions carry uncertainty

## Inputs

- Bulk transcriptomic profile (RNA-seq or microarray) of a cohort with survival data, e.g., TCGA SKCM06 metastatic melanoma clinical info and expression
- Cell-type marker sets (scRNA-seq-derived or prior knowledge, e.g., Immune_Marker_n1293, CellType_Bindea2013_cancer, CellType_Newman2015_LM22)
- Clinical event/time-to-event information filtered via TimiCheckEvent

## Outputs

- Directed cell-cell interaction network (TimiCellNetwork export, Cytoscape-importable files)
- Enrichment results of prognostic marker pairs on cell-pair annotations with adjusted p-values and TimiPermFDR FDR estimates
- Per-cell-type favorable/unfavorable favorability scores (TimiFS)
- Marker pair score (MPS) matrix and Cox regression results (HR, p-values, BH-adjusted QV)

## How to apply

Preprocess the cohort with TimiCheckEvent to drop low-quality event/time-to-event clinical info, and TimiPrePropress to log-transform and gene-wise median-normalize marker expression. Generate the binary Marker Pair Score matrix with TimiGenePair (A>B=1, A<B=0), then run TimiCOX univariate Cox regression with Benjamini-Hochberg adjustment (p.adj='BH'), reversing pairs with HR>1 so all retained pairs associate with favorable prognosis. Build the enrichment background with TimiBG and cell interaction annotations with TimiCellPair, run TimiEnrich (p.adj='BH') on marker pairs with QV<0.05, and estimate FDR with TimiPermFDR (niter=100). Export the directed cell-cell network with TimiCellNetwork (cutoff 0.05 on Adjust.P.Value) and compute per-cell favorable/unfavorable scores with TimiFS; robustness can be checked by re-running with alternative marker sets (e.g., CellType_Charoentong2017_Bindea2013_Xu2018_Immune vs CellType_Newman2015_LM22) and testing network similarity across cohorts.

## Related tools

- **TimiGP** (Core R package implementing the full pipeline: TimiCheckEvent, TimiPrePropress, TimiGenePair, TimiCOX, TimiBG, TimiCellPair, TimiEnrich, TimiPermFDR, TimiCellNetwork, TimiFS) — https://github.com/CSkylarL/TimiGP
- **R** (Runtime language; TimiGP installed via devtools from GitHub)
- **devtools** (Used to install the TimiGP R package from GitHub (devtools::install_github)) — https://github.com/r-lib/devtools
- **MSofTimiGP** (Companion repository with codes and data reproducing manuscript figures and cross-cohort analyses) — https://github.com/CSkylarL/MSofTimiGP

## Examples

```
devtools::install_github("CSkylarL/TimiGP")
```

## Evaluation signals

- The cytotoxic cell → tumor cell control interaction is significant (BH-adjusted p < 0.05) in all eight metastatic melanoma datasets
- Cross-dataset cell-cell interaction networks show significant similarity (p < 0.0002)
- Marker pairs retained after TimiCOX all associate with favorable prognosis (pairs with HR>1 reversed; QV < 0.05)
- Favorable activated T cell / NK cell versus unfavorable neutrophil favorability pattern (TimiFS) recurs across cohorts and alternative marker sets
- Results are robust to the choice of marker annotation (CellType_Charoentong2017_Bindea2013_Xu2018_Immune vs CellType_Newman2015_LM22 give highly consistent networks)

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information; power depends on sample size, number of events, and quality of survival information
- Because interactions are identified via enrichment analysis, sensitivity to a specific interaction is affected by the number of corresponding marker genes; cell types with fewer marker genes are less likely to be identified
- No wet-lab experiments were performed to validate the findings, raising uncertainty about the reliability of inferred cell-cell interactions

## Evidence
