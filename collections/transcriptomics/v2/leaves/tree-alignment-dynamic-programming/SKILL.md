---
name: tree-alignment-dynamic-programming
description: Use when two single-cell datasets each have pseudotime trajectories with
  multiple branches (trajectory trees) and a global node-to-node correspondence between
  the trees is needed, e.g. comparing differentiation dynamics across species or conditions.
when_to_use_negative:
- The trajectories are single linear lineages and only a pairwise path comparison
  is needed — DTW-based alignment (tl.dtw) suffices and is simpler.
- The inferred structures are complex graphs such as cycles or disconnected graphs
  — CAPITAL in its present form can deal only with trajectory trees.
- The two datasets have very different global topological structure — alignment accuracy
  degraded in the authors' benchmarking test.
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0089
tools:
- name: CAPITAL
  role: Implements trajectory-tree inference (tl.trajectory_tree), the tree-alignment
    dynamic program (tl.tree_alignment), and packaged synthetic datasets for validation
  repo: https://github.com/ykat0/capital
- name: Python
  role: Implementation language for CAPITAL and for reimplementing/validating the
    Equations 2-8 recursion in isolation
- name: JupyterLab
  role: Interactive development environment in which CAPITAL (Python) can be used
- name: Scanpy
  role: Single-cell analysis toolkit used inside CAPITAL for preprocessing, plotting,
    and datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:16dd8fa75a77dae183a76395dee792252d001371792da5a86cdb1b51e1a5e053
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - JupyterLab
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tree-alignment-dynamic-programming

## Summary

A dynamic-programming tree-alignment method (related to tree edit distance) used by CAPITAL to compute an optimal alignment between two branched pseudotime trajectory trees from different but related scRNA-seq datasets, avoiding manual selection of linear paths.

## When to use

Use when two single-cell datasets each have pseudotime trajectories with multiple branches (trajectory trees) and a global node-to-node correspondence between the trees is needed, e.g. comparing differentiation dynamics across species or conditions.

## When NOT to use

- The trajectories are single linear lineages and only a pairwise path comparison is needed — DTW-based alignment (tl.dtw) suffices and is simpler.
- The inferred structures are complex graphs such as cycles or disconnected graphs — CAPITAL in its present form can deal only with trajectory trees.
- The two datasets have very different global topological structure — alignment accuracy degraded in the authors' benchmarking test.

## Inputs

- Two pseudotime trajectory trees (rooted, bounded-degree, unordered) whose nodes carry gene-expression profiles, e.g. computed from scRNA-seq data as minimum spanning trees over clusters
- Optionally, packaged CAPITAL synthetic datasets (capital.dataset.synthetic_dataset1, synthetic_dataset2) for validation

## Outputs

- Optimal alignment distance (minimum total cost) between the two trajectory trees
- Node mapping (traceback) pairing trajectory-tree nodes across datasets
- Aligned pseudotime kinetics of molecular patterns across experiments

## How to apply

First infer a branched trajectory per dataset as a centroid-based minimum spanning tree over clusters (e.g. via CAPITAL's tl.trajectory_tree), with nodes carrying gene-expression profiles. Then compute the per-node matching cost gamma as 1 minus the Spearman correlation between the expression profiles of candidate matched nodes, and charge a gap penalty delta = 1 for unmatched (deleted/inserted) subtrees. Run the dynamic-programming recursion (Equations 2-8 in the paper) over all node pairs and child sets to obtain the minimum total cost — the optimal alignment distance — and record the argmin traceback as the node mapping (tl.tree_alignment). Validate the recursion in isolation on hand-checkable toy trees (single lineages and simple branchings) before trusting outputs on real data.

## Related tools

- **CAPITAL** (Implements trajectory-tree inference (tl.trajectory_tree), the tree-alignment dynamic program (tl.tree_alignment), and packaged synthetic datasets for validation) — https://github.com/ykat0/capital
- **Python** (Implementation language for CAPITAL and for reimplementing/validating the Equations 2-8 recursion in isolation)
- **JupyterLab** (Interactive development environment in which CAPITAL (Python) can be used)
- **Scanpy** (Single-cell analysis toolkit used inside CAPITAL for preprocessing, plotting, and datasets)

## Examples

```
import capital; adata1 = capital.dataset.synthetic_dataset1(); capital.tl.trajectory_tree(adata1); capital.tl.tree_alignment(adata1, adata2)
```

## Evaluation signals

- On toy trees (single lineage vs. simple branching), the recursion returns the expected hand-checked minimum distance and a consistent node mapping.
- The reported objective equals the sum of per-node gamma costs (1 - Spearman correlation) for matched pairs plus delta = 1 per unmatched subtree.
- The traceback yields a valid tree alignment: matched node pairs are one-to-one and respect ancestor-descendant (branching) structure of both trees.
- On packaged synthetic datasets (capital.dataset.synthetic_dataset1/2), CAPITAL achieves the expected accurate alignment of trajectory trees.
- Output is written to tree_alignment_result with both the distance and the mapping recorded.

## Limitations

- The recursion itself was not validated in isolation in the original publication — the finding explicitly notes 'no validation of the recursion in isolation'.
- The algorithm computes trajectories as minimum spanning trees, and the authors state this 'may not be the best method'.
- Alignment accuracy degrades for datasets with different global topological structure.
- Only trajectory trees (including single lineages) are supported; cycles or disconnected graphs are out of scope and would require graph-aware models plus graph alignment.

## Evidence

- [results] The alignment of branched trajectories is computed by a dynamic programming tree-algorithm: [evidence span withheld — anchor status: not_found]
- [results] with no validation of the recursion in isolation.: [evidence span withheld — anchor status: not_found]
- [discussion] whereas CAPITAL computes Spearman's correlation: "Scanorama and Seurat calculate mutual nearest neighbors in a low-dimensional space, whereas CAPITAL computes Spearman’s correlation"
- [abstract] branching trajectories can be automatically compared: "a method for comparing single-cell trajectories with tree alignment whereby branching trajectories can be automatically compared"
- [discussion] We implemented CAPITAL in Python, which can also be used in the interactive development environment JupyterLab: "We implemented CAPITAL in Python, which can also be used in the interactive development environment JupyterLab"
- [methods] capital.dataset.synthetic_dataset1: "capital.dataset.synthetic\_dataset1"
