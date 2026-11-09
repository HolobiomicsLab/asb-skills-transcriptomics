---
name: cell-cell-interaction-network-degree-analysis
description: Use when bulk transcriptome + survival data have been reduced to prognostic
  immune marker gene pairs (IMGPs) and you need to rank cell types by favorable vs
  unfavorable prognostic associations via network degree.
when_to_use_negative:
- Input lacks matched survival information or has low-quality event/time-to-event
  data — TimiGP requires large datasets with matched survival to identify prognostic
  gene pairs
- The goal is deconvolving absolute cell-type abundances from bulk data (e.g., CIBERSORTx/xCell-style
  estimation) rather than prognostic cell-cell interaction ranking
- Cell types of interest have very few marker genes — enrichment-based sensitivity
  is low and interactions are unlikely to be detected
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: TimiGP
  role: R package providing TimiCellPair, TimiBG, TimiEnrich, TimiPermFDR, TimiCellNetwork,
    and TimiFS to build the interaction network and degree-based favorability scores
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Execution environment for the TimiGP workflow and aggregation of per-cohort
    scores into pancancer_favorability_scores.csv
- name: devtools
  role: Used to install the TimiGP package from GitHub
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:37688e9cfc0bc1b100915a81c367193adf190ae2c7349683818ea676be8c92f4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - devtools
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Cell-cell interaction network degree analysis

## Summary

Builds a directed cell-cell interaction network from TimiGP's prognostic marker gene pairs and computes per-cell favorability (out-degree vs in-degree) scores, aggregated across 23 TCGA solid tumor types to reveal consistently favorable (B, cytotoxic) versus unfavorable (neutrophil, Th2) immune cells.

## When to use

Use when bulk transcriptome + survival data have been reduced to prognostic immune marker gene pairs (IMGPs) and you need to rank cell types by favorable vs unfavorable prognostic associations via network degree.

## When NOT to use

- Input lacks matched survival information or has low-quality event/time-to-event data — TimiGP requires large datasets with matched survival to identify prognostic gene pairs
- The goal is deconvolving absolute cell-type abundances from bulk data (e.g., CIBERSORTx/xCell-style estimation) rather than prognostic cell-cell interaction ranking
- Cell types of interest have very few marker genes — enrichment-based sensitivity is low and interactions are unlikely to be detected

## Inputs

- Marker gene pair score matrix (MPS) of prognostic IMGPs selected by TimiCOX with BH adjustment (top 4352 pairs, QV<0.05)
- Cell-type marker annotations (e.g., CellType_Bindea2013_cancer with immune cell types plus 1 cancer cell type)
- Per-cohort TimiEnrich/TimiPermFDR enrichment results for each of the 23 TCGA cohorts
- Background gene pairs from TimiBG

## Outputs

- Directed cell-cell interaction network (from TimiCellNetwork)
- Per-cell favorability scores: favorable = out-degree/total out-degree*100; unfavorable = in-degree/total in-degree*100
- Aggregated pan-cancer favorability table (pancancer_favorability_scores.csv) across 23 cancer types

## How to apply

Run TimiCellPair on the marker annotations to generate cell-interaction marker pair annotations and TimiBG to build the enrichment background, then run TimiEnrich with p.adj='BH' on query gene pairs (QV<0.05) followed by TimiPermFDR with niter=100. Construct the directed cell-cell interaction network with TimiCellNetwork, where pair direction encodes prognosis: pairs with HR>1 are reversed so A_B=1 denotes a favorable-prognosis association. Compute per-cell favorability with TimiFS, where favorable score = out-degree/total out-degree*100 and unfavorable score = in-degree/total in-degree*100. Repeat per cohort (23 TCGA tumor types, each preprocessed with TimiCheckEvent and TimiPrePropress with log=TRUE and GMNorm=TRUE, capped at the top 4352 IMGPs via TimiCOX with BH adjustment) and aggregate into pancancer_favorability_scores.csv.

## Related tools

- **TimiGP** (R package providing TimiCellPair, TimiBG, TimiEnrich, TimiPermFDR, TimiCellNetwork, and TimiFS to build the interaction network and degree-based favorability scores) — https://github.com/CSkylarL/TimiGP
- **R** (Execution environment for the TimiGP workflow and aggregation of per-cohort scores into pancancer_favorability_scores.csv)
- **devtools** (Used to install the TimiGP package from GitHub) — https://github.com/r-lib/devtools

## Examples

```
devtools::install_github("CSkylarL/TimiGP"); library(TimiGP)
```

## Evaluation signals

- Each prognostic pair direction is oriented so A_B=1 denotes a favorable-prognosis association (HR>1 pairs reversed) before network construction
- Favorable and unfavorable scores are bounded 0–100 by construction (out-degree/total out-degree*100 and in-degree/total in-degree*100)
- Enrichment results pass TimiPermFDR with niter=100 after BH adjustment at QV<0.05
- Reproduced biology: enrichment of cytotoxic-to-other and other-to-tumor interactions, with B and cytotoxic cells consistently favorable and neutrophils/Th2 cells unfavorable across cancer types
- Aggregated table contains favorability scores for all 23 TCGA solid tumor types in pancancer_favorability_scores.csv

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs; power depends on sample size, number of events, and survival-data quality
- Since interactions are identified by enrichment analysis, sensitivity for a specific interaction depends on the number of its marker genes; cell types with fewer markers are less likely to show significant interactions
- No wet-lab experiments were performed to validate the findings, raising uncertainty about the reliability of inferred cell-cell interactions

## Evidence

- [methods] cell interaction annotations built via TimiCellPair/TimiBG enrichment: "`TimiCellPair` will generate marker pair annotations of cell interactions."
- [methods] background generation for enrichment: "`TimiBG` will generate background gene pairs for enrichment analysis."
- [methods] network analysis step: "5. Build and analyze the cell-cell interaction network."
- [readme] rationale for degree-based favorability: "The tumor immune microenvironment(TIME) is a balance between anti-tumor and pro-tumor immune cells."
- [discussion] network-based prognostic association estimation: "TimiGP starts with pairwise relations at the expression level and estimates prognostic association of immune cells through network analysis"
