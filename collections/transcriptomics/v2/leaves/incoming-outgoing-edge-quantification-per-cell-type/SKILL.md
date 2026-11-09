---
name: incoming-outgoing-edge-quantification-per-cell-type
description: Use when dsCellNet has inferred developmental series cell-cell interaction
  networks (e.g., GSE153164 corticogenesis time points) and you need to rank cell
  types by incoming/outgoing/total interaction counts per time point to identify the
  most communicating sender and receiver cell types.
when_to_use_negative:
- Input is a single-condition (stationary) scRNA-seq dataset without developmental
  series time-point information, since the scores are computed per time point of a
  developmental series network
- Cell-cell interaction networks have not yet been inferred — first run preparepairs,
  singlenet, and developcorrelation before quantifying per-cell-type edge importance
- The goal is to identify individual ligand-receptor gene pairs rather than cell-type-level
  communication roles; this skill aggregates edges by cell type only
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: dsCellNet
  role: R package whose importance_plot function computes and plots row-wise incoming/outgoing/total
    scores of each cell type at each time point from the inferred networks in resloc
  repo: github.com/songzh523/dsCellNet
- name: R
  role: Runtime environment (R >= 4.0.3) in which dsCellNet is installed and the importance_plot
    calls are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:a0c003924631cf49c1cc32d305a3c247919eed610739a3452de65a508afd51a9
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# incoming-outgoing-edge-quantification-per-cell-type

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify the communication importance of each cell type at each developmental time point by aggregating inferred ligand-receptor edges into per-cell-type incoming, outgoing, and total edge scores, using the importance_plot function of the dsCellNet R package.

## When to use

Use when dsCellNet has inferred developmental series cell-cell interaction networks (e.g., GSE153164 corticogenesis time points) and you need to rank cell types by incoming/outgoing/total interaction counts per time point to identify the most communicating sender and receiver cell types.

## When NOT to use

- Input is a single-condition (stationary) scRNA-seq dataset without developmental series time-point information, since the scores are computed per time point of a developmental series network
- Cell-cell interaction networks have not yet been inferred — first run preparepairs, singlenet, and developcorrelation before quantifying per-cell-type edge importance
- The goal is to identify individual ligand-receptor gene pairs rather than cell-type-level communication roles; this skill aggregates edges by cell type only

## Inputs

- Inferred cell-cell interaction networks per time point from dsCellNet (resloc results directory)
- Cell-type color assignments (usedcolo) from definecolo
- Cell-type labels and time-point information from datainput (inptyp, inptim)

## Outputs

- Per-cell-type incoming interaction counts per time point
- Per-cell-type outgoing interaction counts per time point
- Per-cell-type total scores per time point (importance plots from importance_plot)
- Stage-by-cell-type importance table (e.g., celltype_importance_scores.csv)

## How to apply

After running the dsCellNet pipeline (datainput, definecolo, preparepairs, singlenet, developcorrelation), load the inferred cell-cell interaction networks for each time point from the results directory (resloc) together with the cell-type color assignments (usedcolo) produced by definecolo. For each time point, aggregate the inferred ligand-receptor edges by sender cell type to obtain per-cell-type outgoing interaction counts, and by receiver cell type to obtain incoming counts; combine both into total scores. Call importance_plot(resloc, 'row', 'incoming', 'None'), importance_plot(resloc, 'row', 'outgoing', 'None'), and importance_plot(resloc, 'row', 'total', 'None') to compute and plot the row-wise scores of each cell type at each time point. Assemble the scores into a stage-by-cell-type table (e.g., celltype_importance_scores.csv) and check that dominant communicators match expected biology, e.g., apical progenitors (APs) highest in the early corticogenesis stage (E10.5-E12.5) and intermediate progenitors (IPs) highest in the middle stage (E12.5-E17.5).

## Related tools

- **dsCellNet** (R package whose importance_plot function computes and plots row-wise incoming/outgoing/total scores of each cell type at each time point from the inferred networks in resloc) — github.com/songzh523/dsCellNet
- **R** (Runtime environment (R >= 4.0.3) in which dsCellNet is installed and the importance_plot calls are executed)

## Examples

```
importance_plot(resloc,"row","total","None")
```

## Evaluation signals

- Output table contains rows for every time point x cell type combination present in the input data
- AP (apical progenitor) cell type receives the highest total score in the early stage (E10.5-E12.5), matching the reported communication pattern
- Intermediate progenitors (IPs) rank highest in the middle stage (E12.5-E17.5)
- Incoming and outgoing counts sum consistently to the total score for each cell type x time point
- importance_plot runs without error and produces plots for all three score types ('incoming', 'outgoing', 'total') with mode 'row'

## Limitations

- Scores depend on the default L-R gene list (5,744 interactions kept for the default list); limited L-R list size can cause missed interactions (e.g., MG–Neurons), though users can supply supplementary L-R genes
- The quantification reflects network topology of dsCellNet-inferred interactions, which are reinforced via dynamic time warping and refined by fuzzy clustering, so upstream inference quality directly affects the scores

## Evidence
