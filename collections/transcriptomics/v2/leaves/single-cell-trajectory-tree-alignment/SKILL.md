---
name: single-cell-trajectory-tree-alignment
description: Use when two scRNA-seq datasets each contain branching pseudotime trajectories
  and you need to globally compare their cell-state transition trees, or benchmark
  how topological/clustering changes affect alignment.
when_to_use_negative:
- The trajectories contain complex graph structures such as cycles or disconnected
  graphs — CAPITAL can only deal with trajectory trees computed from the input datasets
- Only a single linear lineage needs comparison and a simple dynamic time warping
  alignment suffices, since CAPITAL is designed for branched trajectories
- The datasets have no expected relation, so global tree alignment of cell-state transitions
  is meaningless
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: CAPITAL
  role: Core package providing tl.preprocessing, tl.trajectory_tree, and tl.tree_alignment
    to infer and align trajectory trees
  repo: https://github.com/ykat0/capital
- name: Python 3.8
  role: Runtime environment for CAPITAL (conda create -n capital python=3.8 graphviz)
- name: conda
  role: Environment manager used to create the CAPITAL environment with graphviz dependency
- name: JupyterLab
  role: Interactive development environment in which CAPITAL (implemented in Python)
    can be used
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:e67b17045b8f1eb5708fcda790df7ca92e77b527f2fa4c3f115cfda593db3ccc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - JupyterLab
  - Python
  - Python 3.8
  - conda
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-trajectory-tree-alignment

## Summary

Aligns branched pseudotime trajectory trees from two different but related scRNA-seq datasets using CAPITAL's tree-alignment algorithm, removing the need to pre-select linear lineage paths. Alignment quality is quantified by normalized alignment distance, enabling benchmarking of topological similarity, clustering dissimilarity, and clustering resolution effects.

## When to use

Use when two scRNA-seq datasets each contain branching pseudotime trajectories and you need to globally compare their cell-state transition trees, or benchmark how topological/clustering changes affect alignment.

## When NOT to use

- The trajectories contain complex graph structures such as cycles or disconnected graphs — CAPITAL can only deal with trajectory trees computed from the input datasets
- Only a single linear lineage needs comparison and a simple dynamic time warping alignment suffices, since CAPITAL is designed for branched trajectories
- The datasets have no expected relation, so global tree alignment of cell-state transitions is meaningless

## Inputs

- Preprocessed scRNA-seq expression matrices (AnnData objects) from two related datasets (e.g., paul15, setty19, velten17, or synthetic_dataset1/synthetic_dataset2)
- Clustered cells per dataset with inferred branched pseudotime trajectories (trajectory trees as MSTs over clusters)

## Outputs

- Optimal tree alignment between the two trajectory trees
- Normalized alignment distance per dataset pair (e.g., table saved to synthetic_alignment_distances.csv grouped by benchmark condition)

## How to apply

Install CAPITAL in a conda environment (Python 3.8/3.9 with graphviz, then 'pip install capital'). Preprocess each expression matrix with capital.tl.preprocessing, then infer a branched trajectory per dataset as a minimum spanning tree over clusters with capital.tl.trajectory_tree. Compute the optimal alignment between the two trajectory trees with capital.tl.tree_alignment and record the normalized alignment distance; CAPITAL computes Spearman's correlation between cells (rather than mutual nearest neighbors as in Scanorama/Seurat) to score cluster correspondence. For benchmarking, generate synthetic tree pairs spanning topological-similarity (e.g., root removal with two or three children), clustering-similarity, and clustering-resolution conditions, collect distances grouped by condition, and write e.g. synthetic_alignment_distances.csv. Rationale: the method solves cluster alignment de novo in a global fashion, avoiding the all-against-all alignment of single lineages required by dynamic time warping-based approaches.

## Related tools

- **CAPITAL** (Core package providing tl.preprocessing, tl.trajectory_tree, and tl.tree_alignment to infer and align trajectory trees) — https://github.com/ykat0/capital
- **Python 3.8** (Runtime environment for CAPITAL (conda create -n capital python=3.8 graphviz))
- **conda** (Environment manager used to create the CAPITAL environment with graphviz dependency)
- **JupyterLab** (Interactive development environment in which CAPITAL (implemented in Python) can be used)

## Examples

```
conda create -n capital python=3.8 graphviz && conda activate capital && pip install capital
```

## Evaluation signals

- The saved distance table (e.g., synthetic_alignment_distances.csv) exists and contains one normalized alignment distance per benchmarked pair (53,987 topological-similarity, 6,800 clustering-similarity, 40,800 clustering-resolution pairs)
- Pairs with matching global topological structure yield low/robust alignment distances
- Disruptions that destroy global topological similarity (e.g., removing the root with two or three children) visibly increase the normalized alignment distance
- Alignment distances degrade in a controlled, monotonic way as clustering resolution or clustering dissimilarity conditions diverge
- The pipeline reproduces the paper's qualitative finding that alignment accuracy is robust within matching topological structure but degrades for datasets with different global topological structure

## Limitations

- CAPITAL computes trajectory trees as minimum spanning trees over clusters, and part of this algorithm may not be the best method
- CAPITAL in its present form can deal only with trajectory trees (including single lineages) and not complex graphs such as cycles or disconnected graphs; handling graphs beyond trees plus graph alignment would be required for complex comparative pseudotime analysis
- Alignment accuracy of CAPITAL degrades for datasets with different global topological structure
- Anchor-based data integration alternatives can wrongly match cell subpopulations, producing integrated clusters with imbalanced numbers of cells

## Evidence

- [abstract] CAPITAL achieved accurate and robust alignments of trajectory trees: "Computational tests on synthetic datasets and authentic bone marrow cells datasets indicate that CAPITAL has achieved accurate and robust alignments of trajectory trees"
- [methods] capital.tree_alignment API: "   tl.trajectory_tree
   tl.tree_alignment"
- [methods] conda install: "Create a new environment for CAPITAL (recommended)::"
- [discussion] MST-based trajectory computation: "part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method"
- [discussion] Spearman correlation vs MNN: "Scanorama and Seurat calculate mutual nearest neighbors in a low-dimensional space, whereas CAPITAL computes Spearman’s correlation"
- [readme] README installation instructions: [evidence span withheld — anchor status: external_doc]
- [readme] README Code Ocean capsule: [evidence span withheld — anchor status: external_doc]
