---
name: myeloid-lineage-marker-validation
description: Use when CellPath has resolved parallel monocyte differentiation paths
  in the 6,555 day-4 in vivo mouse hematopoiesis cells and marker genes distinguishing
  each route must be recovered and verified via per-trajectory DE analysis.
when_to_use_negative:
- The dataset lacks spliced/unspliced counts, so scVelo cannot estimate RNA velocity
  and CellPath's velocity-based trajectories are not applicable
- The goal is trajectory inference alone on a simple topology dataset — traditional
  TI methods (e.g., Slingshot, Monocle-style backbone-constrained methods) suffice
  and per-trajectory DE validation is unnecessary
- The biological question concerns non-myeloid lineages where the expected markers
  (Elane, Mpo, Cd74) provide no validation signal
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellPath
  role: Infers the parallel monocyte trajectories via meta-cell construction and greedy
    path selection, and provides the per-trajectory DE analysis framework
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Computes stochastic-mode RNA velocity used by CellPath as trajectory input
    when preprocess=True
- name: statsmodels
  role: Powers the GAM likelihood-ratio test used for the per-trajectory differentially
    expressed gene analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:e89999588f82a38b4f754adeae898880750148563e5e81a66af3e78610237f6f
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
  - scVelo
  - statsmodels
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# myeloid-lineage-marker-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates myeloid lineage markers on CellPath-resolved parallel monocyte routes: after RNA-velocity-based trajectory inference on the mouse hematopoiesis data, per-trajectory differentially expressed gene analysis recovers Elane/Mpo for the Neu-like monocyte route and Cd74 for the DC-like route.

## When to use

Use when CellPath has resolved parallel monocyte differentiation paths in the 6,555 day-4 in vivo mouse hematopoiesis cells and marker genes distinguishing each route must be recovered and verified via per-trajectory DE analysis.

## When NOT to use

- The dataset lacks spliced/unspliced counts, so scVelo cannot estimate RNA velocity and CellPath's velocity-based trajectories are not applicable
- The goal is trajectory inference alone on a simple topology dataset — traditional TI methods (e.g., Slingshot, Monocle-style backbone-constrained methods) suffice and per-trajectory DE validation is unnecessary
- The biological question concerns non-myeloid lineages where the expected markers (Elane, Mpo, Cd74) provide no validation signal

## Inputs

- Mouse hematopoiesis scRNA-seq count matrix (h5ad) with spliced/unspliced layers, 6,555 day-4 in vivo cells from GEO:GSE140802 (benchmarking deposit via CellPath Dropbox link)
- RNA velocity matrix computed by scVelo in stochastic mode (or computed during CellPath initialization with preprocess=True)

## Outputs

- Greedy-selected set of approximately 20 major trajectory paths with parallel monocyte routes identified (path 0 Neu-like, path 8 DC-like)
- Cell-level pseudotime and branching assignments in cellpath_obj.pseudo_order
- Per-trajectory DE gene tables from the statsmodels GAM likelihood-ratio test containing Elane/Mpo (path 0) and Cd74 (path 8)
- cellpath_hema_results.csv

## How to apply

Initialize the model with cp.CellPath(adata, preprocess=True) so scVelo computes stochastic-mode RNA velocity on the 6,555 day-4 in vivo cells (GSE140802). Aggregate cells into meta-cells with cellpath.meta_cell_construction(flavor='leiden'), the clustering recommended for datasets with intricate trajectories, then run cellpath_obj.all_in_one(num_metacells=N, n_neighs=10, pruning=False, num_trajs=20, insertion=True, prop_insert=0.50) to build the symmetric k-nn meta-cell graph and greedily select the top 20 major trajectories. Extract cell-level pseudotime and branching assignments from cellpath_obj.pseudo_order and identify the parallel monocyte routes (path 0 Neu-like, path 8 DC-like). Run the per-trajectory differentially expressed gene analysis with the statsmodels-based GAM likelihood-ratio test, checking that Elane and Mpo (Neu-like monocyte progenitor markers) rank on path 0 and Cd74 (early DC/lymphoid marker) ranks on path 8. Write the trajectory assignments and DE tables to cellpath_hema_results.csv.

## Related tools

- **CellPath** (Infers the parallel monocyte trajectories via meta-cell construction and greedy path selection, and provides the per-trajectory DE analysis framework) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Computes stochastic-mode RNA velocity used by CellPath as trajectory input when preprocess=True)
- **statsmodels** (Powers the GAM likelihood-ratio test used for the per-trajectory differentially expressed gene analysis)

## Examples

```
cellpath_obj = cp.CellPath(adata=adata, preprocess=True); cellpath_obj.meta_cell_construction(flavor='leiden'); cellpath_obj.all_in_one(num_metacells=N, n_neighs=10, pruning=False, num_trajs=20, insertion=True, prop_insert=0.50)
```

## Evaluation signals

- The run yields approximately 20 greedy-selected paths including both parallel monocyte routes (path 0 Neu-like, path 8 DC-like)
- The DE table contains Elane and Mpo for path 0 (Neu-like monocyte progenitor markers)
- The DE table contains Cd74 for path 8 (early DC/lymphoid marker)
- cellpath_obj.pseudo_order contains cell-level pseudotime and branching assignments for the 6,555 day-4 in vivo cells
- All results are written to cellpath_hema_results.csv

## Limitations

- Performance of all RNA velocity-based methods, including CellPath, is limited by the accuracy of RNA velocity inference (noise in estimated velocity)
- CellPath detects both global and local paths but cannot yet align locally detected paths into a global view, and connecting points between paths are not implemented
- Challenges in RNA velocity estimation remain unresolved (Bergen et al., 2021), affecting the reliability of inferred trajectories and downstream marker validation

## Evidence
