---
name: marker-pair-enrichment-analysis
description: Use when you have a bulk RNA-seq or microarray profile with matched survival
  (event + time-to-event) info and a cell-type marker geneset, and you need directed
  cell-cell interactions (FDR<0.05) plus per-cell prognostic favorability scores.
when_to_use_negative:
- The expression dataset is small or lacks high-quality matched survival information
  — TimiGP requires a large gene expression dataset with matched survival data to
  identify prognostic pairs
- The cell types of interest have very few marker genes — sensitivity to identify
  a specific interaction is affected by the number of corresponding marker genes
- The goal is direct ligand-receptor communication from single-cell data rather than
  prognostic association inferred from bulk co-expression with survival
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: TimiGP
  role: R package providing TimiGenePair, TimiCOX, TimiCellPair, TimiBG, TimiEnrich,
    TimiPermFDR, TimiCellNetwork, and TimiFS for the full marker-pair enrichment workflow
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Execution environment; TimiGP is installed via devtools::install_github('CSkylarL/TimiGP')
  repo: https://github.com/r-lib/devtools
- name: survival
  role: Underlying Cox regression used by TimiCOX to select prognostic marker pairs
    against survival outcomes
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_004
  - task_008
  - task_002
  - task_005
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:f1508e13711061bdc2bbde2438adc4d75d51fec574e410e092accf15d739b0e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiCOX
  - TimiFS
  - TimiGP
  - TimiPermFDR
  - survival
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# marker-pair-enrichment-analysis

## Summary

TimiGP's enrichment-based procedure that converts prognostic marker gene pairs (IMGPs) from univariate Cox screening into directed cell-cell interaction calls and per-cell favorability scores, using hypergeometric enrichment plus permutation-based FDR on bulk transcriptomes with matched survival data.

## When to use

Use when you have a bulk RNA-seq or microarray profile with matched survival (event + time-to-event) info and a cell-type marker geneset, and you need directed cell-cell interactions (FDR<0.05) plus per-cell prognostic favorability scores.

## When NOT to use

- The expression dataset is small or lacks high-quality matched survival information — TimiGP requires a large gene expression dataset with matched survival data to identify prognostic pairs
- The cell types of interest have very few marker genes — sensitivity to identify a specific interaction is affected by the number of corresponding marker genes
- The goal is direct ligand-receptor communication from single-cell data rather than prognostic association inferred from bulk co-expression with survival

## Inputs

- Bulk transcriptomic profile (RNA-seq or microarray), e.g. SKCM06rna
- Clinical survival info with event and time-to-event fields, e.g. SKCM06info
- Cell-type marker annotation geneset, e.g. CellType_Bindea2013_cancer, CellType_Charoentong2017_Bindea2013_Xu2018_Immune, CellType_Newman2015_LM22, or scRNA-seq-derived markers

## Outputs

- Marker Pair Score (MPS) binary matrix from TimiGenePair
- Cox-selected prognostic IMGP table with hazard ratios, BH-adjusted Q values, and directionality (HR>1 pairs reversed)
- Significant directed cell-cell interactions with adjusted p-values and permutation FDR (<0.05), e.g. the 54/462 significant pairs in TCGA_SKCM06
- Directed cell-cell interaction network from TimiCellNetwork
- Per-cell favorable/unfavorable favorability scores from TimiFS (out-degree- and in-degree-based, scaled by 100)

## How to apply

First filter clinical info with TimiCheckEvent and preprocess marker expression with TimiPrePropress (log=TRUE, GMNorm=TRUE for gene-wise median normalization). Build the binary Marker Pair Score matrix with TimiGenePair (default logical relation: A>B=1), then run TimiCOX univariate Cox regression on each pair with BH adjustment (QV<0.05), reversing pairs with HR>1 so all retained pairs associate with favorable prognosis. Generate cell-interaction annotations with TimiCellPair on the marker geneset and the enrichment background with TimiBG (covering both directions of each pair). Query gene pairs with QV<0.05, run TimiEnrich (hypergeometric test, p.adj='BH') on the cell-pair annotation, and estimate significance with TimiPermFDR (niter=100). Finally build the directed cell network with TimiCellNetwork (cutoff 0.05 on Adjust.P.Value) and compute per-cell favorability with TimiFS (favorable = out-degree/total out-degree*100, unfavorable = in-degree/total in-degree*100).

## Related tools

- **TimiGP** (R package providing TimiGenePair, TimiCOX, TimiCellPair, TimiBG, TimiEnrich, TimiPermFDR, TimiCellNetwork, and TimiFS for the full marker-pair enrichment workflow) — https://github.com/CSkylarL/TimiGP
- **R** (Execution environment; TimiGP is installed via devtools::install_github('CSkylarL/TimiGP')) — https://github.com/r-lib/devtools
- **survival** (Underlying Cox regression used by TimiCOX to select prognostic marker pairs against survival outcomes)

## Examples

```
res <- TimiPermFDR(resdata = res, geneset = geneset, gene = GP, niter = 100)
```

## Evaluation signals

- On TCGA_SKCM06 with CellType_Bindea2013_cancer, TimiCellNetwork must yield 54 significant cell pairs out of 462 possible pairs (FDR<0.05)
- The cytotoxic cells → tumor cells interaction must rank among the top significant interactions in metastatic melanoma cohorts
- Tumor cells must receive the highest unfavorable favorability score, while B/cytotoxic cells score favorably (pan-cancer: favorable B/cytotoxic vs unfavorable neutrophils/Th2)
- TimiCOX output must contain only pairs with QV<0.05, and every retained pair with HR>1 must be reversed so it associates with favorable prognosis
- The TimiFS score table must contain a favorable and an unfavorable score for every cell type in the input marker set

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- Power of the survival analysis is determined by sample size, number of events, and quality of survival information
- Since cell-cell interactions are identified by enrichment analysis, sensitivity for a specific interaction depends on the number of corresponding marker genes; cell types with fewer markers are less likely to show significant interactions
- No wet-lab experiments were performed to validate the findings, raising some uncertainty about the reliability of inferred cell-cell interactions

## Evidence
