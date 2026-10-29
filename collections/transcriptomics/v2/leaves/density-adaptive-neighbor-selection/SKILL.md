---
name: density-adaptive-neighbor-selection
description: Use when constructing a single-cell cell–cell network from archetypal/multilevel
  decomposition outputs (H*) and you need neighborhood sizes to adapt to local density
  and heterogeneity rather than fixing k in advance.
when_to_use_negative:
- Input is raw or preprocessed gene-expression data and no archetypal decomposition
  (H*) has been computed yet — the k*-NN metric is defined in the archetypal cell
  space, not gene space
- You need a standard fixed-k graph (e.g., scRNA kNN used by standard Scanpy clustering)
  — k*-NN explicitly removes the fixed-neighbor-count assumption
- Data are chromatin accessibility peaks without prior reduction (LSI/chromVAR/ACTION)
  — the extremely sparse, high-dimensional peak matrix is not a suitable direct input
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ACTIONet (Python package)
  role: Provides an.pp.reduce_kernel to produce the archetypal metric cell space (H*)
    that the k*-NN network is built over; installed from the python-devel branch
  repo: https://github.com/shmohadi86/ACTIONet
- name: ACTIONet
  role: Framework whose adaptive nearest-neighbor (k*-NN) step performs the density-dependent
    multiresolution network construction
  repo: https://github.com/shmohammadi86/ACTIONet
- name: Python
  role: Runtime language for loading the PBMC h5 matrix with scanpy, computing square-root
    JSD distances, solving the neighbor-selection optimization, and writing the sparse
    graph
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:d9b52e8c5656c2071f4a5ef3125a73d4fb900c2cd00b64d5e49b32312e8898b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (Python package)
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# density-adaptive-neighbor-selection

## Summary

ACTIONet's k*-NN step builds the cell–cell similarity network by solving a per-node optimal neighbor-selection problem in the archetypal metric space (square-root Jensen–Shannon distances over the normalized multilevel encoding H*), so each cell receives a density-adaptive neighbor count k* instead of a fixed k.

## When to use

Use when constructing a single-cell cell–cell network from archetypal/multilevel decomposition outputs (H*) and you need neighborhood sizes to adapt to local density and heterogeneity rather than fixing k in advance.

## When NOT to use

- Input is raw or preprocessed gene-expression data and no archetypal decomposition (H*) has been computed yet — the k*-NN metric is defined in the archetypal cell space, not gene space
- You need a standard fixed-k graph (e.g., scRNA kNN used by standard Scanpy clustering) — k*-NN explicitly removes the fixed-neighbor-count assumption
- Data are chromatin accessibility peaks without prior reduction (LSI/chromVAR/ACTION) — the extremely sparse, high-dimensional peak matrix is not a suitable direct input

## Inputs

- Normalized multilevel encoding matrix H* (reduced archetypal metric cell space) from an.pp.reduce_kernel on a preprocessed .h5ad AnnData object
- Sparsity parameter kappa controlling the minimum density-derived neighbor count

## Outputs

- Sparse cell–cell k*-NN adjacency matrix with square-root-JSD edge weights in (0, 1] (e.g., serialized as kstar_nn_graph.npz)
- Per-cell adaptive neighbor counts k* derived from the optimization with sparsity parameter kappa

## How to apply

First obtain the reduced archetypal metric cell space by calling an.pp.reduce_kernel(adata) on preprocessed PBMC-style data (e.g., pbmc_10k_v3.h5 loaded via sc.read_10x_h5, filtered with an.pp.filter_adata(min_cells_per_feature=0.01, min_features_per_cell=1000), then sc.pp.normalize_total and sc.pp.log1p). Compute pairwise square-root Jensen-Shannon distances between cells in the H* space, since ACTIONet builds its archetypal-based metric cell space specifically to enable measuring cell distances. Then solve the optimal per-node neighbor-selection problem with a sparsity parameter kappa so that each cell's neighborhood heterogeneity and density determine its own neighbor count k*, removing the need to fix the number of neighbors a priori. Finally assemble the k*-NN edges into a sparse adjacency matrix retaining the square-root-JSD edge weights (which lie in (0, 1]) and serialize it, e.g. to kstar_nn_graph.npz. The rationale is that a multiresolution network built with automatic neighbor size selection supports downstream UMAP-style SGD projection and Leiden clustering over the pattern graph.

## Related tools

- **ACTIONet (Python package)** (Provides an.pp.reduce_kernel to produce the archetypal metric cell space (H*) that the k*-NN network is built over; installed from the python-devel branch) — https://github.com/shmohadi86/ACTIONet
- **ACTIONet** (Framework whose adaptive nearest-neighbor (k*-NN) step performs the density-dependent multiresolution network construction) — https://github.com/shmohammadi86/ACTIONet
- **Python** (Runtime language for loading the PBMC h5 matrix with scanpy, computing square-root JSD distances, solving the neighbor-selection optimization, and writing the sparse graph)

## Examples

```
import ACTIONet as an; H_star = an.pp.reduce_kernel(adata)
```

## Evaluation signals

- kstar_nn_graph.npz exists and loads as a valid scipy sparse matrix
- Every row of the adjacency matrix contains at least the minimum kappa-derived neighbor count (no empty rows)
- All off-diagonal edge weights (retained square-root-JSD values) lie strictly in (0, 1]
- The script runs end-to-end from pbmc_10k_v3.h5 preprocessing through an.pp.reduce_kernel to the serialized graph without error

## Limitations

- The number of neighbors per cell is not fixed in advance, so graph density is data-dependent and governed by the choice of sparsity parameter kappa
- Distances must be computed in the archetypal metric space (square-root Jensen-Shannon distance in H*), not on raw expression, or the density-adaptive selection is invalid
- No software/tool versions are stated for ACTIONet, Python, or the Leiden/UMAP implementations used downstream

## Evidence

- [discussion] Adaptive nearest-neighbor (k*-NN) multiresolution network construction: "an adaptive nearest-neighbor algorithm to build a multiresolution network with automatic neighbor size selection"
- [discussion] Archetypal-based metric cell space: "an archetypal-based metric cell space construction that enables measuring cell distances"
- [methods] Pipeline preprocessing thresholds: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [readme] Network definition from README: "The result is a network, the ACTIONet, in which cells are nodes positioned by their
archetypal composition"
- [discussion] Downstream use of the network: "an adaptation of UMAP’s stochastic gradient descent (SGD)-based algorithm to project a multiresolution network into 2 and 3D space"
