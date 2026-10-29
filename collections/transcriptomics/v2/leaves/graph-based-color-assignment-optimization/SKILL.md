---
name: graph-based-color-assignment-optimization
description: Use when a spatial transcriptomics colorization step must assign colors
  from a fixed palette to cell-type clusters such that adjacent or interlaced clusters
  (quantified by Degree of Interlacement) get perceptually contrasting colors.
when_to_use_negative:
- When the user wants colors to follow literature conventions or personal preferences
  rather than spatial interlacement — Spaco enforces a correspondence between color
  differences and the spatial distribution of cells, which may not match such conventions.
- When coloring hundreds of clusters or requiring CVD-supporting colorization where
  visual contrast available in color space is inherently limited.
- When only cell-type labels exist without any spatial coordinates — the CI-graph
  cannot be built and the optimization objective is undefined.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Spaco
  role: Provides the graph-guided colorization mode whose cluster-color assignment
    step is implemented here; invoked via spaco.colorize
  repo: https://github.com/BrainStOrmics/Spaco
- name: Python
  role: Implementation language for the stochastic permutation-sampling and color-swap
    fine-tuning optimization
- name: numpy
  role: Matrix algebra for the Frobenius objective and permutation-matrix operations;
    must be pinned to 1.22.x or 1.23.x
- name: scipy
  role: Supports the optimization/evaluation of permutation seeds
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:1bff4dc269e3f22c3904f24ed6509c33e6dbebae98a7fa391aad2b98dc95de06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# graph-based-color-assignment-optimization

## Summary

Spaco's cluster-color assignment module solves a one-to-one graph-matching problem between the cluster interlacement graph (CI-graph) and the color-difference graph (CD-graph) using coarse-to-fine stochastic optimization, so heavily interlaced cell-type clusters receive maximally distinct colors.

## When to use

Use when a spatial transcriptomics colorization step must assign colors from a fixed palette to cell-type clusters such that adjacent or interlaced clusters (quantified by Degree of Interlacement) get perceptually contrasting colors.

## When NOT to use

- When the user wants colors to follow literature conventions or personal preferences rather than spatial interlacement — Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match such conventions.
- When coloring hundreds of clusters or requiring CVD-supporting colorization where visual contrast available in color space is inherently limited.
- When only cell-type labels exist without any spatial coordinates — the CI-graph cannot be built and the optimization objective is undefined.

## Inputs

- CI-graph adjacency matrix from Spaco's cell cluster interlacement modeling module (Degree of Interlacement weighted)
- color palette / color-difference target matrix A_CD (from the palette-selection module)
- cell-by-cluster label annotations for the SRT dataset (e.g. .h5ad, as in sq.datasets.seqfish())

## Outputs

- color_mapping: optimized one-to-one cluster-to-color assignment (permutation matrix P)
- loss trace of the Frobenius objective L(P) = ||P A_CI P^T − A_CD||_F across the optimization

## How to apply

Load the CI-graph adjacency matrix from the cell cluster interlacement modeling module and build the color-difference target matrix A_CD from the selected palette. Sample 2000 random permutations of the cluster-to-color assignment as optimization seeds and evaluate the Frobenius objective L(P) = ||P A_CI P^T − A_CD||_F with numpy/scipy, keeping the best permutation. Refine the best seed by pairwise color-swap fine-tuning capped at 5000 iterations with a stall stop after 100 non-improving iterations. Validate that the returned P is a valid permutation matrix (one-hot rows and columns) with a monotonically decreasing loss trace, and pin numpy to 1.22.x or 1.23.x because the numpy version influences the result of Spaco's graph-guided mode. Save the optimized cluster-to-color mapping together with its loss trace as color_mapping.

## Related tools

- **Spaco** (Provides the graph-guided colorization mode whose cluster-color assignment step is implemented here; invoked via spaco.colorize) — https://github.com/BrainStOrmics/Spaco
- **Python** (Implementation language for the stochastic permutation-sampling and color-swap fine-tuning optimization)
- **numpy** (Matrix algebra for the Frobenius objective and permutation-matrix operations; must be pinned to 1.22.x or 1.23.x)
- **scipy** (Supports the optimization/evaluation of permutation seeds)

## Examples

```
color_mapping = spaco.colorize(adata, spacial_info_key='spatial', mode='graph-guided')
```

## Evaluation signals

- Returned P is a valid permutation matrix: every row and column is one-hot.
- Recorded Frobenius loss trace decreases monotonically over the color-swap fine-tuning iterations.
- Heavily interlaced cluster pairs (high Degree of Interlacement) end up with larger color differences than isolated/adjacent pairs.
- Optimization respects the iteration budget (≤5000 color swaps) with stall stop after 100 non-improving iterations.
- Reproducibility check: results should be consistent when numpy is held at 1.22.x or 1.23.x.

## Limitations

- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco.
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions.
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or when coloring hundreds of clusters.
- 3D colorization requires fine-tuning of parameters to differentiate 'sparse' from 'dense' cell types.

## Evidence

- [intro] the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment: "the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment"
- [intro] a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships: "a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships."
- [methods] numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [methods] color_mapping = spaco.colorize(: "color_mapping = spaco.colorize("
- [discussion] One major limitation of Spaco is its enforcement of a correspondence between color differences and the spatial distribution of cells.: "One major limitation of Spaco is its enforcement of a correspondence between color differences and the spatial distribution of cells."
