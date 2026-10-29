---
name: jensen-shannon-distance-computation
description: 'Use when constructing a cell-cell similarity network from ACTIONet multilevel
  decomposition outputs: cells are represented as convex archetype-mixing weight vectors
  and a distribution-aware, bounded distance is needed for density-dependent k*-NN
  edge selection.'
when_to_use_negative:
- Input is raw or only library-size-normalized gene expression (not the reduced H*
  archetype-weight space) — distances should be computed after an.pp.reduce_kernel,
  not directly on counts
- The goal is a fixed-k nearest-neighbor graph (e.g. standard scanpy neighbors) rather
  than ACTIONet's density-adaptive k*-NN multiresolution network
- Cell representations are not nonnegative mixture weights over archetypes, since
  Jensen-Shannon divergence presumes probability-like distributions
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ACTIONet (Python package)
  role: Provides an.pp.reduce_kernel, which produces the reduced archetypal metric
    cell space H* in which the square-root JSD distances are computed
  repo: https://github.com/shmohammadi86/ACTIONet
- name: ACTIONet
  role: Framework whose adaptive k*-NN multiresolution network construction consumes
    the square-root JSD distances as edge weights
  repo: https://github.com/shmohammadi86/ACTIONet
- name: Python
  role: Implementation environment for computing pairwise square-root JSD and assembling
    the sparse k*-NN adjacency
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:3043bd71265940921ef67786aa1eca20aac012c6722491674c7e9bcae0e645b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (Python package)
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# jensen-shannon-distance-computation

## Summary

Compute pairwise square-root Jensen-Shannon distances between cells in ACTIONet's reduced archetypal metric space (the normalized multilevel encoding H*) so that the subsequent density-adaptive k*-NN network is built on a bounded, probability-simplex-aware metric rather than Euclidean distance in gene space.

## When to use

Use when constructing a cell-cell similarity network from ACTIONet multilevel decomposition outputs: cells are represented as convex archetype-mixing weight vectors and a distribution-aware, bounded distance is needed for density-dependent k*-NN edge selection.

## When NOT to use

- Input is raw or only library-size-normalized gene expression (not the reduced H* archetype-weight space) — distances should be computed after an.pp.reduce_kernel, not directly on counts
- The goal is a fixed-k nearest-neighbor graph (e.g. standard scanpy neighbors) rather than ACTIONet's density-adaptive k*-NN multiresolution network
- Cell representations are not nonnegative mixture weights over archetypes, since Jensen-Shannon divergence presumes probability-like distributions

## Inputs

- Preprocessed scRNA-seq count matrix (e.g. pbmc_10k_v3.h5 read via scanpy, filtered and log1p-normalized)
- Reduced archetypal metric cell space H*: the normalized multilevel encoding produced by an.pp.reduce_kernel(adata)
- Sparsity parameter kappa controlling the minimum/adaptive neighbor count

## Outputs

- Pairwise square-root Jensen-Shannon distance matrix between cells in the H* space
- Sparse k*-NN adjacency matrix (kstar_nn_graph.npz) with square-root-JSD edge weights in (0,1]

## How to apply

First produce the reduced archetypal metric cell space by calling an.pp.reduce_kernel(adata) on the preprocessed (filter_adata with min_cells_per_feature=0.01 and min_features_per_cell=1000, normalize_total, log1p) PBMC-style 10x count matrix, yielding the normalized multilevel encoding H*. Treat each cell's row of H* as a probability-like distribution over archetypes, then compute the pairwise square-root Jensen-Shannon distance between all cell pairs; the square root is taken so the metric is a true metric with values in (0,1]. Next, solve the per-node optimal neighbor-selection problem with sparsity parameter kappa so each cell receives a density-adaptive neighbor count k* that reflects local neighborhood heterogeneity and density. Finally, assemble the k*-NN edges into a sparse adjacency matrix retaining the square-root-JSD edge weights (e.g. serialized to kstar_nn_graph.npz). The rationale, per the paper, is that cell distance is measured in an archetypal-based metric cell space, and neighbor counts adapt automatically instead of being fixed in advance.

## Related tools

- **ACTIONet (Python package)** (Provides an.pp.reduce_kernel, which produces the reduced archetypal metric cell space H* in which the square-root JSD distances are computed) — https://github.com/shmohammadi86/ACTIONet
- **ACTIONet** (Framework whose adaptive k*-NN multiresolution network construction consumes the square-root JSD distances as edge weights) — https://github.com/shmohammadi86/ACTIONet
- **Python** (Implementation environment for computing pairwise square-root JSD and assembling the sparse k*-NN adjacency)

## Examples

```
H_star = an.pp.reduce_kernel(adata)  # then compute pairwise sqrt-JSD in H* for k*-NN edges
```

## Evaluation signals

- kstar_nn_graph.npz exists and loads as a valid scipy sparse matrix
- Every row of the adjacency contains at least the minimum kappa-derived neighbor count k*
- All off-diagonal edge weights lie strictly in (0, 1], consistent with square-root JSD boundedness
- Diagonal entries are zero (no self-edges retained in the off-diagonal weight check)
- Distances are computed on rows of H* that are normalized archetype-mixing distributions (multilevel encoding), not on raw or log-normalized gene expression

## Limitations

- Jensen-Shannon distances are only meaningful in the reduced archetypal metric cell space; the article states no software/tool versions for ACTIONet, so package/branch drift (python-devel branch) may change behavior
- Neighbor counts are density-adaptive via kappa rather than fixed, so graph sparsity varies with local neighborhood heterogeneity and cannot be preset exactly
- Datasets cited in the article are referenced without accessions or download links, complicating exact reproduction

## Evidence

- [discussion] Archetypal-based metric cell space construction is the distance substrate: "an archetypal-based metric cell space construction that enables measuring cell distances"
- [discussion] Adaptive neighbor selection consumes the distances: "an adaptive nearest-neighbor algorithm to build a multiresolution network with automatic neighbor size selection"
- [methods] Preprocessing that must precede reduce_kernel in the PBMC quick-start: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [readme] ACTIONet output is a network of cells positioned by archetypal composition: "The result is a network, the ACTIONet, in which cells are nodes positioned by their archetypal composition, with a matching low-dimensional embedding for visualisation."
