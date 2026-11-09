---
name: ci-graph-umap-embedding-cielab
description: Use when coloring cellular-resolution spatial transcriptomics (SRT) data
  where a default palette (e.g., Scanpy's) makes spatially adjacent cell types perceptually
  ambiguous, and you need colors chosen to maximize CIELab contrast among neighboring
  clusters.
when_to_use_negative:
- When spatial relationships between cell types are irrelevant to the visualization
  (e.g., non-spatial single-cell RNA-seq plots), a spatially unaware palette suffices
- When the user must preserve literature-conventional or manually specified colors
  for specific cell types without spatial optimization of the whole palette
- When coloring hundreds of clusters or requiring CVD-supporting colorization, where
  obtainable visual contrast through color space is limited per the article's discussion
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: Builds the CI-graph from Degree of Interlacement, embeds it in CIELab via
    UMAP, and performs graph-guided palette selection and cluster-color assignment
  repo: https://github.com/BrainStOrmics/Spaco
- name: Spaco_scripts
  role: Reproduction scripts for the benchmarking/analysis results, including the
    axolotl CI-graph-guided colorization workflow
  repo: https://github.com/BrainStOrmics/Spaco_scripts
- name: Scanpy
  role: Provides the default palette used as the baseline in the CIEDE2000 color-difference
    comparison
- name: Squidpy
  role: Loading the spatial dataset (e.g., sq.datasets) with spatial coordinates and
    cell-type labels
- name: matplotlib
  role: Renders the color-difference distribution comparison plot
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:ca4ef10e2ee419a4737f10fb20664413a1a071eebad3fe9c33620aa3e00be646
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Scanpy
  - Spaco
  - Spaco_scripts
  - Squidpy
  - matplotlib
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 21
---

# ci-graph-umap-embedding-cielab

## Summary

Spaco's graph-guided adaptive colorization: a cluster interlacement (CI-) graph built from cell-type spatial relationships (Degree of Interlacement) is embedded into CIELab space via UMAP, and colors are selected from the palette to amplify chromatic distinctions between spatially adjacent cell types.

## When to use

Use when coloring cellular-resolution spatial transcriptomics (SRT) data where a default palette (e.g., Scanpy's) makes spatially adjacent cell types perceptually ambiguous, and you need colors chosen to maximize CIELab contrast among neighboring clusters.

## When NOT to use

- When spatial relationships between cell types are irrelevant to the visualization (e.g., non-spatial single-cell RNA-seq plots), a spatially unaware palette suffices
- When the user must preserve literature-conventional or manually specified colors for specific cell types without spatial optimization of the whole palette
- When coloring hundreds of clusters or requiring CVD-supporting colorization, where obtainable visual contrast through color space is limited per the article's discussion

## Inputs

- Spatially resolved single-cell data (e.g., Stereo-seq .h5ad-style AnnData) with per-cell spatial coordinates and cell-type/cluster labels
- A candidate color palette (pre-defined or adaptively generated) to be assigned to cell types

## Outputs

- Cell-type-to-color mapping (color assignment) optimized for spatial contrast
- Pairwise CIEDE2000 color-difference table among palette colors (e.g., axolotl_colordiff_results.csv)
- Distribution comparison plot of color differences (e.g., axolotl_colordiff_comparison.png rendered with matplotlib)
- One-sided Wilcoxon signed-rank p-value comparing Spaco vs default palette color-difference distributions

## How to apply

Load the SRT data (e.g., the Stereo-seq axolotl brain dataset with 16 cell types) with spatial coordinates and cell-type labels via Squidpy/Scanpy, then use Spaco's three modules: (1) cell cluster interlacement modeling, which computes the Degree of Interlacement between every cell-type pair to build the weighted CI-graph; (2) color palette selection, where in graph-guided mode the CI-graph is embedded into CIELab space via UMAP and palette colors are selected to amplify chromatic distinctions; (3) cluster-color assignment. Compare against a baseline palette (e.g., Scanpy default) by computing pairwise CIEDE2000 color differences, restricted to spatially adjacent cell-type pairs, and test the paired distributions with a one-sided Wilcoxon signed-rank test. For exact reproducibility, pin the numpy version to 1.22.x or 1.23.x, as the numpy version can influence the graph-guided mode's result.

## Related tools

- **Spaco** (Builds the CI-graph from Degree of Interlacement, embeds it in CIELab via UMAP, and performs graph-guided palette selection and cluster-color assignment) — https://github.com/BrainStOrmics/Spaco
- **Spaco_scripts** (Reproduction scripts for the benchmarking/analysis results, including the axolotl CI-graph-guided colorization workflow) — https://github.com/BrainStOrmics/Spaco_scripts
- **Scanpy** (Provides the default palette used as the baseline in the CIEDE2000 color-difference comparison)
- **Squidpy** (Loading the spatial dataset (e.g., sq.datasets) with spatial coordinates and cell-type labels)
- **matplotlib** (Renders the color-difference distribution comparison plot)

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git
```

## Evaluation signals

- One-sided Wilcoxon signed-rank test on paired CIEDE2000 color-difference distributions (Spaco vs Scanpy default, adjacent cell-type pairs only) yields p ≈ 1.27e-11 (within numerical tolerance)
- Spaco's color-difference distribution is shifted upward relative to the default palette's distribution
- The CI-graph is correctly derived: Degree of Interlacement computed for all cell-type pairs from spatial coordinates before palette selection
- Embedding is performed in CIELab space (not RGB), so CIEDE2000 distances are computed in a perceptually uniform space
- Reproducibility check: numpy is pinned to 1.22.x or 1.23.x and the graph-guided mode output is stable across reruns

## Limitations

- numpy version (1.22.x or 1.23.x) could influence the result of the graph-guided mode of Spaco, so it must be fixed for exact reproducibility
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or when coloring hundreds of clusters
- For 3D data, fine-tuning of parameters is crucial to differentiate 'sparse' from 'dense' cell types for efficient 3D colorization

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [intro] a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships.: "a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships."
- [methods] numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [intro] the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment: "the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment"
- [methods] pip install git+https://github.com/BrainStOrmics/Spaco.git: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
