---
name: developmental-time-series-interaction-analysis
description: Use when scRNA-seq data span multiple developmental time points and you
  need per-cell-type sender/receiver communication scores at each stage to identify
  the most active cell types over development.
when_to_use_negative:
- Data are from a single stationary condition/time point with no developmental series
  — dsCellNet's DTW and fuzzy clustering require adjacent time points to reinforce
  interactions
- The goal is identifying ligand-receptor gene pairs rather than ranking cell-type
  communication activity — use the preparepairs/singlenet outputs directly instead
- Interactions involve cell types absent from dsCellNet's default 5,744-pair L-R list
  and no supplementary L-R gene list is supplied
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: dsCellNet
  role: Infer developmental-series cell-cell interaction networks per time point and
    compute incoming/outgoing/total importance scores via importance_plot
  repo: github.com/songzh523/dsCellNet
- name: R
  role: Implementation environment; dsCellNet is an open-access R package (requires
    R >= 4.0.3)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:f2f6ded93c70caca04e1d5b2a92edc43239f288fed77551e02a0eab3cb51dc0b
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# developmental-time-series-interaction-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies which cell types are the most communicating (highest incoming, outgoing, and total edge counts) at each developmental time point by aggregating inferred ligand-receptor edges from dsCellNet developmental-series cell-cell interaction networks, e.g. during mouse corticogenesis (GSE153164).

## When to use

Use when scRNA-seq data span multiple developmental time points and you need per-cell-type sender/receiver communication scores at each stage to identify the most active cell types over development.

## When NOT to use

- Data are from a single stationary condition/time point with no developmental series — dsCellNet's DTW and fuzzy clustering require adjacent time points to reinforce interactions
- The goal is identifying ligand-receptor gene pairs rather than ranking cell-type communication activity — use the preparepairs/singlenet outputs directly instead
- Interactions involve cell types absent from dsCellNet's default 5,744-pair L-R list and no supplementary L-R gene list is supplied

## Inputs

- Gene expression matrix file for the developmental series scRNA-seq data (e.g. GSE153164 mouse embryonic cerebral cortex)
- Cell-type label file (inptyp$Type)
- Time-point annotation file (inptim$Time)
- dsCellNet result directory (resloc) with inferred cell-cell interaction networks
- Cell-type color assignments (usedcolo) from definecolo

## Outputs

- Per-cell-type incoming, outgoing, and total communication score plots per time point (importance_plot output)
- Stage-by-cell-type score table (e.g. celltype_importance_scores.csv) with one row per time point x cell type

## How to apply

Run the dsCellNet pipeline (R >= 4.0.3) from datainput through singlenet and developcorrelation to infer per-time-point ligand-receptor networks, using definecolo to fix cell-type color assignments (usedcolo). For each time point, aggregate inferred edges by sender cell type for outgoing counts, by receiver cell type for incoming counts, and combine for total scores. Call importance_plot(resloc, "row", "incoming"/"outgoing"/"total", "None") to compute and plot row-wise per-cell-type scores at each time point. Assemble scores into a stage-by-cell-type table (e.g. celltype_importance_scores.csv) with one row per time point x cell type combination, then check that the expected dominant cell types (e.g. apical progenitors early, intermediate progenitors mid-corticogenesis) rank highest.

## Related tools

- **dsCellNet** (Infer developmental-series cell-cell interaction networks per time point and compute incoming/outgoing/total importance scores via importance_plot) — github.com/songzh523/dsCellNet
- **R** (Implementation environment; dsCellNet is an open-access R package (requires R >= 4.0.3))

## Examples

```
importance_plot(resloc,"row","total","None")
```

## Evaluation signals

- Output table exists and contains a row for every time point x cell type combination
- Row-wise scores are non-negative and incoming + outgoing edge counts are consistent with the aggregated inferred networks in resloc
- AP (apical progenitor) cell type receives the highest total score in the early stage (E10.5-E12.5) of corticogenesis
- Intermediate progenitors (IPs) rank highest in the middle stage (E12.5-E17.5), matching the reported communication pattern
- importance_plot was called for all three metrics ('incoming', 'outgoing', 'total') with identical resloc and usedcolo as the network inference steps

## Limitations

- The size of the default L-R gene list limits detection of some interactions (e.g. MG-Neurons); users can supply supplementary L-R genes
- Cause of missing MG-Neurons interactions in individual samples remains unexplained in the article
- A single map of cell-cell interactions cannot provide a global picture of dynamic interaction changes — the temporal (developmental series) design is essential

## Evidence
