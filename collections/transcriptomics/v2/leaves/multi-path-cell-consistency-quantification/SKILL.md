---
name: multi-path-cell-consistency-quantification
description: Use when a cell-level trajectory inference output (e.g. cellpath_obj.pseudo_order)
  assigns cells to more than one path and you need a per-dataset statistic of pseudotime
  agreement across those paths.
when_to_use_negative:
- Cells assigned to only a single CellPath trajectory path — Diffpst across paths
  is undefined for them
- You need a global alignment of locally detected paths rather than a per-cell pseudotime
  consistency statistic — the article lists global path alignment as a not-yet-implemented
  future direction
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellPath
  role: Produces the pseudo_order pseudo-time and branching assignment results whose
    multi-path cells are analyzed for pseudotime consistency
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: pandas
  role: Tabulates the per-dataset Diffpst category proportions and writes diffpst_category_proportions.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:22cf3b3a77db29367b8f55d081e7fe4a7a548ee40ae573b09bb4f233e47892e8
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
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# multi-path-cell-consistency-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies whether CellPath-assigned pseudotimes are consistent for cells that belong to multiple trajectory paths, by computing Diffpst (mean absolute deviation of path-percentile positions) per dataset, as in the CellPath paper's Figure S4G consistency analysis.

## When to use

Use when a cell-level trajectory inference output (e.g. cellpath_obj.pseudo_order) assigns cells to more than one path and you need a per-dataset statistic of pseudotime agreement across those paths.

## When NOT to use

- Cells assigned to only a single CellPath trajectory path — Diffpst across paths is undefined for them
- You need a global alignment of locally detected paths rather than a per-cell pseudotime consistency statistic — the article lists global path alignment as a not-yet-implemented future direction

## Inputs

- CellPath pseudo-time and branching assignment results (cellpath_obj.pseudo_order) for each dataset
- Processed scRNA-seq AnnData with velocity (e.g. fb_clust.h5ad for forebrain), used to run CellPath
- Four real benchmark datasets: mouse hematopoiesis, dentate gyrus, pancreatic endocrinogenesis, forebrain

## Outputs

- Per-cell Diffpst values (mean absolute deviation of percentile pseudotime across assigned paths)
- diffpst_category_proportions.csv — per-dataset proportions of cells in four Diffpst deviation categories
- Category proportions per dataset summing to 1

## How to apply

Load the CellPath pseudo-time and branching assignment results stored in cellpath_obj.pseudo_order for each benchmark dataset (mouse hematopoiesis, dentate gyrus, pancreatic endocrinogenesis, forebrain). For each trajectory path, rescale cell pseudotimes into percentile positions in [0, 1] so positions are comparable across paths. For every cell assigned to more than one path, compute Diffpst as the mean absolute deviation of its percentile positions across its assigned paths. Bin Diffpst into four small-to-large deviation categories, count multi-path cells per category, and tabulate per-dataset category proportions with pandas, writing the table to diffpst_category_proportions.csv. The forebrain column should reproduce the reported large-Diffpst proportion of approximately 10.2% (Figure S4G).

## Related tools

- **CellPath** (Produces the pseudo_order pseudo-time and branching assignment results whose multi-path cells are analyzed for pseudotime consistency) — https://github.com/PeterZZQ/CellPaths.git
- **pandas** (Tabulates the per-dataset Diffpst category proportions and writes diffpst_category_proportions.csv)

## Examples

```
df = pd.DataFrame({ds: proportions[d] for d in datasets}); df.to_csv('diffpst_category_proportions.csv')
```

## Evaluation signals

- Forebrain column of the proportions table reproduces the reported ~10.2% of cells in the large-Diffpst category (Figure S4G)
- In each dataset row, the four Diffpst category proportions sum to 1
- Only cells assigned to more than one path contribute to the Diffpst statistics
- Percentile positions per path fall in [0, 1] before computing absolute deviations

## Limitations

- The forebrain dataset has the largest proportion of relatively large-Diffpst cells among the four real datasets, but that proportion is small (10.2%)
- CellPath detects local paths but does not yet align them into a global view, so cross-path consistency is quantified per-cell rather than by globally reconciled pseudotime
- Performance depends on RNA velocity inference accuracy; noisy velocity estimates propagate into the pseudotimes being compared

## Evidence

- [results] Pseudotimes across paths are largely consistent; the forebrain dataset has the largest proportion of cells with relatively large Diffpst among all four real datasets, but that proportion is small (10.2%).: [evidence span withheld — anchor status: not_found]
