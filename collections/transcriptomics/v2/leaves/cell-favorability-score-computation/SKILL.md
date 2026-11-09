---
name: cell-favorability-score-computation
description: Use when a significant directed cell-cell interaction network (e.g.,
  cytotoxic cells → tumor cells) has already been inferred from bulk expression plus
  survival data with TimiGP and you need per-cell favorable/unfavorable prognostic
  rankings.
when_to_use_negative:
- The cohort lacks matched survival information or is too small/few events to power
  the univariate Cox selection of prognostic marker pairs
- Cell types of interest have very few marker genes, since interaction detection is
  enrichment-based and such cell types are less likely to yield significant edges
- The input is already a per-cell abundance/feature table (e.g., CIBERSORTx/xCell
  output) rather than bulk expression amenable to marker-pair scoring
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: TimiGP
  role: R package providing the full five-step framework including TimiCellNetwork
    and TimiFS for cell-cell interaction network construction and favorability scoring
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Execution environment in which the TimiGP functions (TimiCOX, TimiEnrich,
    TimiPermFDR, TimiCellNetwork, TimiFS) are called
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:e2fbc7ad1d4340a9ab9f3b916c3099402eb9440f31a278d80476dc7bf38c07f5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cell-favorability-score-computation

## Summary

Computes per-cell favorability scores from a TimiGP-derived directed cell-cell interaction network using TimiFS: an out-degree-based favorable score and an in-degree-based unfavorable score, each normalized across all cell types and scaled by 100. This ranks immune cells by their prognostic impact in the tumor immune microenvironment.

## When to use

Use when a significant directed cell-cell interaction network (e.g., cytotoxic cells → tumor cells) has already been inferred from bulk expression plus survival data with TimiGP and you need per-cell favorable/unfavorable prognostic rankings.

## When NOT to use

- The cohort lacks matched survival information or is too small/few events to power the univariate Cox selection of prognostic marker pairs
- Cell types of interest have very few marker genes, since interaction detection is enrichment-based and such cell types are less likely to yield significant edges
- The input is already a per-cell abundance/feature table (e.g., CIBERSORTx/xCell output) rather than bulk expression amenable to marker-pair scoring

## Inputs

- Bulk transcriptomic profile (RNA-seq or microarray) of marker genes (e.g., TCGA SKCM06 SKCM06rna)
- Matched clinical survival information (e.g., SKCM06info), event and time-to-event
- Cell-type marker annotations (e.g., CellType_Bindea2013_cancer, CellType_Newman2015_LM22, Immune_Marker_n1293)
- Significant directed cell-cell interaction network from TimiCellNetwork (cutoff 0.05 on Adjust.P.Value)

## Outputs

- Favorability score table with one favorable score and one unfavorable score per cell type, normalized across cells and scaled by 100
- Ranked per-cell prognostic prioritization (e.g., favorable cytotoxic/activated T cells and NK cells, unfavorable neutrophils in metastatic melanoma)

## How to apply

First run the upstream TimiGP pipeline: preprocess with TimiCheckEvent and TimiPrePropress (log=TRUE, GMNorm=TRUE), build the Marker Pair Score matrix with TimiGenePair, select favorable pairs with TimiCOX univariate Cox regression (BH-adjusted QV<0.05, HR>1 pairs reversed), annotate cross-cell pairs with TimiCellPair, build the TimiBG background, and test enrichment with TimiEnrich (p.adj='BH') followed by TimiPermFDR (niter=100). Construct the directed cell-cell interaction network with TimiCellNetwork using a 0.05 cutoff on Adjust.P.Value. Then apply TimiFS to this network: it computes an out-degree-based favorable score (a cell type is favorable when its markers dominate the favorable prognostic pairs pointing outward) and an in-degree-based unfavorable score, each normalized to the sum over all cell types and scaled by 100. In metastatic melanoma this yields favorable activated T cell / NK cell scores and unfavorable neutrophil scores, with the cytotoxic-to-tumor control interaction recurring across datasets.

## Related tools

- **TimiGP** (R package providing the full five-step framework including TimiCellNetwork and TimiFS for cell-cell interaction network construction and favorability scoring) — https://github.com/CSkylarL/TimiGP
- **R** (Execution environment in which the TimiGP functions (TimiCOX, TimiEnrich, TimiPermFDR, TimiCellNetwork, TimiFS) are called)

## Examples

```
devtools::install_github("CSkylarL/TimiGP"); fs <- TimiFS(cell_network)
```

## Evaluation signals

- Cox selection (TimiCOX) retains only marker pairs with BH-adjusted QV<0.05, and pairs with HR>1 are reversed so all retained pairs associate with favorable prognosis
- The favorability score table contains both a favorable and an unfavorable score for every cell type present in the marker set
- Each score is normalized to the sum over all cell types and scaled by 100 (values on a comparable 0–100 scale across cells)
- The cytotoxic cells → tumor cells control interaction is significant (adjusted p<0.05) in the network feeding TimiFS, e.g., across all eight metastatic melanoma cohorts
- Cross-dataset/alternative-marker-set runs produce similar cell-cell interaction networks (similarity p<0.0002) and recurring favorable activated T cell / NK cell vs unfavorable neutrophil rankings

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs; power depends on sample size, number of events, and quality of survival information
- Because cell-cell interactions are identified by enrichment analysis, sensitivity for a specific interaction depends on the number of corresponding marker genes; cell types with fewer markers are less likely to score significantly
- No wet-lab experiments were performed to validate the important findings, raising uncertainty about the reliability of inferred cell-cell interactions and downstream favorability scores

## Evidence
