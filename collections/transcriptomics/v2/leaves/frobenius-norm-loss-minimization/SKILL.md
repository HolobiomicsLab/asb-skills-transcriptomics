---
name: frobenius-norm-loss-minimization
description: Use when you have a weighted cluster interlacement adjacency matrix (CI-graph)
  from Spaco's DOI modeling plus a selected color palette, and need an optimal cluster-to-color
  mapping.
when_to_use_negative:
- When the user wants colors to follow literature conventions or personal preferences
  rather than spatial contrast — Spaco enforces a correspondence between color differences
  and the spatial distribution of cells
- When hundreds of clusters must be colored or strict color-vision-deficiency support
  is required, since the achievable visual contrast through color space is limited
- When only a naive default coloring suffices — this stochastic graph-matching is
  meant for spatially constrained single-cell-resolution SRT colorization, not generic
  chart coloring
tools:
- name: Spaco
  role: Provides the graph-guided colorization mode whose cluster-color assignment
    this optimization implements; invoked via spaco.colorize
  repo: github.com/BrainStOrmics/Spaco
- name: Python
  role: Implementation language for the permutation sampling and optimization loop
  repo: github.com/BrainStOrmics/Spaco
- name: numpy
  role: Matrix algebra (permutation matrix multiplication, Frobenius norm evaluation);
    version 1.22.x/1.23.x required
  repo: github.com/BrainStOrmics/Spaco
- name: scipy
  role: Numerical routines supporting the stochastic optimization and fine-tuning
  repo: github.com/BrainStOrmics/Spaco
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:b906a36e5b5745db0ffa328555e4c2f1496335f6c1bbf9c02171cfe76cd0e590
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
yes_rate: 0.75
n_claims: 8
---

# frobenius-norm-loss-minimization

## Summary

Solves Spaco's cluster-color assignment by stochastically searching for a one-to-one permutation matrix P that minimizes the Frobenius norm loss L(P) = ||P A_CI P^T − A_CD||_F, aligning the cluster interlacement (CI-graph) with the color-difference (CD-graph) so spatially interlaced clusters receive distinct colors.

## When to use

Use when you have a weighted cluster interlacement adjacency matrix (CI-graph) from Spaco's DOI modeling plus a selected color palette, and need an optimal cluster-to-color mapping.

## When NOT to use

- When the user wants colors to follow literature conventions or personal preferences rather than spatial contrast — Spaco enforces a correspondence between color differences and the spatial distribution of cells
- When hundreds of clusters must be colored or strict color-vision-deficiency support is required, since the achievable visual contrast through color space is limited
- When only a naive default coloring suffices — this stochastic graph-matching is meant for spatially constrained single-cell-resolution SRT colorization, not generic chart coloring

## Inputs

- CI-graph adjacency matrix A_CI (weighted cluster interlacement matrix from Spaco's Degree of Interlacement modeling)
- Color palette (from Spaco's adaptive palette selection step) used to construct the color-difference target matrix A_CD

## Outputs

- color_mapping: optimized cluster-to-color mapping (valid permutation matrix P)
- Loss trace: recorded Frobenius loss values across optimization iterations

## How to apply

Load the CI-graph adjacency matrix A_CI (produced by Spaco's cell cluster interlacement modeling) and build the color-difference target matrix A_CD from the chosen palette. Sample ~2000 random permutations of the cluster-to-color assignment as optimization seeds, then evaluate L(P) = ||P A_CI P^T − A_CD||_F for each seed with numpy/scipy, retaining the best-scoring permutation. Refine the winner with pairwise color-swap fine-tuning capped at 5000 iterations, with a stall stop after 100 iterations without improvement. Verify the returned P is a valid permutation matrix (one-hot rows and columns) and that the recorded Frobenius loss decreases monotonically. Pin numpy to 1.22.x or 1.23.x, since the numpy version influences the result of the graph-guided mode of Spaco.

## Related tools

- **Spaco** (Provides the graph-guided colorization mode whose cluster-color assignment this optimization implements; invoked via spaco.colorize) — github.com/BrainStOrmics/Spaco
- **Python** (Implementation language for the permutation sampling and optimization loop) — github.com/BrainStOrmics/Spaco
- **numpy** (Matrix algebra (permutation matrix multiplication, Frobenius norm evaluation); version 1.22.x/1.23.x required) — github.com/BrainStOrmics/Spaco
- **scipy** (Numerical routines supporting the stochastic optimization and fine-tuning) — github.com/BrainStOrmics/Spaco

## Examples

```
import numpy as np; loss = np.linalg.norm(P @ A_CI @ P.T - A_CD, 'fro')
```

## Evaluation signals

- Returned P is a valid permutation matrix: every row and column is one-hot
- Recorded Frobenius loss L(P) = ||P A_CI P^T − A_CD||_F decreases monotonically across the optimization trace
- Heavily interlaced cluster pairs (high DOI edge weights in the CI-graph) receive colors with larger differences in A_CD
- Pairwise color-swap fine-tuning terminates with the 5000-iteration cap or after 100 stall iterations without improvement
- numpy version is pinned to 1.22.x or 1.23.x so results are reproducible

## Limitations

- numpy version (1.22.x or 1.23.x) influences the result of the graph-guided mode of Spaco, so results may not be reproducible under other versions
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast obtainable through color space is limited with CVD-supporting colorization or when coloring hundreds of clusters

## Evidence

- [intro] module 3 quote: "the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment"
- [methods] color assignment optimization: "By quantifying the complex topology between cell type clusters, We optimized color assignment of to achieve better visual recognizability."
- [methods] numpy version caveat: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [discussion] limitation on correspondence: "One major limitation of Spaco is its enforcement of a correspondence between color differences and the spatial distribution of cells."
- [methods] colorize call: "color_mapping = spaco.colorize("
