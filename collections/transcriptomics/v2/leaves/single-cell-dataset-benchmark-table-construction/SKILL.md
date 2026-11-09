---
name: single-cell-dataset-benchmark-table-construction
description: Use when a cell can belong to multiple inferred trajectories and you
  need to quantify whether its inferred pseudotime is consistent across those paths,
  benchmarked per dataset.
when_to_use_negative:
- Cells assigned to only a single trajectory path — Diffpst is undefined since it
  requires deviation across multiple assigned paths
- Trajectory outputs from methods that do not emit per-path cell pseudotime and branching
  assignments (e.g., Directed-PAGA infers only cluster-level trajectories)
- Datasets not processed through CellPath, where percentile pseudotime positions per
  path cannot be extracted from cellpath_obj.pseudo_order
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellPath
  role: Produces the per-path cell pseudotime and branching assignments (cellpath_obj.pseudo_order)
    that Diffpst consistency statistics are computed from
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: pandas
  role: Tabulates per-dataset proportions of the four Diffpst categories and writes
    diffpst_category_proportions.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:d76db6085047ac6092c63406f30eef785fd5d7fdd24075c743fb5ec5f05f3b14
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# single-cell-dataset-benchmark-table-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify per-dataset pseudotime consistency for cells assigned to multiple CellPath trajectories by computing Diffpst (mean absolute deviation of within-path percentile pseudotime positions), binning deviations into four categories, and tabulating per-dataset proportions (e.g., forebrain ~10.2% in the large-deviation bin, Figure S4G).

## When to use

Use when a cell can belong to multiple inferred trajectories and you need to quantify whether its inferred pseudotime is consistent across those paths, benchmarked per dataset.

## When NOT to use

- Cells assigned to only a single trajectory path — Diffpst is undefined since it requires deviation across multiple assigned paths
- Trajectory outputs from methods that do not emit per-path cell pseudotime and branching assignments (e.g., Directed-PAGA infers only cluster-level trajectories)
- Datasets not processed through CellPath, where percentile pseudotime positions per path cannot be extracted from cellpath_obj.pseudo_order

## Inputs

- cellpath_obj.pseudo_order — pseudo-time and branching assignment results for each of the four real benchmark datasets
- Benchmark dataset h5ad files (e.g., example_data/real/forebrain/fb_clust.h5ad) with CellPath trajectories already inferred
- Diffpst category bin definitions (four small-to-large deviation bins)

## Outputs

- diffpst_category_proportions.csv — per-dataset proportions of multi-path cells across four Diffpst deviation categories, each row summing to 1
- Per-dataset proportion of cells in the large-Diffpst category (expected ~10.2% for forebrain)

## How to apply

Load the CellPath pseudo-time and branching assignment results (cellpath_obj.pseudo_order) for each of the four benchmark datasets: mouse hematopoiesis, dentate gyrus, pancreatic endocrinogenesis, and forebrain. For each trajectory path, convert cell pseudotimes into percentile positions scaled to [0, 1] along the path. For every cell assigned to more than one path, compute Diffpst as the mean absolute deviation of its percentile positions across its assigned paths. Bin Diffpst values into four deviation categories and count multi-path cells per category per dataset. Tabulate per-dataset proportions of the four Diffpst categories with pandas and write to diffpst_category_proportions.csv.

## Related tools

- **CellPath** (Produces the per-path cell pseudotime and branching assignments (cellpath_obj.pseudo_order) that Diffpst consistency statistics are computed from) — https://github.com/PeterZZQ/CellPaths.git
- **pandas** (Tabulates per-dataset proportions of the four Diffpst categories and writes diffpst_category_proportions.csv)

## Examples

```
import pandas as pd; proportions.to_csv('diffpst_category_proportions.csv')
```

## Evaluation signals

- The forebrain column reproduces the reported proportion of multi-path cells in the large-Diffpst category of approximately 10.2% from Figure S4G
- The four Diffpst category proportions in each dataset row of diffpst_category_proportions.csv sum to 1
- Forebrain shows the largest proportion of cells with relatively large Diffpst among all four real datasets
- Only cells assigned to more than one CellPath trajectory path contribute to the Diffpst computation
- All within-path pseudotime percentiles fall within [0, 1] after scaling

## Limitations

- Pseudotime consistency is measured only for multi-path cells; single-path cells are excluded from the statistic
- CellPath does not yet provide a global-view alignment of locally detected paths or connecting points between paths, so cross-path consistency is a proxy rather than a global trajectory alignment
- Performance of the underlying trajectories depends on RNA velocity inference accuracy, since noise in estimated RNA velocity affects all RNA velocity-based methods

## Evidence

- [supplementary] Pseudotimes across paths are largely consistent; the forebrain dataset has the largest proportion of cells with relatively large Diffpst among all four real datasets, but that proportion is small (10.2%).: [evidence span withheld — anchor status: not_found]
