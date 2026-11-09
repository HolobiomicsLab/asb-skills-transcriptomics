---
name: pseudotime-branching-assignment
description: Use when you have run CellPath's all_in_one() on scRNA-seq data with
  scVelo-computed velocities and need per-cell pseudotime and branching assignments
  to identify lineage paths (e.g., α/β/δ endocrine fates) or run path-level differential
  expression.
when_to_use_negative:
- The goal is cluster-level trajectory topology only with no need for per-cell pseudotime
  — Directed-PAGA-style cluster graphs may suffice and CellPath's high-resolution
  output would be overly complex.
- RNA velocity estimates are unavailable or expected to be unreliable, since CellPath's
  performance is limited by the accuracy of RNA velocity inference.
- The dataset requires a single pre-defined trajectory topology; traditional TI methods
  restricted to one connected trajectory are the intended scope there, and CellPath's
  multi-path output would need manual interpretation.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: CellPath
  role: Runs the full pipeline (meta-cell construction, kNN graph, path finding, greedy
    trajectory selection, cell-level insertion) and produces pseudo_order with pseudotime
    and branching assignments
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Computes the RNA velocity matrix (dynamical mode) that CellPath uses as input
    during initialization when preprocess=True
- name: statsmodels
  role: Optional dependency for differentially expressed gene analysis on cells stratified
    by the pseudotime/branching assignment
- name: anndata
  role: Provides the AnnData object format carrying counts and velocity matrices that
    CellPath consumes
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:e8c51be463aa96a899c0cb8c63b969a5141983023973b7e4816f8652a7a37e58
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
  - anndata
  - scVelo
  - statsmodels
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# pseudotime-branching-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract cell-level pseudotime and branching/trajectory-path assignments from CellPath after meta-cell kNN graph construction and path selection, by reading the cellpath_obj.pseudo_order result object. This converts RNA-velocity-informed meta-cell-level paths into per-cell pseudotime values and branch labels for downstream lineage and DE analyses.

## When to use

Use when you have run CellPath's all_in_one() on scRNA-seq data with scVelo-computed velocities and need per-cell pseudotime and branching assignments to identify lineage paths (e.g., α/β/δ endocrine fates) or run path-level differential expression.

## When NOT to use

- The goal is cluster-level trajectory topology only with no need for per-cell pseudotime — Directed-PAGA-style cluster graphs may suffice and CellPath's high-resolution output would be overly complex.
- RNA velocity estimates are unavailable or expected to be unreliable, since CellPath's performance is limited by the accuracy of RNA velocity inference.
- The dataset requires a single pre-defined trajectory topology; traditional TI methods restricted to one connected trajectory are the intended scope there, and CellPath's multi-path output would need manual interpretation.

## Inputs

- AnnData (h5ad) object with scRNA-seq count matrix and RNA velocity matrix (from scVelo or velocyto), e.g., fb_clust.h5ad or pancreatic endocrinogenesis data (GEO:GSE132188, 3,696 E15.5 cells)
- CellPath object configured with meta-cell clustering flavor ('k-means' or 'leiden') and all_in_one() parameters (n_neighs=10, num_trajs, insertion=True, prop_insert=0.50)

## Outputs

- cellpath_obj.pseudo_order: per-cell pseudotime and branching/trajectory-path assignment table
- Set of meta-cell-level trajectory paths (num_trajs paths, e.g., 7 or 14) with automatic direction assignment
- Path-stratified cell sets usable for differential expression analysis (e.g., path6_de_genes.csv)

## How to apply

After initializing CellPath with an AnnData object (preprocess=True triggers velocity calculation via scvelo if not already stored in adata), construct meta-cells with flavor='k-means' (or 'leiden' for intricate trajectories) and run all_in_one() with parameters such as num_metacells, n_neighs=10, pruning=False, a dataset-appropriate num_trajs (e.g., 7 for pancreatic endocrinogenesis, 14 for dentate gyrus), insertion=True, and prop_insert=0.50. Retrieve pseudotime and branching assignments from cellpath_obj.pseudo_order, which maps each cell to a trajectory path and its pseudotime coordinate via first-order pseudotime reconstruction. Use these path assignments to stratify cells for downstream analyses, e.g., statsmodels-based differential expression per path to identify lineage markers such as Pcsk2/Ero1lb/Cpe (beta-cell path 0), Pax4 (delta-cell path 1), or Arx (alpha-cell path 4) in the pancreatic endocrinogenesis dataset, or Camk2a/Adcy1 increases along within-population path 6 in the dentate gyrus. Validate by confirming the expected number of trajectories is output and that DE tables contain the expected lineage markers per path.

## Related tools

- **CellPath** (Runs the full pipeline (meta-cell construction, kNN graph, path finding, greedy trajectory selection, cell-level insertion) and produces pseudo_order with pseudotime and branching assignments) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Computes the RNA velocity matrix (dynamical mode) that CellPath uses as input during initialization when preprocess=True)
- **statsmodels** (Optional dependency for differentially expressed gene analysis on cells stratified by the pseudotime/branching assignment)
- **anndata** (Provides the AnnData object format carrying counts and velocity matrices that CellPath consumes)

## Examples

```
cellpath_obj = cp.CellPath(adata=adata, preprocess=True); cellpath_obj.all_in_one(num_metacells=num_metacells, n_neighs=10, pruning=False, num_trajs=7, insertion=True, prop_insert=0.50); pseudo = cellpath_obj.pseudo_order
```

## Evaluation signals

- cellpath_obj.pseudo_order is populated and contains both pseudotime values and branching/trajectory-path assignments for cells.
- The number of output trajectories matches num_trajs (e.g., 7 for pancreatic endocrinogenesis; 14 for dentate gyrus).
- DE analysis per path recovers expected lineage markers: Pcsk2/Ero1lb/Cpe for beta-cell path 0, Pax4 for delta-cell path 1, Arx for alpha-cell path 4, and cell-cycle genes Kif23/Clspn/Aurkb/Spc24.
- Within-population paths (e.g., dentate gyrus path 6) show biologically coherent gradients, e.g., Camk2a and Adcy1 increasing along the path.
- Inserted cells (prop_insert=0.50) are incorporated into trajectories when insertion=True, and path6_de_genes.csv (or equivalent) is a valid CSV containing expected gene rows.

## Limitations

- Performance of all RNA velocity-based methods, including CellPath, is limited by the accuracy of RNA velocity inference; challenges in RNA velocity estimation remain unresolved (Bergen et al., 2021).
- CellPath detects both global and local paths but does not yet align locally detected paths into a global view.
- Connecting points between paths are not yet implemented and are noted as a future direction.

## Evidence
