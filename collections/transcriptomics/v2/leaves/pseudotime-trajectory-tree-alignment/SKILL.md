---
name: pseudotime-trajectory-tree-alignment
description: Use when two scRNA-seq datasets each have branching pseudotime trajectories
  (multiple lineages) and you need to match corresponding clusters/lineages across
  datasets without pre-selecting which single lineages to compare.
when_to_use_negative:
- The trajectories are simple linear lineages with no branches — existing DTW-based
  linear alignment tools suffice and CAPITAL's tree machinery is unnecessary
- The underlying structures are general graphs with cycles or disconnected components
  — CAPITAL can deal only with trajectory trees computed from the input datasets
- Datasets have different global topological structure — the paper reports alignment
  accuracy degrades in its benchmarking test for such pairs
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: CAPITAL
  role: 'Core package: capital.tl.preprocessing, capital.tl.dpt, capital.tl.trajectory_tree,
    capital.tl.tree_alignment, capital.tl.dtw, capital.tl.genes_similarity_score implement
    the tree-inference and alignment workflow'
  repo: github.com/ykat0/capital
- name: Palantir
  role: Source of the processed human bone marrow anndata (human_cd34_bm_rep1/2/3.h5ad)
    used as capital.dataset.setty19 input for the tree-alignment demonstration
  repo: dpeerlab/Palantir
- name: Scanorama
  role: Integration baseline providing cluster correspondences against which CAPITAL's
    ATC-score alignment accuracy is compared
- name: Seurat
  role: Integration baseline providing cluster correspondences for the ATC-score benchmark
- name: scVI
  role: Integration baseline providing cluster correspondences for the ATC-score benchmark
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_002
  - task_006
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-33681-3
    title: Alignment of single-cell trajectory trees with CAPITAL
  source_content_hash: sha256:ce20c96701a92969eaf33392a30aa096ca0587468999f7d600ab8796095f6488
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAPITAL
  - JupyterLab
  - Palantir
  - Python
  - Scanorama
  - Seurat
  - conda
  - graphviz
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# pseudotime-trajectory-tree-alignment

## Summary

Align branched pseudotime trajectories from two related scRNA-seq datasets by computing trajectory trees (minimum spanning trees over clusters) and running CAPITAL's dynamic-programming tree alignment, removing the need to hand-select linear paths for DTW.

## When to use

Use when two scRNA-seq datasets each have branching pseudotime trajectories (multiple lineages) and you need to match corresponding clusters/lineages across datasets without pre-selecting which single lineages to compare.

## When NOT to use

- The trajectories are simple linear lineages with no branches — existing DTW-based linear alignment tools suffice and CAPITAL's tree machinery is unnecessary
- The underlying structures are general graphs with cycles or disconnected components — CAPITAL can deal only with trajectory trees computed from the input datasets
- Datasets have different global topological structure — the paper reports alignment accuracy degrades in its benchmarking test for such pairs

## Inputs

- scRNA-seq expression matrices (anndata objects) from two related datasets, e.g. capital.dataset.setty19 (Setty et al. bone marrow replicate 1) and capital.dataset.velten17 (Velten et al., GEO GSE75478), or capital.dataset.synthetic_dataset1/2
- Marker-gene lists for downstream DTW comparison of pseudotime kinetics

## Outputs

- Trajectory tree per dataset (minimum spanning tree over clusters) from capital.tl.trajectory_tree
- Optimal tree alignment distance and node/cluster mapping between the two trees from capital.tl.tree_alignment
- DTW-aligned marker-gene pseudotime expression trends from capital.tl.dtw
- Benchmark metrics: ATC score and normalized alignment distance per dataset pair

## How to apply

Set up a Python 3.8 conda environment with graphviz and install CAPITAL via 'pip install capital'. For each dataset, run capital.tl.preprocessing (MAGIC denoising, PCA, k-NN), compute pseudotime with capital.tl.dpt, and build the trajectory tree as a centroid-based minimum spanning tree over clusters with capital.tl.trajectory_tree. Align the two trees with capital.tl.tree_alignment, which runs the dynamic-programming recursion with per-node cost gamma = 1 minus Spearman correlation of node expression profiles and a gap penalty delta = 1 for unmatched subtrees, returning the optimal alignment distance and node mapping. Follow up with capital.tl.dtw to compare marker-gene pseudotime kinetics along the aligned lineages. For benchmarks, score alignments with the ATC score and normalized alignment distance, and compare CAPITAL against Scanorama/scVI/Seurat-derived cluster correspondences using one-sided Wilcoxon signed-rank tests.

## Related tools

- **CAPITAL** (Core package: capital.tl.preprocessing, capital.tl.dpt, capital.tl.trajectory_tree, capital.tl.tree_alignment, capital.tl.dtw, capital.tl.genes_similarity_score implement the tree-inference and alignment workflow) — github.com/ykat0/capital
- **Palantir** (Source of the processed human bone marrow anndata (human_cd34_bm_rep1/2/3.h5ad) used as capital.dataset.setty19 input for the tree-alignment demonstration) — dpeerlab/Palantir
- **Scanorama** (Integration baseline providing cluster correspondences against which CAPITAL's ATC-score alignment accuracy is compared)
- **Seurat** (Integration baseline providing cluster correspondences for the ATC-score benchmark)
- **scVI** (Integration baseline providing cluster correspondences for the ATC-score benchmark)

## Examples

```
conda create -n capital python=3.8 graphviz && conda activate capital && pip install capital
```

## Evaluation signals

- On synthetic pairs, the ATC score of CAPITAL's alignment is statistically significantly higher than integration baselines (one-sided Wilcoxon signed-rank p < 1.00e-226 vs Scanorama, scVI, Seurat across 2,278 pairs)
- Normalized alignment distance stays stable for Gaussian noise with SD below 3.0 and deteriorates at SD >= 3.0 (noise robustness check)
- On bone marrow data, the aligned trajectory tree reproduces expected leaf matches: erythrocytes, megakaryocytes, pre-B-cells, and dendritic cells across Setty et al. and Velten et al.
- DTW-aligned marker-gene plots show similar expression dynamics along pseudotime between the two datasets
- capital.tl.tree_alignment runs faster than the summed all-against-all capital.tl.dtw alignment of all single lineages (root-to-leaf paths) extracted from the trees

## Limitations

- CAPITAL in its present form can deal only with trajectory trees (including single lineages) and not complex graphs such as cycles or disconnected graphs
- Part of the CAPITAL algorithm for computing a trajectory as a minimum spanning tree may not be the best method
- Alignment accuracy degrades for datasets with different global topological structure (e.g. removing the root with two or three children increases normalized alignment distance)
- Denoising and imputing expression counts can generate more predicted clusters than previously reported, including intermediate states; anchor-based data integration can wrongly match cell subpopulations

## Evidence

- [abstract] CAPITAL presented: "Here we present CAPITAL (comparative analysis of pseudotime trajectory inference with tree alignment), a method for comparing single-cell trajectories with tree alignment"
- [methods] environment setup: "conda create -n capital python=3.8 graphviz"
- [methods] tree alignment module: "   tl.trajectory_tree
   tl.tree_alignment
   tl.dpt
   tl.dtw"
- [discussion] speed advantage: "CAPITAL can solve it de novo in a global fashion and run faster than the all-against-all alignment of the single lineages"
- [readme] tree-alignment scope: "We present CAPITAL, a computational method for comparing pseudotime trajectories with tree alignment whereby trajectories including branches can be automatically compared."
