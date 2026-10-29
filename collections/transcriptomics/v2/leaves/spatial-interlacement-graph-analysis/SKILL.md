---
name: spatial-interlacement-graph-analysis
description: Use when single-cell spatially resolved transcriptomics data (SRT) has
  been clustered and cell-type colors must be assigned so that spatially interlaced/adjacent
  clusters get maximally contrasting colors from a limited palette.
when_to_use_negative:
- Clusters are spatially isolated with no interlacement — the CI-graph carries little
  signal and default framework colorization (Seurat/Giotto/Scanpy/Squidpy via matplotlib
  or ggplot2) suffices
- User has fixed colors for specific cell types that must not be optimized, or literature-conventional
  colors must be preserved — Spaco enforces color-difference correspondence to spatial
  distribution rather than user preference
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: 'Primary tool: implements cell cluster interlacement modeling (CI-graph),
    color palette selection, and cluster-color assignment via graph matching'
  repo: https://github.com/BrainStOrmics/Spaco
- name: Python
  role: Implementation language; numpy/scipy used to evaluate permutation objective
    and refine assignment
- name: SpacoR
  role: R port of Spaco for Seurat workflows (several functions not yet implemented
    at this stage)
  repo: https://github.com/BrainStOrmics/SpacoR
- name: Spaco_scripts
  role: Scripts to reproduce benchmarking and analytic results in the Spaco paper
  repo: https://github.com/BrainStOrmics/Spaco_scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:b311fdc3a18d6c5e772dc6fd86746bd27f705d6734b2cf641a9fefc66a81d6e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - SpacoR
  - Spaco_scripts
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spatial-interlacement-graph-analysis

## Summary

Constructs Spaco's cluster interlacement graph (CI-graph) from the Degree of Interlacement between cell-type pairs and solves its optimal one-to-one matching to the color-difference (CD-graph) target via coarse-to-fine stochastic permutation optimization, so heavily interlaced neighboring clusters receive more distinct colors.

## When to use

Use when single-cell spatially resolved transcriptomics data (SRT) has been clustered and cell-type colors must be assigned so that spatially interlaced/adjacent clusters get maximally contrasting colors from a limited palette.

## When NOT to use

- Clusters are spatially isolated with no interlacement — the CI-graph carries little signal and default framework colorization (Seurat/Giotto/Scanpy/Squidpy via matplotlib or ggplot2) suffices
- User has fixed colors for specific cell types that must not be optimized, or literature-conventional colors must be preserved — Spaco enforces color-difference correspondence to spatial distribution rather than user preference

## Inputs

- Clustered single-cell SRT data (e.g., AnnData .h5ad with spatial coordinates and cluster labels)
- Cluster interlacement graph (CI-graph) adjacency matrix A_CI from DOI computation (e.g., produced by the cell cluster interlacement modeling module)
- Initial color palette / color-difference target matrix A_CD

## Outputs

- Optimized cluster-to-color mapping (permutation matrix P and assignment)
- Frobenius loss trace L(P) = ||P A_CI P^T − A_CD||_F across optimization iterations
- color_mapping object consumable by spaco.colorize for rendering colored spatial plots

## How to apply

First compute the Degree of Interlacement (DOI) for every cell-type pair and assemble the weighted CI-graph adjacency matrix A_CI that globally models spatial relationships between clusters. Build the color-difference target matrix A_CD from the selected palette. Then find the one-to-one cluster-to-color correspondence by sampling 2000 random permutations as seeds, evaluating L(P) = ||P A_CI P^T − A_CD||_F with numpy/scipy, and retaining the best seed. Refine the winning permutation with pairwise color-swap fine-tuning capped at 5000 iterations, stopping after 100 stall iterations without improvement. Validate that P is a valid permutation matrix (one-hot rows and columns) with a monotonically decreasing Frobenius loss trace, and pin numpy to 1.22.x or 1.23.x because the numpy version influences the graph-guided mode's result.

## Related tools

- **Spaco** (Primary tool: implements cell cluster interlacement modeling (CI-graph), color palette selection, and cluster-color assignment via graph matching) — https://github.com/BrainStOrmics/Spaco
- **Python** (Implementation language; numpy/scipy used to evaluate permutation objective and refine assignment)
- **SpacoR** (R port of Spaco for Seurat workflows (several functions not yet implemented at this stage)) — https://github.com/BrainStOrmics/SpacoR
- **Spaco_scripts** (Scripts to reproduce benchmarking and analytic results in the Spaco paper) — https://github.com/BrainStOrmics/Spaco_scripts

## Examples

```
color_mapping = spaco.colorize(adata, palette=palette, colorblind_safe=True)
```

## Evaluation signals

- Returned P is a valid permutation matrix: every row and column is exactly one-hot (one-to-one cluster-color correspondence)
- Recorded Frobenius loss L(P) = ||P A_CI P^T − A_CD||_F decreases monotonically across the fine-tuning trace
- Fine-tuning terminates within the 5000-iteration cap or stops after 100 iterations without improvement (stall stop)
- Heavily interlaced cell-type pairs (high DOI) receive larger color-difference values than isolated pairs in the final mapping
- Reproducibility check: results are generated with numpy pinned to 1.22.x or 1.23.x, since the numpy version influences the graph-guided mode result

## Limitations

- numpy version (1.22.x or 1.23.x) influences the result of the graph-guided mode of Spaco, so versions must be pinned for reproducibility
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- The visual contrast obtainable through color space is limited when performing CVD-supporting colorization or when coloring hundreds of clusters

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types"
- [intro] CI-graph construction and the three Spaco modules: "a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships."
- [intro] Three-module pipeline framing: "the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment"
- [methods] numpy version dependency of the graph-guided mode: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [methods] colorize API entry point: "color_mapping = spaco.colorize("
- [readme] Installation of the Python implementation: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
