---
name: gap-penalty-scoring-design
description: Use when designing or reimplementing the scoring function for aligning
  branched pseudotime trajectory trees (or any bounded-degree unordered rooted labeled
  trees) via tree-alignment dynamic programming, and you must define match costs and
  deletion/insertion penalties.
when_to_use_negative:
- The input structures are complex graphs with cycles or disconnected components —
  CAPITAL's tree alignment deals only with trajectory trees (including single lineages)
- You need cell-level linear alignment of a single selected path; dynamic time warping
  (CAPITAL's tl.dtw) is the linear alternative
- Datasets have very different global topological structure — the paper reports alignment
  accuracy degrades in its benchmarking test for such cases
tools:
- name: CAPITAL
  role: Implements the trajectory-tree construction and the tree-alignment dynamic
    program whose match cost and gap penalty this skill designs; installed via 'pip
    install capital'
  repo: github.com/ykat0/capital
- name: Python
  role: Language in which CAPITAL (and any reimplementation of the Equations 2-8 recursion
    with the gamma/delta scoring) is written
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:891b6d819596c73d8679be4cff521be7a4ce97113ad83194cafed815aa2b8d61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAPITAL
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gap-penalty-scoring-design

## Summary

Design of the node-cost and gap-penalty scheme used in CAPITAL's tree-alignment dynamic program, where matching two trajectory-tree nodes is scored as 1 minus the Spearman correlation of their gene-expression profiles and unmatched (deleted/inserted) subtrees are charged a fixed gap penalty delta = 1.

## When to use

Use when designing or reimplementing the scoring function for aligning branched pseudotime trajectory trees (or any bounded-degree unordered rooted labeled trees) via tree-alignment dynamic programming, and you must define match costs and deletion/insertion penalties.

## When NOT to use

- The input structures are complex graphs with cycles or disconnected components — CAPITAL's tree alignment deals only with trajectory trees (including single lineages)
- You need cell-level linear alignment of a single selected path; dynamic time warping (CAPITAL's tl.dtw) is the linear alternative
- Datasets have very different global topological structure — the paper reports alignment accuracy degrades in its benchmarking test for such cases

## Inputs

- Two rooted trajectory trees with bounded degree, derived as minimum spanning trees over cluster centroids, whose nodes carry gene-expression profiles (e.g., Scanpy AnnData-derived centroids)
- Precomputed Spearman correlation values between node expression profiles across datasets

## Outputs

- Optimal alignment distance (minimum total cost over match costs gamma and gap penalty delta = 1)
- Node mapping (argmin traceback) aligning nodes of the two trajectory trees
- tree_alignment_result record with the distance and mapping for downstream pseudotime-kinetics comparison

## How to apply

Build a centroid-based minimum spanning tree per scRNA-seq dataset so each trajectory-tree node carries a gene-expression profile (e.g., from CAPITAL's packaged synthetic_dataset1/synthetic_dataset2 or paul15 datasets). Define the per-node cost gamma(candidate pair) as 1 minus the Spearman correlation of the two nodes' expression profiles — this is the analog of CAPITAL computing Spearman's correlation where other integration tools use mutual nearest neighbors. Charge a fixed gap penalty delta = 1 for any unmatched subtree (deleted or inserted), so gap cost is subtree-level rather than per-cell, mirroring the tree-edit-distance formulation of Equations 2-8. Run the dynamic-programming recursion over all node pairs and child sets to obtain the minimum total cost (the optimal alignment distance) and record the argmin traceback as the node mapping. Validate the scoring on hand-checkable toy trees — single lineages and simple branchings — before applying it to real datasets, since the article's recursion was not validated in isolation.

## Related tools

- **CAPITAL** (Implements the trajectory-tree construction and the tree-alignment dynamic program whose match cost and gap penalty this skill designs; installed via 'pip install capital') — github.com/ykat0/capital
- **Python** (Language in which CAPITAL (and any reimplementation of the Equations 2-8 recursion with the gamma/delta scoring) is written)

## Examples

```
conda create -n capital python=3.8 graphviz && conda activate capital && pip install capital
```

## Evaluation signals

- On single-lineage toy trees the recursion returns the expected DTW-like distance for the chosen gamma (1 - Spearman correlation) and delta = 1
- On hand-checkable branching toy trees (e.g., from capital.dataset.synthetic_dataset1/2) the returned distance matches a brute-force enumeration of all possible mappings
- The argmin traceback yields a consistent node mapping: every matched pair has cost 1 - Spearman correlation of its expression profiles, and unmatched subtrees are charged exactly delta = 1
- Total alignment cost is non-negative and equals the sum of matched-node costs plus (number of unmatched subtrees) x 1
- Reproducing the packaged synthetic datasets through CAPITAL's tl.trajectory_tree and tl.tree_alignment gives the expected optimal alignment distance

## Limitations

- The article notes 'no validation of the recursion in isolation' — the tree-alignment recursion and its scoring were not unit-tested independently in the original work
- Alignment accuracy degrades for datasets with different global topological structure
- CAPITAL can deal only with trajectory trees (including single lineages), not complex graphs such as cycles or disconnected graphs; a model handling graphs beyond trees plus graph alignment would be required
- Computing a trajectory as a minimum spanning tree 'may not be the best method', which propagates into the node profiles that the gamma cost scores

## Evidence

- [discussion] scoring via Spearman correlation: [evidence span withheld — anchor status: not_found]
- [intro] tree alignment of branched trajectories: "CAPITAL seeks to infer a pseudotime trajectory that can include multiple branches for each dataset, and then to compute an optimal alignment between the two trajectories."
- [abstract] tree alignment formulation: "a method for comparing single-cell trajectories with tree alignment whereby branching trajectories can be automatically compared"
- [discussion] scope limited to trees: "CAPITAL in its present form can deal only with trajectory trees (including single lineages) that are computed from input datasets"
- [readme] installation of the implementing tool: "We present CAPITAL, a computational method for comparing pseudotime trajectories with tree alignment whereby trajectories including branches can be automatically compared."
