---
name: knn-graph-trajectory-path-finding
description: Use when snapshot scRNA-seq data (with scVelo/velocyto spliced-unspliced
  velocity) contains multiple or branching trajectories whose direction and topology
  cannot be resolved by expression-only TI methods.
when_to_use_negative:
- Velocity estimates are unreliable or unavailable — the paper notes all RNA velocity-based
  methods are limited by the accuracy of RNA velocity inference
- Only a single pre-defined linear backbone is needed and expression-based methods
  (e.g., Slingshot, PAGA) suffice
- A cluster-level global trajectory graph is sufficient; Directed-PAGA-style outputs
  are more appropriate than high-resolution local paths
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CellPath
  role: Implements meta-cell construction, meta-cell k-nn graph building, path finding,
    greedy trajectory selection, and first-order pseudotime reconstruction
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Computes stochastic-mode RNA velocity from spliced/unspliced counts during
    CellPath preprocessing (preprocess=True)
- name: statsmodels
  role: Powers the optional GAM likelihood-ratio test for per-trajectory differentially
    expressed gene analysis on recovered paths
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:7b0ccefd03fdc868a3af545f6801f2ce95053f8bc2707e642c834259c79cf893
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# knn-graph-trajectory-path-finding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs k-nearest-neighbor graphs on smoothed meta-cells derived from scRNA-seq counts and RNA velocity, then enumerates and greedily selects major trajectories to infer high-resolution, directed cell paths with pseudotime — the core trajectory inference engine of CellPath.

## When to use

Use when snapshot scRNA-seq data (with scVelo/velocyto spliced-unspliced velocity) contains multiple or branching trajectories whose direction and topology cannot be resolved by expression-only TI methods.

## When NOT to use

- Velocity estimates are unreliable or unavailable — the paper notes all RNA velocity-based methods are limited by the accuracy of RNA velocity inference
- Only a single pre-defined linear backbone is needed and expression-based methods (e.g., Slingshot, PAGA) suffice
- A cluster-level global trajectory graph is sufficient; Directed-PAGA-style outputs are more appropriate than high-resolution local paths

## Inputs

- AnnData (.h5ad) object with scRNA-seq count matrix and spliced/unspliced layers for RNA velocity inference
- RNA velocity matrix computed by scVelo or velocyto (computed automatically when preprocess=True)
- Number of meta-cells, number of neighbors (n_neighs), number of trajectories (num_trajs), and insertion settings

## Outputs

- Meta-cell-level major trajectories selected from the candidate path pool (e.g., ~20 greedy-selected paths)
- Cell-level trajectory assignments and pseudotime stored in cellpath_obj.pseudo_order
- Optionally, per-trajectory DE gene tables (e.g., Elane/Mpo on path 0, Cd74 on path 8) written to CSV

## How to apply

Initialize CellPath with cp.CellPath(adata, preprocess=True) so scVelo computes stochastic-mode RNA velocity on the count matrix (e.g., the 6,555 day-4 mouse hematopoiesis in vivo cells). Aggregate cells into meta-cells with cellpath.meta_cell_construction(flavor='leiden'), the clustering flavor recommended for datasets with intricate trajectories, and obtain smoothed per-meta-cell RNA velocity via a regression model. Run cellpath_obj.all_in_one(num_metacells=N, n_neighs=10, pruning=False, num_trajs=20, insertion=True, prop_insert=0.50) to build a symmetric k-nn graph on meta-cells, enumerate a pool of candidate paths, and greedily select the top 20 major trajectories. Assign cell-level trajectories and pseudotime with first-order pseudotime reconstruction, recovering results from cellpath_obj.pseudo_order; set insertion=True with prop_insert=0.50 to incorporate about half of initially unassigned cells. The topology-free greedy selection is what allows parallel routes (e.g., the Neu-like path 0 and DC-like path 8 monocyte routes) to be recovered without predefining a backbone.

## Related tools

- **CellPath** (Implements meta-cell construction, meta-cell k-nn graph building, path finding, greedy trajectory selection, and first-order pseudotime reconstruction) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Computes stochastic-mode RNA velocity from spliced/unspliced counts during CellPath preprocessing (preprocess=True))
- **statsmodels** (Powers the optional GAM likelihood-ratio test for per-trajectory differentially expressed gene analysis on recovered paths)

## Examples

```
cellpath_obj = cp.CellPath(adata, preprocess=True); cellpath_obj.meta_cell_construction(flavor='leiden'); cellpath_obj.all_in_one(num_metacells=200, n_neighs=10, pruning=False, num_trajs=20, insertion=True, prop_insert=0.50)
```

## Evaluation signals

- The run yields approximately 20 greedy-selected major trajectories covering the main biological processes
- cellpath_obj.pseudo_order contains non-null per-cell pseudotime and branching (path) assignments for the expected fraction of cells (insertion=True, prop_insert=0.50)
- On the mouse hematopoiesis benchmark, both parallel monocyte routes are detected: path 0 (Neu-like) and path 8 (DC-like)
- Downstream GAM likelihood-ratio DE analysis recovers expected markers: Elane and Mpo on path 0 and Cd74 on path 8
- Results written to cellpath_hema_results.csv contain the DE table rows for Elane/Mpo (path 0) and Cd74 (path 8)

## Limitations

- Performance of all RNA velocity-based methods, including CellPath, is limited by the accuracy of RNA velocity inference
- Local paths are not yet aligned into a global view; no connecting points between paths are implemented
- VeTra-style 2D projection losses are avoided, but results still depend on meta-cell resolution and n_neighs choices (num_metacells, n_neighs=10)

## Evidence
