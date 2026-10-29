---
name: permutation-matrix-optimization
description: Use when a weighted cluster interlacement graph (CI-graph) of cell-type
  spatial relationships and a color-difference target matrix (CD-graph) must be matched
  to assign contrasting colors to neighboring cell types.
when_to_use_negative:
- When no spatial context is available — the optimization is only meaningful for spatially
  resolved (SRT) data where interlacement between cell types can be quantified
- When users have pre-specified fixed colors for cell types that must not be permuted
  by the optimizer (Spaco only optimizes the remaining cell types)
tools:
- name: Spaco
  role: Implements the graph-guided cluster-color assignment module that optimizes
    the permutation matrix between CI-graph and CD-graph
  repo: github.com/BrainStOrmics/Spaco
- name: Python
  role: Reference implementation language for the stochastic optimization
- name: numpy
  role: Matrix operations for permutation evaluation of the Frobenius objective; version
    1.22.x/1.23.x required
- name: scipy
  role: Numerical routines supporting permutation evaluation and refinement
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:e69a82396158f646590f12a1eb6e16f46e84ef90a923d3da42e24c82f6baf9bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Spaco
  - numpy
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# permutation-matrix-optimization

## Summary

A coarse-to-fine stochastic optimization that finds a one-to-one cluster-to-color permutation matrix P minimizing ||P A_CI P^T − A_CD||_F, so that heavily interlaced cell-type clusters in spatially resolved transcriptomics data receive maximally distinct colors.

## When to use

Use when a weighted cluster interlacement graph (CI-graph) of cell-type spatial relationships and a color-difference target matrix (CD-graph) must be matched to assign contrasting colors to neighboring cell types.

## When NOT to use

- When no spatial context is available — the optimization is only meaningful for spatially resolved (SRT) data where interlacement between cell types can be quantified
- When users have pre-specified fixed colors for cell types that must not be permuted by the optimizer (Spaco only optimizes the remaining cell types)

## Inputs

- CI-graph adjacency matrix (weighted cluster interlacement graph built via the Degree of Interlacement metric)
- Initial color palette used to build the color-difference target matrix A_CD

## Outputs

- Optimized cluster-to-color permutation matrix P (one-hot rows and columns)
- Frobenius loss trace showing monotonic decrease of L(P)
- color_mapping object consumed by spaco.colorize

## How to apply

Load the CI-graph adjacency matrix built from the Degree of Interlacement (DOI) metric and construct the color-difference target matrix A_CD from the selected palette. Sample 2000 random permutations of the cluster-to-color assignment as seeds, then evaluate the Frobenius-norm objective L(P) = ||P A_CI P^T − A_CD||_F for each seed with numpy/scipy, retaining the best permutation. Refine the best assignment via pairwise color-swap fine-tuning capped at 5000 iterations with a stall stop after 100 non-improving iterations. Finally, verify that the returned P is a valid permutation matrix (one-hot rows and columns) and that the recorded Frobenius loss decreases monotonically; pin numpy to 1.22.x or 1.23.x since the numpy version influences the graph-guided mode of Spaco.

## Related tools

- **Spaco** (Implements the graph-guided cluster-color assignment module that optimizes the permutation matrix between CI-graph and CD-graph) — github.com/BrainStOrmics/Spaco
- **Python** (Reference implementation language for the stochastic optimization)
- **numpy** (Matrix operations for permutation evaluation of the Frobenius objective; version 1.22.x/1.23.x required)
- **scipy** (Numerical routines supporting permutation evaluation and refinement)

## Examples

```
color_mapping = spaco.colorize(...)
```

## Evaluation signals

- Returned P is a valid permutation matrix: rows and columns are one-hot vectors summing to 1
- Recorded Frobenius loss trace decreases monotonically across iterations
- Objective value L(P) = ||P A_CI P^T − A_CD||_F is lower than all 2000 random-permutation seed scores
- Interlaced cluster pairs (high CI-graph weights) receive colors with larger differences in A_CD than pre-optimization assignments
- numpy version confirmed as 1.22.x or 1.23.x, since other versions can alter graph-guided mode results

## Limitations

- The numpy version (1.22.x or 1.23.x) influences the result of the graph-guided mode of Spaco
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or when coloring hundreds of clusters

## Evidence

- [intro] weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships.: "a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships."
- [intro] Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment: "the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment"
- [methods] By quantifying the complex topology between cell type clusters, We optimized color assignment of to achieve better visual recognizability.: "By quantifying the complex topology between cell type clusters, We optimized color assignment of to achieve better visual recognizability."
- [methods] numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [methods] pip install git+https://github.com/BrainStOrmics/Spaco.git: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
