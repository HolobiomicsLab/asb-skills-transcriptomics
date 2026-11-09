---
name: single-cell-batch-integration-benchmarking
description: Use when quantitatively comparing data-integration methods (Scanorama,
  scVI, Seurat) against a trajectory-tree-alignment method on paired single-cell datasets,
  measuring biological variance conservation with the ATC score.
when_to_use_negative:
- Trajectories are simple linear lineages — DTW-based linear alignment tools suffice
  without tree construction.
- Input datasets have different global topological structure, where CAPITAL's benchmarked
  alignment accuracy was reported to degrade.
- The goal is to compare graphs with cycles or disconnected components — CAPITAL handles
  only trajectory trees, not complex graphs.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CAPITAL
  role: Computes trajectory trees (tl.trajectory_tree), tree alignments (tl.tree_alignment,
    tl.dtw) and gene-similarity ATC scores being benchmarked
  repo: https://github.com/ykat0/capital
- name: Scanorama
  role: Data-integration baseline that derives cluster correspondences between synthetic
    dataset pairs via mutual nearest neighbors
- name: scVI
  role: Data-integration baseline used to derive cluster correspondences for each
    synthetic pair
- name: Seurat
  role: Data-integration baseline deriving cluster correspondences via mutual nearest
    neighbors in a low-dimensional space
- name: conda
  role: Environment creation for reproducible benchmark runs (conda create -n capital
    python=3.8 graphviz)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:6ae6694a5d7cafd5b20e0743d030410c0474e5afc766e52bf663a10bb1a7141d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ykat0/capital
  tools:
  - CAPITAL
  - Harmony
  - LIGER
  - MASI
  - Python
  - Scanorama
  - Seurat
  - UnsupervisedTrainer
  - aricode
  - conda
  - fastMNN
  - presto
  - scETM
  - scVAE-GM
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 24
---

# single-cell-batch-integration-benchmarking

## Summary

Benchmark protocol for scoring how well data-integration baselines (Scanorama, scVI, Seurat) versus CAPITAL's tree alignment conserve biological variance of trajectory trees, using the ATC score over 2,278 pairs of synthetic scRNA-seq datasets and one-sided Wilcoxon signed-rank tests (Fig. 4a).

## When to use

Use when quantitatively comparing data-integration methods (Scanorama, scVI, Seurat) against a trajectory-tree-alignment method on paired single-cell datasets, measuring biological variance conservation with the ATC score.

## When NOT to use

- Trajectories are simple linear lineages — DTW-based linear alignment tools suffice without tree construction.
- Input datasets have different global topological structure, where CAPITAL's benchmarked alignment accuracy was reported to degrade.
- The goal is to compare graphs with cycles or disconnected components — CAPITAL handles only trajectory trees, not complex graphs.

## Inputs

- 68 synthetic scRNA-seq datasets loaded via capital.dataset.synthetic_dataset1 and capital.dataset.synthetic_dataset2
- All 2,278 pairs of synthetic datasets (paired adata objects)
- Cluster correspondences produced by Scanorama, scVI, and Seurat integrations for each pair

## Outputs

- ATC alignment-accuracy score distributions for CAPITAL vs. each integration baseline across 2,278 pairs
- One-sided Wilcoxon signed-rank p-values per method comparison (reference: < 1.00e-226)
- ATC-score benchmark figure (Fig. 4a), e.g. fig4a_atc_score_benchmark.png

## How to apply

Create a Python 3.8 conda environment with graphviz and install CAPITAL via pip, then load the 68 synthetic datasets through capital.dataset.synthetic_dataset1 and synthetic_dataset2. Enumerate all 2,278 dataset pairs; for each pair derive cluster correspondences with the integration baselines (Scanorama, scVI, Seurat), and infer a minimum-spanning-tree pseudotime trajectory with capital.tl.trajectory_tree (pseudotime via capital.tl.dpt) followed by capital.tl.tree_alignment and capital.tl.dtw. Score biological variance conservation with capital.tl.genes_similarity_score-style ATC alignment-accuracy scores on the aligned trees for CAPITAL versus each baseline across all pairs. Finally compare the score distributions with one-sided Wilcoxon signed-rank tests; the reference result is CAPITAL significantly outperforming all three baselines with p < 1.00e-226 on 2,278 pairs. A Code Ocean compute capsule is provided to reproduce the paper's results.

## Related tools

- **CAPITAL** (Computes trajectory trees (tl.trajectory_tree), tree alignments (tl.tree_alignment, tl.dtw) and gene-similarity ATC scores being benchmarked) — https://github.com/ykat0/capital
- **Scanorama** (Data-integration baseline that derives cluster correspondences between synthetic dataset pairs via mutual nearest neighbors)
- **scVI** (Data-integration baseline used to derive cluster correspondences for each synthetic pair)
- **Seurat** (Data-integration baseline deriving cluster correspondences via mutual nearest neighbors in a low-dimensional space)
- **conda** (Environment creation for reproducible benchmark runs (conda create -n capital python=3.8 graphviz))

## Examples

```
conda create -n capital python=3.8 graphviz; conda activate capital; pip install capital
```

## Evaluation signals

- All 2,278 synthetic pairs complete each integration/alignment pipeline without failure and yield a finite ATC score per method.
- One-sided Wilcoxon signed-rank tests over the paired score distributions reproduce significance (reference: p < 1.00e-226 for CAPITAL vs. Scanorama, scVI, and Seurat).
- Each pair's trajectory is a tree computed as a minimum spanning tree from clusters, and alignment runs return results from tl.tree_alignment and tl.dtw.
- Reproducibility check against the published Code Ocean compute capsule (https://codeocean.com/capsule/5673663/tree/v1).
- Resulting figure matches the structure of Fig. 4a: paired ATC-score distributions with per-comparison p-values.

## Limitations

- Alignment accuracy of CAPITAL for datasets with different global topological structure degraded in the benchmarking test.
- CAPITAL in its present form deals only with trajectory trees (including single lineages), not graphs with cycles or disconnected components; graph-based predictive models plus graph alignment would be required.
- Data-integration anchor-based transformation can wrongly match cell subpopulations, producing integrated clusters with imbalanced numbers of cells — a caveat when interpreting baseline cluster correspondences.

## Evidence

- [discussion] For example, Scanorama and Seurat calculate mutual nearest neighbors in a low-dimensional space, whereas CAPITAL computes Spearman's correlation: "For example, Scanorama and Seurat calculate mutual nearest neighbors in a low-dimensional space, whereas CAPITAL computes Spearman’s correlation"
- [methods] Create a new environment for CAPITAL (recommended): conda create -n capital python=3.8 graphviz: "Create a new environment for CAPITAL (recommended)::

    conda create -n capital python=3.8 graphviz"
- [discussion] there can be the wrong matching of cell subpopulations such as an integrated cluster with the imbalanced number of cells produced by data integration methods: "there can be the wrong matching of cell subpopulations such as an integrated cluster with the imbalanced number of cells produced by data integration methods"
- [readme] We also provide a Code Ocean compute capsule to reproduce our results.: [evidence span withheld — anchor status: external_doc]
- [discussion] part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method: "part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method"
