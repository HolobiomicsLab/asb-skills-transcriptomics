---
name: pseudotime-percentile-normalization
description: Use when CellPath assigns a cell to more than one trajectory path (cellpath_obj.pseudo_order)
  and you need to quantify pseudotime consistency across paths per dataset, e.g. reproducing
  Figure S4G Diffpst category proportions.
when_to_use_negative:
- Cells assigned to only a single trajectory path — no cross-path comparison is possible,
  so Diffpst is undefined
- Trajectory outputs from methods that provide only one global trajectory per dataset
  (e.g. Directed-PAGA, Slingshot) rather than multiple overlapping paths
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellPath
  role: Produces the pseudo_order pseudo-time and branching assignment results that
    are percentile-normalized and compared across paths
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: pandas
  role: Tabulates the per-dataset proportions of the four Diffpst categories and writes
    diffpst_category_proportions.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:aeec2f8ba7c48ebf8d60928f612751c6153f2dd916f2980269a9c061f0738d2a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pseudotime-percentile-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies whether a cell's inferred pseudotime is consistent across the multiple CellPath trajectories it belongs to, by normalizing per-path pseudotimes to percentile positions in [0, 1] and computing a Diffpst deviation statistic per multi-path cell, binned into categories per dataset.

## When to use

Use when CellPath assigns a cell to more than one trajectory path (cellpath_obj.pseudo_order) and you need to quantify pseudotime consistency across paths per dataset, e.g. reproducing Figure S4G Diffpst category proportions.

## When NOT to use

- Cells assigned to only a single trajectory path — no cross-path comparison is possible, so Diffpst is undefined
- Trajectory outputs from methods that provide only one global trajectory per dataset (e.g. Directed-PAGA, Slingshot) rather than multiple overlapping paths

## Inputs

- cellpath_obj.pseudo_order: CellPath pseudo-time and branching assignment results for each of the four real benchmark datasets
- Annotated single-cell data files (e.g. fb_clust.h5ad for the forebrain dataset) used to run CellPath
- Path membership (branching assignment) per cell across trajectory paths

## Outputs

- diffpst_category_proportions.csv: per-dataset proportions of multi-path cells in four Diffpst deviation categories (rows sum to 1)
- Per-cell Diffpst values (mean absolute deviation of percentile-normalized pseudotimes across assigned paths)

## How to apply

Load the pseudo-time and branching assignment results (cellpath_obj.pseudo_order) produced by CellPath's first-order pseudotime reconstruction for each of the four benchmark datasets (mouse hematopoiesis, dentate gyrus, pancreatic endocrinogenesis, forebrain). For each trajectory path, rescale the cells' pseudotimes to percentile positions in [0, 1] along the path so that paths of different lengths and cell counts become comparable. For every cell assigned to more than one path, compute Diffpst as the mean absolute deviation of its percentile positions across its assigned paths. Bin Diffpst values into four small-to-large deviation categories, tabulate per-dataset proportions of multi-path cells per category with pandas, and write diffpst_category_proportions.csv. The rationale is that CellPath outputs multiple high-resolution trajectories without topology constraints, so cells can appear on several paths and their pseudotimes should largely agree if the inference is internally consistent.

## Related tools

- **CellPath** (Produces the pseudo_order pseudo-time and branching assignment results that are percentile-normalized and compared across paths) — https://github.com/PeterZZQ/CellPaths.git
- **pandas** (Tabulates the per-dataset proportions of the four Diffpst categories and writes diffpst_category_proportions.csv)

## Examples

```
adata = anndata.read_h5ad("../example_data/real/forebrain/fb_clust.h5ad"); cellpath_obj = cp.CellPath(adata=adata, preprocess=True); cellpath_obj.all_in_one(num_metacells=num_metacells, n_neighs=10, pruning=False, num_trajs=num_trajs, insertion=True, prop_insert=0.50); paths = cellpath_obj.pseudo_order
```

## Evaluation signals

- The forebrain column of diffpst_category_proportions.csv reproduces the reported proportion of cells in the large-Diffpst category of approximately 10.2% (Figure S4G)
- The four Diffpst category proportions in each dataset row sum to 1
- Percentile positions along every path fall within [0, 1] after rescaling
- Diffpst is computed only for cells assigned to more than one path; single-path cells are excluded
- All four datasets (hematopoiesis, dentate gyrus, pancreatic endocrinogenesis, forebrain) yield proportions, with forebrain showing the largest share of relatively large Diffpst values

## Limitations

- The forebrain dataset has the largest proportion of cells with relatively large Diffpst, indicating lower cross-path pseudotime consistency there (10.2% of multi-path cells), even though pseudotimes are largely consistent overall
- CellPath's performance depends on RNA velocity accuracy; noisy velocity estimates affect the trajectory and pseudotime inputs to this analysis
- Insertion of unassigned cells (insertion=True, prop_insert) changes path membership and thus which cells receive a Diffpst value

## Evidence
