---
name: cell-type-importance-visualization
description: Use when you have dsCellNet-inferred ligand-receptor networks across
  multiple developmental time points (e.g., GSE153164 corticogenesis stages) and need
  per-cell-type incoming/outgoing/total edge counts to rank the most communicating
  cell types per stage.
when_to_use_negative:
- You only have a single-time-point snapshot of cell-cell interactions — dsCellNet's
  importance scoring is designed for developmental series data with multiple adjacent
  time points
- You have not yet run the dsCellNet inference pipeline (datainput, preparepairs,
  singlenet, developcorrelation), so no inferred networks exist in resloc to aggregate
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3306
tools:
- name: dsCellNet
  role: R package whose importance_plot function computes and plots row-wise incoming/outgoing/total
    scores of each cell type per time point from the inferred developmental series
    networks
  repo: github.com/songzh523/dsCellNet
- name: R
  role: Runtime environment (R >= 4.0.3) in which dsCellNet is installed and executed
  repo: github.com/songzh523/dsCellNet
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:1b76a9cb6ee78df9f057d8e70efa2272ff29a8a1723ec8319b471c2d4edd50db
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  tools:
  - R
  - dsCellNet
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3306
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cell-type-importance-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies and visualizes the incoming, outgoing, and total interaction scores of each cell type at each time point in developmental series cell-cell communication networks inferred with dsCellNet, to identify the most communicating sender and receiver cell types (e.g., apical progenitors dominating early mouse corticogenesis, intermediate progenitors dominating the middle stage).

## When to use

Use when you have dsCellNet-inferred ligand-receptor networks across multiple developmental time points (e.g., GSE153164 corticogenesis stages) and need per-cell-type incoming/outgoing/total edge counts to rank the most communicating cell types per stage.

## When NOT to use

- You only have a single-time-point snapshot of cell-cell interactions — dsCellNet's importance scoring is designed for developmental series data with multiple adjacent time points
- You have not yet run the dsCellNet inference pipeline (datainput, preparepairs, singlenet, developcorrelation), so no inferred networks exist in resloc to aggregate

## Inputs

- dsCellNet-inferred cell-cell interaction networks per time point (files in the resloc results directory)
- Cell-type color assignments produced by definecolo (usedcolo)
- Cell-type label file (typnam) and time-point file (timnam) used as dsCellNet inputs

## Outputs

- Row-wise incoming/outgoing/total score plots per cell type per time point (importance_plot output)
- Stage-by-cell-type importance score table, e.g., celltype_importance_scores.csv, with one row per time point x cell type

## How to apply

Load the inferred per-time-point cell-cell interaction networks from the dsCellNet results directory (resloc) together with the cell-type color assignments produced by definecolo (usedcolo). Aggregate ligand-receptor edges by sender cell type to obtain outgoing interaction counts, by receiver cell type to obtain incoming counts, and combine both into total scores per time point. Call importance_plot(resloc, "row", "incoming", "None"), importance_plot(resloc, "row", "outgoing", "None"), and importance_plot(resloc, "row", "total", "None") in the dsCellNet R package to compute and plot the row-wise scores of each cell type at each time point. Assemble the scores into a stage-by-cell-type table (e.g., celltype_importance_scores.csv) with one row per time point x cell type combination. The rationale is that summing edge counts by sender/receiver role reveals which cell types are the most active communicators at each developmental stage, per the README step 'statistic the incoming/outgoing/total score of each cell-type in each time-point'.

## Related tools

- **dsCellNet** (R package whose importance_plot function computes and plots row-wise incoming/outgoing/total scores of each cell type per time point from the inferred developmental series networks) — github.com/songzh523/dsCellNet
- **R** (Runtime environment (R >= 4.0.3) in which dsCellNet is installed and executed) — github.com/songzh523/dsCellNet

## Examples

```
importance_plot(resloc,"row","total","None")
```

## Evaluation signals

- Output file celltype_importance_scores.csv exists and contains one row for every time point x cell type combination
- In the GSE153164 corticogenesis analysis, apical progenitors (APs) receive the highest total score in the early stage (E10.5-E12.5)
- Intermediate progenitors (IPs) rank highest in the middle stage (E12.5-E17.5), matching the reported communication pattern
- Incoming, outgoing, and total score plots are all generated for each time point (the three importance_plot modes: incoming, outgoing, total)

## Limitations

- Scores depend on the default L-R pair list size (5,744 interactions); a limited default list can miss some interactions (e.g., MG–Neurons), which would propagate to the per-cell-type importance scores — users can supply supplementary L-R genes
- Ranks reflect edge counts aggregated by sender/receiver role; the underlying inference quality depends on protein localization filtering and DTW-based reinforcement across adjacent time points
- The tool covers developmental series RNA-seq data; stationary single-condition datasets are outside its intended scope

## Evidence

- [methods] importance_plot(resloc,"row","incoming","None"): "importance_plot(resloc,"row","incoming","None")"
- [abstract] our methods can help identify the most active cell type and genes: "our methods can help identify the most active cell type and genes, which may provide a powerful tool to uncover the mechanisms underlying the organismal development and disease"
- [readme] README importance score step: "* statistic the incoming/outgoing/total score of each cell-type in each time-point"
