---
name: rooted-tree-data-structure-implementation
description: Use when the analysis requires representing branched single-cell pseudotime
  trajectories as rooted trees with node-level expression profiles so they can be
  compared via tree alignment rather than linear dynamic time warping.
when_to_use_negative:
- The trajectories are known to be single linear lineages only — dynamic time warping
  (tl.dtw) suffices and tree alignment is unnecessary overhead
- The trajectories contain cycles or disconnected graphs — CAPITAL in its present
  form can deal only with trajectory trees, not complex graphs
- The two datasets have very different global topological structure — CAPITAL's alignment
  accuracy degrades in its benchmarking test for such cases
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: CAPITAL
  role: Provides tl.trajectory_tree to compute rooted trajectory trees as minimum
    spanning trees from cluster centroids, and tl.tree_alignment to align them; installed
    via pip in a Python>=3.8 conda environment
  repo: https://github.com/ykat0/capital
- name: Python
  role: Implementation language for CAPITAL and for custom toy-tree validation of
    the tree-alignment recursion
- name: Scanpy
  role: Underlying single-cell toolkit used in CAPITAL's implementation for preprocessing,
    plotting, and packaged datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:3556f58d367b759c6f0e2541ec9459a8b6bf6f3a77fae82b4a118f35aff708ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# rooted-tree-data-structure-implementation

## Summary

Implement and manipulate rooted, bounded-degree, unordered labeled trees whose nodes carry gene-expression profiles, as required by CAPITAL's trajectory-tree alignment. Each dataset's trajectory is represented as a centroid-based minimum spanning tree over clusters, which becomes the input to dynamic-programming tree alignment.

## When to use

Use when the analysis requires representing branched single-cell pseudotime trajectories as rooted trees with node-level expression profiles so they can be compared via tree alignment rather than linear dynamic time warping.

## When NOT to use

- The trajectories are known to be single linear lineages only — dynamic time warping (tl.dtw) suffices and tree alignment is unnecessary overhead
- The trajectories contain cycles or disconnected graphs — CAPITAL in its present form can deal only with trajectory trees, not complex graphs
- The two datasets have very different global topological structure — CAPITAL's alignment accuracy degrades in its benchmarking test for such cases

## Inputs

- Preprocessed scRNA-seq datasets (Scanpy AnnData objects) with clusters and inferred pseudotime
- Cluster centroid coordinates from which the minimum spanning trajectory tree is computed
- Small hand-checkable rooted test trees with nodes carrying gene-expression profiles (from capital.dataset.synthetic_dataset1 / synthetic_dataset2)

## Outputs

- Rooted, bounded-degree, unordered trajectory tree per dataset with expression profiles at nodes
- Tree objects consumable by CAPITAL's tl.tree_alignment dynamic-programming recursion
- Validation artifacts recording expected alignment distances and node mappings on toy trees (e.g. tree_alignment_result)

## How to apply

For each scRNA-seq dataset, cluster the cells and compute a minimum spanning tree over cluster centroids to obtain a rooted trajectory tree (CAPITAL's tl.trajectory_tree), ensuring nodes carry gene-expression profiles suitable for Spearman-correlation comparison. Represent each tree as a bounded-degree, unordered rooted structure (e.g. via networkx, capped at networkx<=2.8.3 in the CAPITAL environment) so that per-node candidate matches and child-set pairings can be enumerated for the Equations 2-8 recursion. For validation, construct small hand-checkable toy trees (single lineages and simple branchings), for example from capital.dataset.synthetic_dataset1 and synthetic_dataset2, and charge a gap penalty delta = 1 for unmatched (deleted/inserted) subtrees. The rationale is that dynamic time warping handles only linear paths, so a tree data structure is the prerequisite for global alignment of branching trajectories without pre-selecting lineages.

## Related tools

- **CAPITAL** (Provides tl.trajectory_tree to compute rooted trajectory trees as minimum spanning trees from cluster centroids, and tl.tree_alignment to align them; installed via pip in a Python>=3.8 conda environment) — https://github.com/ykat0/capital
- **Python** (Implementation language for CAPITAL and for custom toy-tree validation of the tree-alignment recursion)
- **Scanpy** (Underlying single-cell toolkit used in CAPITAL's implementation for preprocessing, plotting, and packaged datasets)

## Examples

```
import capital as cap; adata1, adata2 = cap.dataset.synthetic_dataset1(), cap.dataset.synthetic_dataset2(); cap.tl.trajectory_tree(adata1); cap.tl.trajectory_tree(adata2); cap.tl.tree_alignment(adata1, adata2)
```

## Evaluation signals

- On single-lineage toy trees, the tree-alignment recursion returns the same distance as an independent dynamic time warping check (tl.dtw equivalence case)
- On toy trees with a simple branching, the traceback node mapping is consistent: every matched node pair exists in both trees and unmatched subtrees incur exactly the gap penalty delta = 1
- The per-node cost gamma equals 1 minus the Spearman correlation of the two nodes' gene-expression profiles, reproducible with an independent Spearman computation
- The alignment distance is symmetric: aligning tree A to tree B yields the same minimum total cost as B to A
- The built trajectory tree is acyclic, connected, and spans all clusters (a valid minimum spanning tree over centroids)

## Limitations

- CAPITAL can deal only with trajectory trees (including single lineages) and not complex graphs such as cycles or disconnected graphs
- Alignment accuracy degrades for datasets with different global topological structure
- Part of the algorithm for computing a trajectory as a minimum spanning tree may not be the best method
- The dynamic-programming recursion (Equations 2-8) had no isolated validation in the original study

## Evidence

- [abstract] Tree alignment of pseudotime trajectories: "a method for comparing single-cell trajectories with tree alignment whereby branching trajectories can be automatically compared"
- [discussion] MST trajectory trees: "part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method"
- [methods] Tree alignment workflow entry points: [evidence span withheld — anchor status: not_found]
- [discussion] Tree-only scope limitation: "CAPITAL in its present form can deal only with trajectory trees (including single lineages) that are computed from input datasets"
- [readme] Python implementation and environment: "Create a new environment for CAPITAL (recommended):: conda create -n capital python=3.9 graphviz"
