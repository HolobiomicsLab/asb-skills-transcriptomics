---
name: nearest-neighbor-graph-construction
description: Use when you need a cell-cell similarity network whose neighbor count
  adapts to local neighborhood density and heterogeneity (e.g., after computing multilevel
  archetype encodings H*), without pre-specifying k.
when_to_use_negative:
- You need a fixed-k nearest-neighbor graph with a uniform neighbor count for all
  cells — the density-adaptive k* deliberately varies neighbor counts per cell.
- The data has not been reduced to the archetypal (H*) space yet — distances should
  be computed in that space, not on raw or merely PCA-reduced expression.
- You are clustering expression directly without ACTIONet's multilevel archetypal
  decomposition; k*-NN here presupposes the ACTION/decomposition outputs.
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ACTIONet (Python package)
  role: Provides reduce_kernel to obtain the H* space and the pipeline (pp.filter_adata,
    pp.reduce_kernel) feeding the k*-NN network construction; installed from the python-devel
    branch.
  repo: https://github.com/shmohammadi86/ACTIONet
- name: ACTIONet
  role: Framework whose adaptive nearest-neighbor (k*-NN) multiresolution network
    construction step this skill implements.
  repo: https://github.com/shmohammadi86/ACTIONet
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:fbf5b886fdfbffbe29a0d997995f30d73ed6ee2969d4ad7882477bb2106b395c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (Python package)
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nearest-neighbor-graph-construction

## Summary

Constructs the ACTIONet cell–cell similarity network with a density-dependent adaptive nearest-neighbor algorithm (k*-NN), where each cell's neighbor count k* is solved from a sparsity parameter (kappa) rather than fixed in advance, over square-root Jensen–Shannon distances in the reduced archetypal (H*) space.

## When to use

Use when you need a cell-cell similarity network whose neighbor count adapts to local neighborhood density and heterogeneity (e.g., after computing multilevel archetype encodings H*), without pre-specifying k.

## When NOT to use

- You need a fixed-k nearest-neighbor graph with a uniform neighbor count for all cells — the density-adaptive k* deliberately varies neighbor counts per cell.
- The data has not been reduced to the archetypal (H*) space yet — distances should be computed in that space, not on raw or merely PCA-reduced expression.
- You are clustering expression directly without ACTIONet's multilevel archetypal decomposition; k*-NN here presupposes the ACTION/decomposition outputs.

## Inputs

- Preprocessed scRNA-seq count matrix (e.g., 10x Genomics .h5 such as pbmc_10k_v3.h5)
- Reduced archetypal metric cell space: normalized multilevel encoding H* from an.pp.reduce_kernel(adata)
- Sparsity parameter kappa controlling the density-adaptive neighbor count

## Outputs

- Sparse k*-NN adjacency matrix with square-root-JSD edge weights, serialized as kstar_nn_graph.npz
- Multiresolution cell–cell network in which cells are nodes positioned by their archetypal composition

## How to apply

First produce the reduced archetypal metric space: load counts (e.g., pbmc_10k_v3.h5 via sc.read_10x_h5), call an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000), apply sc.pp.normalize_total and sc.pp.log1p, then run an.pp.reduce_kernel(adata) from the ACTIONet Python package (python-devel branch) to obtain the normalized multilevel encoding H*. Next compute pairwise square-root Jensen–Shannon distances between cells in this H* space, since the archetypal space is what enables measuring cell distances. Then solve the optimal per-node neighbor-selection problem with sparsity parameter kappa so each cell receives a density-adaptive neighbor count k*, reflecting both proximity and the heterogeneity/density of each cell's neighborhood. Finally assemble the k*-NN edges into a sparse adjacency matrix retaining the square-root-JSD edge weights (off-diagonal weights in (0, 1]) and serialize it, e.g., to kstar_nn_graph.npz.

## Related tools

- **ACTIONet (Python package)** (Provides reduce_kernel to obtain the H* space and the pipeline (pp.filter_adata, pp.reduce_kernel) feeding the k*-NN network construction; installed from the python-devel branch.) — https://github.com/shmohammadi86/ACTIONet
- **ACTIONet** (Framework whose adaptive nearest-neighbor (k*-NN) multiresolution network construction step this skill implements.) — https://github.com/shmohammadi86/ACTIONet

## Examples

```
an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000); sc.pp.normalize_total(adata); sc.pp.log1p(adata); an.pp.reduce_kernel(adata)
```

## Evaluation signals

- kstar_nn_graph.npz exists and loads as a valid sparse matrix.
- Every row of the adjacency matrix contains at least the minimum kappa-derived neighbor count.
- All off-diagonal edge weights lie in the interval (0, 1], consistent with square-root-JSD weights.
- The pipeline runs end-to-end from pbmc_10k_v3.h5 through filter_adata, normalize_total, log1p, and reduce_kernel without errors.
- Neighbor counts are not uniform across cells — per-cell k* varies with local neighborhood density and heterogeneity.

## Limitations

- No software/tool version numbers are stated in the article for ACTIONet, Python, or the k*-NN implementation (only CMake 3.19 is versioned).
- Datasets are cited only by reference numbers without accessions or download links.
- The neighbor-selection problem depends on the sparsity parameter kappa; the article context does not state how kappa is chosen automatically.

## Evidence

- [discussion] adaptive k*-NN network construction step: "an adaptive nearest-neighbor algorithm to build a multiresolution network with automatic neighbor size selection"
- [discussion] distance metric / archetypal space: "an archetypal-based metric cell space construction that enables measuring cell distances"
- [methods] reduce_kernel call producing H*: "an.pp.reduce_kernel(adata)"
- [readme] network structure of ACTIONet output: "cells are nodes positioned by their archetypal composition, with a matching low-dimensional embedding for visualisation"
- [readme] why adaptive neighbor size (vs fixed-k clustering): "the resolution at which you cluster decides what you are able to see"
