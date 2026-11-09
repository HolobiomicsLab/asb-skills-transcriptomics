---
name: normalized-alignment-distance-evaluation
description: Use when comparing branched pseudotime trajectories from scRNA-seq datasets
  (e.g., synthetic trajectory-tree pairs or replicate datasets) and you need a scalar,
  tree-level metric that captures whether two branching cell-state transition structures
  align.
when_to_use_negative:
- Trajectories are linear single lineages only — dynamic time warping-based alignment
  (e.g., capital.tl.dtw) is the simpler choice; tree alignment is for branched structures
- The trajectory structures are complex graphs with cycles or disconnected components
  — CAPITAL in its present form deals only with trajectory trees
- You need cell-level correspondence across datasets (integration) rather than a cluster/tree-level
  alignment score — anchor-based integration tools target a different question
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: CAPITAL
  role: Provides tl.trajectory_tree (minimum spanning tree over clusters) and tl.tree_alignment
    to compute the normalized alignment distance between trajectory trees
  repo: https://github.com/ykat0/capital
- name: Python 3.8
  role: Runtime environment for CAPITAL installation and benchmark execution (conda
    create -n capital python=3.8 graphviz)
  repo: https://github.com/ykat0/capital
- name: conda
  role: Creates the isolated Python environment with graphviz dependency required
    by CAPITAL's tree construction
- name: Palantir
  role: Related trajectory-inference method referenced as prior art for differentiation-trajectory
    modeling; not used to compute the alignment distance itself
  repo: https://github.com/dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:5c5425a220d62df9ed17a31b768baf9540a4c0334ca0b24a107b5d88aa551a0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Palantir
  - Python
  - Python 3.8
  - conda
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# normalized-alignment-distance-evaluation

## Summary

Benchmarking skill for CAPITAL-style trajectory-tree alignment: compute the normalized alignment distance between inferred trajectory trees of paired single-cell datasets to quantify how topological similarity, clustering dissimilarity, and clustering resolution affect alignment accuracy.

## When to use

Use when comparing branched pseudotime trajectories from scRNA-seq datasets (e.g., synthetic trajectory-tree pairs or replicate datasets) and you need a scalar, tree-level metric that captures whether two branching cell-state transition structures align.

## When NOT to use

- Trajectories are linear single lineages only — dynamic time warping-based alignment (e.g., capital.tl.dtw) is the simpler choice; tree alignment is for branched structures
- The trajectory structures are complex graphs with cycles or disconnected components — CAPITAL in its present form deals only with trajectory trees
- You need cell-level correspondence across datasets (integration) rather than a cluster/tree-level alignment score — anchor-based integration tools target a different question

## Inputs

- Pairs of single-cell expression datasets as AnnData objects (synthetic benchmark pairs from capital.dataset.synthetic_dataset1/synthetic_dataset2 or real scRNA-seq datasets)
- Clustered expression matrices prepared by capital.tl.preprocessing

## Outputs

- Normalized alignment distance per trajectory-tree pair (scalar, comparable across pairs)
- Distance table grouped by benchmark condition, e.g., synthetic_alignment_distances.csv

## How to apply

Generate or load pairs of trajectory trees (e.g., via capital.dataset.synthetic_dataset1 and synthetic_dataset2, spanning topological-similarity, clustering-similarity, and clustering-resolution conditions). For each dataset, preprocess the expression matrix with capital.tl.preprocessing, then infer a branched pseudotime trajectory with capital.tl.trajectory_tree, which builds a minimum spanning tree over clusters. Run capital.tl.tree_alignment on each pair and record the normalized alignment distance. Group distances by test condition (e.g., 53,987 topological-similarity pairs, 6,800 clustering-similarity pairs, 40,800 clustering-resolution pairs) and interpret: distances should stay low when the two trees share global topological structure, and rise when topology differs (e.g., root removal with two or three children), which is the expected degradation signal rather than a pipeline failure.

## Related tools

- **CAPITAL** (Provides tl.trajectory_tree (minimum spanning tree over clusters) and tl.tree_alignment to compute the normalized alignment distance between trajectory trees) — https://github.com/ykat0/capital
- **Python 3.8** (Runtime environment for CAPITAL installation and benchmark execution (conda create -n capital python=3.8 graphviz)) — https://github.com/ykat0/capital
- **conda** (Creates the isolated Python environment with graphviz dependency required by CAPITAL's tree construction)
- **Palantir** (Related trajectory-inference method referenced as prior art for differentiation-trajectory modeling; not used to compute the alignment distance itself) — https://github.com/dpeerlab/Palantir

## Examples

```
conda create -n capital python=3.8 graphviz; conda activate capital; pip install capital; python -c "import capital as cp; adata1=cp.dataset.synthetic_dataset1(); cp.tl.preprocessing(adata1); cp.tl.trajectory_tree(adata1); d=cp.tl.tree_alignment(adata1,adata1)"
```

## Evaluation signals

- The saved distance table (e.g., synthetic_alignment_distances.csv) exists and contains exactly one normalized distance per benchmarked pair
- Pairs within matching topological structure yield low normalized alignment distances (robust alignment), consistent with the paper's qualitative finding
- Topological disruptions such as removing the root with two or three children measurably increase the normalized alignment distance
- Distances are computed for all benchmark conditions: 53,987 topological-similarity pairs, 6,800 clustering-similarity pairs, and 40,800 clustering-resolution pairs
- Each input dataset successfully yields a trajectory tree via capital.tl.trajectory_tree before alignment (no missing MST/cluster output)

## Limitations

- Alignment accuracy degrades for datasets with different global topological structure, so distance increases across differing topologies reflect method limits, not necessarily pipeline errors
- Computing a trajectory as a minimum spanning tree may not be the best method, which can affect the tree used for alignment
- CAPITAL in its present form can deal only with trajectory trees (including single lineages) and not complex graphs such as cycles or disconnected graphs

## Evidence

- [discussion] Alignment accuracy degrades for datasets with different global topological structure: "alignment accuracy of CAPITAL for datasets with different global topological structure degraded in our benchmarking test"
- [discussion] Trajectory trees as minimum spanning trees: "part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method"
- [discussion] CAPITAL tree-only scope: "CAPITAL in its present form can deal only with trajectory trees (including single lineages) that are computed from input datasets"
- [readme] Installation command from README: [evidence span withheld — anchor status: external_doc]
