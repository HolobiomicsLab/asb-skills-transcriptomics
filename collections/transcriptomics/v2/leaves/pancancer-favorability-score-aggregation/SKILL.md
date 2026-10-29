---
name: pancancer-favorability-score-aggregation
description: Use when TimiGP interaction networks have been built per cohort (e.g.,
  23 TCGA cancer types) and you need to compare per-cell favorable/unfavorable degrees
  across cancer types to identify pan-cancer prognostic patterns.
when_to_use_negative:
- Input lacks matched survival information — TimiGP requires large expression datasets
  with survival data to identify prognostic gene pairs
- The goal is immunotherapy response prediction rather than overall-survival prognostic
  favorability (use the TimiGP-Response module instead)
- Working from scRNA-seq or deconvolution-based cell abundance estimates (e.g., xCell/CIBERSORTx
  outputs) rather than TimiGP marker-pair networks
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: TimiGP
  role: R package providing TimiCOX, TimiCellPair, TimiBG, TimiEnrich, TimiPermFDR,
    TimiCellNetwork, and TimiFS to derive per-cohort favorability scores that are
    aggregated pan-cancer
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Runtime environment; aggregation and summary scripts (e.g., Fig2.script4_summary_favorability_score.R)
    are executed in R
- name: devtools
  role: Used to install the TimiGP package from GitHub before running the pipeline
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:c30f2584b7b19c61c687591eeab77b03a210f849db7f045d699c0cd0ebb947f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - devtools
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# pancancer-favorability-score-aggregation

## Summary

Aggregates per-cell favorability scores computed by TimiFS from TimiGP cell-cell interaction networks across multiple TCGA solid tumor types into a single pan-cancer table (pancancer_favorability_scores.csv), revealing consistent versus heterogeneous prognostic roles of immune cell types.

## When to use

Use when TimiGP interaction networks have been built per cohort (e.g., 23 TCGA cancer types) and you need to compare per-cell favorable/unfavorable degrees across cancer types to identify pan-cancer prognostic patterns.

## When NOT to use

- Input lacks matched survival information — TimiGP requires large expression datasets with survival data to identify prognostic gene pairs
- The goal is immunotherapy response prediction rather than overall-survival prognostic favorability (use the TimiGP-Response module instead)
- Working from scRNA-seq or deconvolution-based cell abundance estimates (e.g., xCell/CIBERSORTx outputs) rather than TimiGP marker-pair networks

## Inputs

- Per-cohort TCGA clinical survival information (pre-filtered with TimiCheckEvent)
- Per-cohort TCGA bulk transcriptome profiles (log=TRUE, GMNorm=TRUE via TimiPrePropress)
- CellType_Bindea2013_cancer marker annotations (immune cell types plus 1 cancer cell type)
- Per-cohort TimiGP cell-cell interaction networks (from TimiCellNetwork after TimiEnrich QV<0.05 and TimiPermFDR niter=100)
- Per-cohort TimiFS favorability score vectors for each cell type

## Outputs

- pancancer_favorability_scores.csv aggregating favorable and unfavorable score percentages across the 23 TCGA cancer types
- Per-cell-type favorability matrix supporting cross-cancer comparison of anti-tumor vs. pro-tumor prognostic associations

## How to apply

For each cohort, first run TimiCOX (p.adj='BH') to select prognostic IMGPs (reversing pairs with HR>1 so A_B=1 denotes a favorable-prognosis association, retaining the top 4352 pairs per cancer type), then TimiCellPair/TimiBG/TimiEnrich (p.adj='BH', QV<0.05) with TimiPermFDR (niter=100), and build the directed cell-cell interaction network with TimiCellNetwork. Compute per-cell favorability with TimiFS, where favorable = out-degree/total out-degree*100 and unfavorable = in-degree/total in-degree*100. Finally, merge the 23 per-cohort score vectors into pancancer_favorability_scores.csv and inspect whether anti-tumor cells (B cells, cytotoxic cells) are consistently favorable while pro-tumor cells (neutrophils, Th2s) and tumor cells are consistently unfavorable. Cross-cancer consistency versus heterogeneity in these patterns is the analytical product of this skill.

## Related tools

- **TimiGP** (R package providing TimiCOX, TimiCellPair, TimiBG, TimiEnrich, TimiPermFDR, TimiCellNetwork, and TimiFS to derive per-cohort favorability scores that are aggregated pan-cancer) — https://github.com/CSkylarL/TimiGP
- **R** (Runtime environment; aggregation and summary scripts (e.g., Fig2.script4_summary_favorability_score.R) are executed in R)
- **devtools** (Used to install the TimiGP package from GitHub before running the pipeline) — https://github.com/r-lib/devtools

## Examples

```
Rscript Fig2/code/Fig2.script4_summary_favorability_score.R
```

## Evaluation signals

- Each cohort contributes a favorability score vector in which favorable + unfavorable percentages derive from out-degree and in-degree fractions of the TimiCellNetwork (out-degree/total out-degree*100 and in-degree/total in-degree*100)
- pancancer_favorability_scores.csv contains rows/columns for all 23 TCGA cancer types and the Bindea2013 cell types plus the cancer cell type
- Reproduced biology check: B cells and cytotoxic cells show consistently high favorable scores while neutrophils, Th2 cells, and tumor cells show unfavorable scores across cancer types
- Interaction directionality check: cytotoxic-to-other and other-to-tumor edges are enriched in the underlying networks feeding TimiFS
- Upstream parameter audit: top 4352 prognostic IMGPs retained per cancer type with QV<0.05 (BH-adjusted) enrichment and TimiPermFDR (niter=100) correction applied

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs; power depends on sample size, number of events, and survival-data quality
- Because cell-cell interactions are identified via enrichment analysis, sensitivity for a specific interaction depends on the number of its marker genes; cell types with fewer marker genes are less likely to show significant interactions
- No wet-lab experiments were performed to validate the important findings, raising uncertainty about the reliability of the inferred cell-cell interactions

## Evidence

- [methods] compute per-cell favorability scores with TimiFS: [evidence span withheld — anchor status: not_found]
- [other] aggregate across 23 cancer types: [evidence span withheld — anchor status: not_found]
- [results] pan-cancer finding: [evidence span withheld — anchor status: not_found]
- [readme] summary script exists in repository: [evidence span withheld — anchor status: not_found]
