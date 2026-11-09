---
name: single-cell-pseudotime-assignment
description: Use when snapshot scRNA-seq data (e.g. h5ad count matrices) lacks direction
  information and you need high-resolution, multi-branch trajectories plus per-cell
  pseudotime, especially with intricate topologies like parallel differentiation routes.
when_to_use_negative:
- When time-resolved (time-course) data already provides direction so velocity inference
  is unnecessary
- When only a single pre-defined linear topology is expected and cluster-level tools
  like PAGA/Slingshot suffice
- When RNA velocity estimates are unreliable for the tissue (all velocity-based TI
  performance is limited by velocity inference accuracy)
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellPath
  role: 'Main method: meta-cell construction, k-nn meta-cell graph path finding, greedy
    trajectory selection, and first-order pseudotime reconstruction'
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Computes the RNA velocity matrix used as CellPath input during initialization
    (preprocess=True)
- name: statsmodels
  role: Powers the optional per-trajectory DE gene analysis via GAM likelihood-ratio
    test
- name: leiden
  role: Clustering flavor used in meta_cell_construction, recommended for datasets
    with intricate trajectories
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:451365814371007cbd5c68d3dcb04bb30d4b1625755dd2921023f5ff096be93f
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
  - leiden
  - scVelo
  - statsmodels
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# single-cell-pseudotime-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assign cell-level pseudotime and branching/trajectory labels by integrating scRNA-seq expression with RNA velocity via CellPath: meta-cells are built with leiden clustering, a symmetric k-nn meta-cell graph is searched for candidate paths, a greedy algorithm selects major trajectories, and first-order pseudotime reconstruction maps cells onto paths.

## When to use

Use when snapshot scRNA-seq data (e.g. h5ad count matrices) lacks direction information and you need high-resolution, multi-branch trajectories plus per-cell pseudotime, especially with intricate topologies like parallel differentiation routes.

## When NOT to use

- When time-resolved (time-course) data already provides direction so velocity inference is unnecessary
- When only a single pre-defined linear topology is expected and cluster-level tools like PAGA/Slingshot suffice
- When RNA velocity estimates are unreliable for the tissue (all velocity-based TI performance is limited by velocity inference accuracy)

## Inputs

- AnnData h5ad object with scRNA-seq count matrix (optionally with precomputed RNA velocity from scVelo/velocyto)
- Number of meta-cells (num_metacells) and number of trajectories to output (num_trajs)

## Outputs

- cellpath_obj.pseudo_order: per-cell pseudotime and branching/trajectory assignments
- Meta-cell-level major trajectories (greedy-selected, e.g. ~20 paths)
- Optional DE gene table per trajectory (e.g. written to cellpath_hema_results.csv)

## How to apply

Initialize cp.CellPath(adata, preprocess=True) so scVelo computes stochastic-mode RNA velocity on the raw count matrix (e.g. the 6,555 day-4 in vivo mouse hematopoiesis cells from GEO:GSE140802). Aggregate cells into meta-cells with cellpath.meta_cell_construction(flavor='leiden'), the clustering recommended for datasets with intricate trajectories. Run cellpath_obj.all_in_one(num_metacells=N, n_neighs=10, pruning=False, num_trajs=20, insertion=True, prop_insert=0.50) to build the symmetric k-nn meta-cell graph, enumerate candidate paths, and greedily select the top 20 major trajectories. Extract cell-level pseudotime and branching assignments from cellpath_obj.pseudo_order, then optionally run the statsmodels-based GAM likelihood-ratio test per trajectory for DE genes (e.g. Elane/Mpo on path 0, Cd74 on path 8).

## Related tools

- **CellPath** (Main method: meta-cell construction, k-nn meta-cell graph path finding, greedy trajectory selection, and first-order pseudotime reconstruction) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Computes the RNA velocity matrix used as CellPath input during initialization (preprocess=True))
- **statsmodels** (Powers the optional per-trajectory DE gene analysis via GAM likelihood-ratio test)
- **leiden** (Clustering flavor used in meta_cell_construction, recommended for datasets with intricate trajectories)

## Examples

```
cellpath_obj = cp.CellPath(adata=adata, preprocess=True); cellpath_obj.meta_cell_construction(flavor='leiden'); cellpath_obj.all_in_one(num_metacells=N, n_neighs=10, pruning=False, num_trajs=20, insertion=True, prop_insert=0.50); cellpath_obj.pseudo_order
```

## Evaluation signals

- Run yields approximately the requested number of greedy-selected major trajectories (e.g. ~20 with num_trajs=20) including expected parallel routes (path 0 Neu-like, path 8 DC-like in mouse hematopoiesis)
- cellpath_obj.pseudo_order is populated with pseudotime and branching assignment for the analyzed cells
- Per-trajectory DE analysis recovers expected markers: Elane and Mpo on path 0 and Cd74 on path 8
- Unassigned cells are incorporated at the configured proportion (insertion=True, prop_insert=0.50)
- Results reproducibly written to cellpath_hema_results.csv matching the CellPath_test manuscript benchmark scripts

## Limitations

- Performance of all RNA velocity-based methods, including CellPath, is limited by the accuracy of RNA velocity inference (Bergen et al., 2021 challenges remain unresolved)
- No global view alignment of locally detected paths yet; aligning local paths into a global view is left as a future direction
- Connecting points between paths are not yet implemented
- rpy2 >= 3.3.0 is required only for the principal curve option

## Evidence
