---
name: cross-dataset-network-consistency-analysis
description: Use when you have several bulk transcriptomic cohorts with matched survival
  data (e.g., metastatic melanoma datasets like TCGA SKCM06) and need to test whether
  inferred cell-cell interaction networks are reproducible across cohorts and across
  different cell-type marker annotations.
when_to_use_negative:
- Datasets lack matched survival information — TimiGP requires a large gene expression
  dataset with matched survival information to identify prognostic gene pairs
- The goal is immunotherapy response rather than prognosis — use the TimiGP-Response
  module with therapy response info (e.g., TNBCaPD1info) instead of the survival-based
  workflow
- Cell types of interest have very few marker genes — enrichment-based identification
  is underpowered for marker-sparse cell types
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: TimiGP
  role: R package providing the full pipeline functions (TimiCheckEvent, TimiPrePropress,
    TimiGenePair, TimiCOX, TimiBG, TimiCellPair, TimiEnrich, TimiPermFDR, TimiCellNetwork,
    TimiFS) applied per cohort
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Runtime environment in which the TimiGP package is executed via devtools installation
- name: devtools
  role: Used to install the TimiGP R package from GitHub
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:e03d82c2efe7d0d99ffce2cf47b85b0364100610672ba27b55922ec859063eba
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - devtools
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-dataset-network-consistency-analysis

## Summary

Repeat the full TimiGP pipeline (TimiCheckEvent → TimiPrePropress → TimiGenePair → TimiCOX → TimiBG/TimiCellPair → TimiEnrich → TimiPermFDR → TimiCellNetwork → TimiFS) across multiple independent cohorts and alternative cell-type marker sets, then compare the resulting directed cell-cell interaction networks and favorability rankings to demonstrate robustness of inferred TIME interactions.

## When to use

Use when you have several bulk transcriptomic cohorts with matched survival data (e.g., metastatic melanoma datasets like TCGA SKCM06) and need to test whether inferred cell-cell interaction networks are reproducible across cohorts and across different cell-type marker annotations.

## When NOT to use

- Datasets lack matched survival information — TimiGP requires a large gene expression dataset with matched survival information to identify prognostic gene pairs
- The goal is immunotherapy response rather than prognosis — use the TimiGP-Response module with therapy response info (e.g., TNBCaPD1info) instead of the survival-based workflow
- Cell types of interest have very few marker genes — enrichment-based identification is underpowered for marker-sparse cell types

## Inputs

- Bulk transcriptomic profiles (RNA-seq or microarray) for each cohort, e.g., SKCM06rna
- Matched clinical survival info for each cohort, e.g., SKCM06info
- Cell-type marker sets, e.g., Immune_Marker_n1293, CellType_Charoentong2017_Bindea2013_Xu2018_Immune, CellType_Newman2015_LM22, CellType_Bindea2013_cancer

## Outputs

- Directed cell-cell interaction network per cohort (TimiCellNetwork, Adjust.P.Value cutoff 0.05)
- Per-cell favorable/unfavorable favorability scores (TimiFS)
- Cross-dataset consistency summary table, e.g., timigp_consistency_summary.csv
- Network similarity statistics across the eight cohorts (p < 0.0002 expected)

## How to apply

For each cohort, preprocess clinical info with TimiCheckEvent (filtering low-quality event/time-to-event data) and normalize marker expression with TimiPrePropress (log transformation plus gene-wise median normalization). Generate the binary Marker Pair Score matrix with TimiGenePair (A>B=1, A<B=0), run TimiCOX univariate Cox regression with BH adjustment (p.adj="BH"), and reverse pairs with HR>1 so all retained pairs associate with favorable prognosis. Build the enrichment background with TimiBG, cell interaction annotations with TimiCellPair, then run TimiEnrich (p.adj="BH") on marker pairs with QV<0.05 and TimiPermFDR (niter=100) to estimate FDR, exporting the directed network with TimiCellNetwork (cutoff 0.05 on Adjust.P.Value) and per-cell favorability with TimiFS. Apply the same workflow to a reference cohort (SKCM06) under alternative marker annotations such as CellType_Charoentong2017_Bindea2013_Xu2018_Immune and CellType_Newman2015_LM22, then test network similarity across all eight datasets, verifying that the cytotoxic-cell-to-tumor-cell control interaction recurs (adjusted p<0.05) and that favorable activated T cell/NK cell versus unfavorable neutrophil patterns are preserved.

## Related tools

- **TimiGP** (R package providing the full pipeline functions (TimiCheckEvent, TimiPrePropress, TimiGenePair, TimiCOX, TimiBG, TimiCellPair, TimiEnrich, TimiPermFDR, TimiCellNetwork, TimiFS) applied per cohort) — https://github.com/CSkylarL/TimiGP
- **R** (Runtime environment in which the TimiGP package is executed via devtools installation)
- **devtools** (Used to install the TimiGP R package from GitHub) — https://github.com/r-lib/devtools

## Examples

```
devtools::install_github("CSkylarL/TimiGP"); data(Immune_Marker_n1293); mps <- TimiGenePair(TimiPrePropress(info=SKCM06info, rna=SKCM06rna), gene=Immune_Marker_n1293)
```

## Evaluation signals

- The cytotoxic-cells → tumor-cells control interaction is significant (adjusted p<0.05) in all eight metastatic melanoma datasets
- Cross-dataset cell-cell interaction network similarity reaches p<0.0002
- Favorable activated T cell / NK cell versus unfavorable neutrophil favorability pattern recurs across cohorts and both alternative marker sets
- All Cox-regression marker pairs retained have HR≤1 after reversing HR>1 pairs (all associate with favorable prognosis)
- TimiPermFDR with niter=100 completes and yields FDR estimates for TimiEnrich results (QV<0.05 marker pairs)

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- Sensitivity to identify a specific cell-cell interaction is affected by the number of corresponding marker genes, since interactions are identified by enrichment analysis
- Cell types with fewer marker genes are less likely to be identified as having significant interactions
- No wet-lab experiments were performed to validate the important findings, raising uncertainty about reliability of the inferred interactions

## Evidence

- [methods] TimiCellNetwork export cutoff: [evidence span withheld — anchor status: not_found]
