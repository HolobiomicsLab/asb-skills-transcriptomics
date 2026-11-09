---
name: trajectory-assignment-deviation-analysis
description: Use when cells are assigned to more than one CellPath trajectory (cellpath_obj.pseudo_order)
  and you need to quantify whether inferred pseudotime is consistent across those
  paths per dataset.
when_to_use_negative:
- Cells assigned to only a single trajectory path, since Diffpst cannot be computed
  without at least two path assignments
- Trajectory outputs from tools other than CellPath, whose pseudo_order structure
  differs (e.g. PAGA, Slingshot, CellRank outputs)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellPath
  role: Source of the pseudo_order pseudo-time and branching assignment results from
    which percentile pseudotime positions and Diffpst deviations are computed
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: pandas
  role: Tabulation of per-dataset proportions of the four Diffpst categories and writing
    the results table to CSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:0e76088cf7793e62e387e7ca1daf894ad16a0984902d35ac3c340b85d0ab267d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PeterZZQ/CellPath
  repo_url: https://github.com/PeterZZQ/CellPaths.git
  tools:
  - CellPath
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# trajectory-assignment-deviation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies pseudotime consistency for cells assigned to multiple CellPath trajectories by computing Diffpst, the mean absolute deviation of each cell's percentile-scaled pseudotime positions across its assigned paths, and tabulating per-dataset Diffpst category proportions.

## When to use

Use when cells are assigned to more than one CellPath trajectory (cellpath_obj.pseudo_order) and you need to quantify whether inferred pseudotime is consistent across those paths per dataset.

## When NOT to use

- Cells assigned to only a single trajectory path, since Diffpst cannot be computed without at least two path assignments
- Trajectory outputs from tools other than CellPath, whose pseudo_order structure differs (e.g. PAGA, Slingshot, CellRank outputs)

## Inputs

- CellPath pseudo-time and branching assignment results (cellpath_obj.pseudo_order) for the four real benchmark datasets (mouse hematopoiesis, dentate gyrus, pancreatic endocrinogenesis, forebrain)
- Cell-to-path assignment metadata indicating which cells belong to multiple trajectory paths

## Outputs

- diffpst_category_proportions.csv: per-dataset proportions of multi-path cells across four Diffpst deviation categories (rows sum to 1)

## How to apply

Load the CellPath pseudo-time and branching assignment results (cellpath_obj.pseudo_order) for each benchmark dataset (mouse hematopoiesis, dentate gyrus, pancreatic endocrinogenesis, forebrain). For each trajectory path, convert cell pseudotimes into percentile positions scaled to [0, 1] along the path so positions are comparable across paths of different lengths. For every cell assigned to more than one path, compute Diffpst as the mean absolute deviation of its percentile positions across its assigned paths. Bin Diffpst values into four deviation categories (small to large), count multi-path cells per category per dataset, and write the per-dataset proportions table to diffpst_category_proportions.csv with pandas.

## Related tools

- **CellPath** (Source of the pseudo_order pseudo-time and branching assignment results from which percentile pseudotime positions and Diffpst deviations are computed) — https://github.com/PeterZZQ/CellPaths.git
- **pandas** (Tabulation of per-dataset proportions of the four Diffpst categories and writing the results table to CSV)

## Examples

```
import pandas as pd; props = counts.div(counts.sum(axis=1), axis=0); props.to_csv('diffpst_category_proportions.csv')
```

## Evaluation signals

- The forebrain column of the table reproduces the reported proportion of cells in the large-Diffpst category of approximately 10.2% (Figure S4G)
- The four Diffpst category proportions in each dataset row sum to 1
- Every Diffpst value is computed only for cells assigned to more than one trajectory path
- All percentile pseudotime positions fall within [0, 1] after path-wise rescaling

## Limitations

- The forebrain dataset has the largest proportion of cells with relatively large Diffpst among the four real datasets, indicating pseudotime consistency is dataset-dependent
- CellPath performance is affected by accuracy of RNA velocity inference, since it depends on velocity estimates from upstream tools (scVelo, velocyto)

## Evidence
