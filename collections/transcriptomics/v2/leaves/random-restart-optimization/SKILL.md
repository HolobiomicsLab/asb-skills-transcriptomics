---
name: random-restart-optimization
description: Use when a discrete assignment problem (e.g., mapping cell clusters to
  palette colors) must be solved by minimizing a Frobenius-norm graph-matching objective,
  and local search alone risks poor local minima.
when_to_use_negative:
- When the user has manually fixed or specified colors for particular cell types and
  only wants the remaining types optimized — use Spaco's manual color-fixing option
  instead of unconstrained restarts
- When the goal is palette selection or palette extraction from an image rather than
  assignment of an already-selected palette to clusters
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: Tool whose cluster-color assignment module implements the random-restart permutation
    search and color-swap refinement
  repo: https://github.com/BrainStOrmics/Spaco
- name: numpy
  role: Permutation sampling, matrix multiplication P A_CI P^T, and Frobenius norm
    evaluation; version (1.22.x/1.23.x) affects results
- name: scipy
  role: Supports evaluation of the graph-matching objective over the sampled permutations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:2e830998db705b080a789ce666427f4bc343fd22d4500bd1c81a70956e69a147
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - numpy
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# random-restart-optimization

## Summary

A coarse-to-fine stochastic optimization strategy for the cluster-color assignment step of Spaco: random permutation restarts over the CI-graph/CD-graph matching objective, followed by pairwise color-swap refinement, to find a one-to-one cluster-to-color mapping that maximizes color contrast between interlaced cell types.

## When to use

Use when a discrete assignment problem (e.g., mapping cell clusters to palette colors) must be solved by minimizing a Frobenius-norm graph-matching objective, and local search alone risks poor local minima.

## When NOT to use

- When the user has manually fixed or specified colors for particular cell types and only wants the remaining types optimized — use Spaco's manual color-fixing option instead of unconstrained restarts
- When the goal is palette selection or palette extraction from an image rather than assignment of an already-selected palette to clusters

## Inputs

- CI-graph adjacency matrix A_CI (cluster interlacement graph from cell-cluster interlacement modeling)
- Color-difference target matrix A_CD built from the selected color palette
- Number of clusters / palette size (dimension of the permutation matrix P)

## Outputs

- Optimized cluster-to-color mapping (permutation matrix P) saved as color_mapping
- Frobenius loss trace recording the monotonic decrease of ||P A_CI P^T − A_CD||_F during optimization

## How to apply

Load the CI-graph adjacency matrix A_CI (built from Degree of Interlacement measurements between cell-type clusters) and the color-difference target matrix A_CD derived from the selected palette. Sample 2000 random permutations of the cluster-to-color assignment as optimization seeds, evaluating each against the objective L(P) = ||P A_CI P^T − A_CD||_F using numpy and scipy, and retain the best-scoring permutation. Refine the winning assignment with pairwise color-swap fine-tuning capped at 5000 iterations, stopping early if 100 iterations pass without improvement. Validate that the returned P is a valid permutation matrix (one-hot rows and columns) and that the recorded Frobenius loss trace decreases monotonically; pin numpy to 1.22.x or 1.23.x because the numpy version influences the result of Spaco's graph-guided mode.

## Related tools

- **Spaco** (Tool whose cluster-color assignment module implements the random-restart permutation search and color-swap refinement) — https://github.com/BrainStOrmics/Spaco
- **numpy** (Permutation sampling, matrix multiplication P A_CI P^T, and Frobenius norm evaluation; version (1.22.x/1.23.x) affects results)
- **scipy** (Supports evaluation of the graph-matching objective over the sampled permutations)

## Examples

```
import numpy as np; loss = np.linalg.norm(P @ A_CI @ P.T - A_CD, 'fro')  # evaluate over 2000 random permutation seeds, then refine with pairwise swaps
```

## Evaluation signals

- Returned P is a valid permutation matrix: every row and column is one-hot
- Recorded Frobenius loss trace L(P) = ||P A_CI P^T − A_CD||_F decreases monotonically across accepted assignments
- The coarse stage sampled exactly 2000 random permutation seeds before refinement
- Fine-tuning terminates within the 5000-iteration cap or stalls after 100 iterations without improvement
- Heavily interlaced cluster pairs (high Degree of Interlacement) receive colors with larger palette-defined color differences

## Limitations

- The numpy version (1.22.x or 1.23.x) influences the result of the graph-guided mode of Spaco, so results are not fully version-stable
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast through color space is limited when performing CVD-supporting colorization or coloring hundreds of clusters, bounding what the optimization can achieve

## Evidence

- [methods] By quantifying the complex topology between cell type clusters, We optimized color assignment of to achieve better visual recognizability.: "By quantifying the complex topology between cell type clusters, We optimized color assignment of to achieve better visual recognizability."
- [methods] numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [intro] a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships.: "a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships."
- [readme] provides a spatially constrained approach that generates discriminate color assignments for visualizing single-cell spatial data in various scenarios: "provides a spatially constrained approach that generates discriminate color assignments for visualizing single-cell spatial data in various scenarios"
