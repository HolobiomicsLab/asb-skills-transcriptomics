---
name: kngraph-path-finding
description: Use when a scRNA-seq AnnData with a computed RNA velocity matrix must
  be resolved into multiple high-resolution trajectories without pre-imposed topology,
  at meta-cell resolution.
when_to_use_negative:
- When only a trajectory-structure-free visualization is needed rather than extracted
  major paths — CellRank or dynamo output initial/terminal states and fate maps but
  do not extract cell trajectories
- When cluster-level output suffices — Directed-PAGA infers cluster-level trajectories,
  so high-resolution meta-cell paths would be unnecessary complexity
- When velocity has not been computed and cannot be estimated reliably — RNA velocity-based
  performance depends on the accuracy of the upstream velocity inference
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: CellPath
  role: Implements meta-cell construction, meta-cell kNN graph building, path finding,
    greedy major-trajectory selection, cell insertion, and pseudotime assignment
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Computes the RNA velocity matrix (dynamical mode) stored in adata, the direction
    signal used to build and orient the meta-cell graph paths
- name: statsmodels
  role: Optional dependency for downstream differentially expressed gene analysis
    per trajectory path
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:91f7972cac68cb55e4ea2c3fb49f6d67cf1e9131e29eb95aae1bf881ce722906
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# kngraph-path-finding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a k-nearest-neighbor graph on K-means (or leiden) meta-cells built from scRNA-seq expression plus smoothed RNA velocity, then extracts and greedily selects a small set of major meta-cell-level trajectories with a path finding algorithm, as implemented in CellPath's all_in_one step.

## When to use

Use when a scRNA-seq AnnData with a computed RNA velocity matrix must be resolved into multiple high-resolution trajectories without pre-imposed topology, at meta-cell resolution.

## When NOT to use

- When only a trajectory-structure-free visualization is needed rather than extracted major paths — CellRank or dynamo output initial/terminal states and fate maps but do not extract cell trajectories
- When cluster-level output suffices — Directed-PAGA infers cluster-level trajectories, so high-resolution meta-cell paths would be unnecessary complexity
- When velocity has not been computed and cannot be estimated reliably — RNA velocity-based performance depends on the accuracy of the upstream velocity inference

## Inputs

- AnnData object (adata) with scRNA-seq count matrix and RNA velocity matrix (e.g., computed with scvelo dynamical mode), such as the 3,696-cell mouse pancreatic endocrinogenesis E15.5 dataset from GEO:GSE132188
- num_metacells: total number of meta-cells
- n_neighs: number of neighbors per meta-cell (default 10 in the paper's runs)
- num_trajs: number of trajectories to output (e.g., 7)

## Outputs

- cellpath_obj.pseudo_order: pseudotime and branching assignment per cell/trajectory
- A set of num_trajs meta-cell-level trajectory paths (e.g., path 0 = beta, path 1 = delta, path 4 = alpha in the endocrinogenesis example)
- Meta-cell kNN graph intermediate object

## How to apply

Initialize CellPath on the AnnData with velocity already computed (scvelo dynamical mode) or with preprocess=True so velocity is calculated during initialization. Build meta-cells with flavor='k-means' (leiden is available for intricate trajectories) in cellpath.meta_cell_construction(), then call cellpath_obj.all_in_one(num_metacells=num_metacells, n_neighs=10, pruning=False, num_trajs=num_trajs, insertion=True, prop_insert=0.50). The graph is built by connecting each meta-cell to its n_neighs nearest neighbors (pruning=False includes more edges rather than pruning knn edges); a path finding algorithm generates a pool of possible trajectories, and a greedy algorithm selects num_trajs major meta-cell-level trajectories (e.g., num_trajs=7 on the 3,696-cell mouse pancreatic endocrinogenesis E15.5 dataset). Unassigned cells are then inserted into trajectories up to prop_insert=0.50 of cells, and pseudotime/branching assignment is read from cellpath_obj.pseudo_order.

## Related tools

- **CellPath** (Implements meta-cell construction, meta-cell kNN graph building, path finding, greedy major-trajectory selection, cell insertion, and pseudotime assignment) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Computes the RNA velocity matrix (dynamical mode) stored in adata, the direction signal used to build and orient the meta-cell graph paths)
- **statsmodels** (Optional dependency for downstream differentially expressed gene analysis per trajectory path)

## Examples

```
import CellPath as cp; cellpath_obj = cp.CellPath(adata=adata, preprocess=True); cellpath_obj.all_in_one(num_metacells=num_metacells, n_neighs=10, pruning=False, num_trajs=7, insertion=True, prop_insert=0.50)
```

## Evaluation signals

- The expected number of trajectories is output: e.g., 7 trajectories for the pancreatic endocrinogenesis dataset with num_trajs=7
- cellpath_obj.pseudo_order exists and contains pseudotime plus branching assignment for cells in each path
- DE analysis per path recovers expected lineage markers: Pcsk2/Ero1lb/Cpe (beta path 0), Pax4 (delta path 1), Arx (alpha path 4), and cell-cycle genes Kif23/Clspn/Aurkb/Spc24
- Paths are directed automatically (no user-supplied root) and cover multiple topologies without pre-defined trajectory structure
- Unassigned cells are incorporated up to the configured prop_insert fraction (0.50)

## Limitations

- Performance is affected by accuracy of RNA velocity inference; noise in estimated velocity propagates to trajectories
- No global-view alignment of locally detected paths is implemented yet (noted as a future direction)
- Connecting points between paths are not yet implemented (noted as a future direction)

## Evidence
