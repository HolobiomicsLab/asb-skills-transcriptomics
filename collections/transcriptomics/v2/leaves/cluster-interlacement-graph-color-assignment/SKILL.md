---
name: cluster-interlacement-graph-color-assignment
description: Use when visualizing single-cell spatial (SRT) data where default palettes
  (e.g., Scanpy, Seurat, Squidpy, Giotto) cause perceptual ambiguity between neighboring,
  biologically distinct cell types that need contrasting colors.
when_to_use_negative:
- Data has no spatial coordinates — the CI-graph/DOI metric requires spatial relationships
  between cells; for purely non-spatial clusterings use standard palette tools
- Coloring hundreds of clusters or requiring CVD-supporting colorization, where the
  article notes visual contrast obtainable through color space is limited
- You need colors to follow literature conventions or user preferences rather than
  the spatial-distribution correspondence Spaco enforces
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: 'Primary tool: builds the CI-graph from DOI and performs spatially aware cluster-color
    assignment via spaco.colorize'
  repo: github.com/BrainStOrmics/Spaco
- name: Spaco_scripts
  role: Benchmark and vignette scripts to reproduce the paper's colorization analyses
  repo: github.com/BrainStOrmics/Spaco_scripts
- name: SpacoR
  role: R implementation for use with Seurat objects (partial feature coverage)
  repo: github.com/BrainStOrmics/SpacoR
- name: Squidpy
  role: Dataset loader for spatial datasets (e.g., sq.datasets.seqfish(), STARmap
    mouse brain)
- name: Scanpy
  role: Handles the AnnData object and provides the default palette used as a baseline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:2bfd0218f845999e1ebffd4224be40aa64198bce076fbea37c73d3362700863e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Scanpy
  - Spaco
  - SpacoR
  - Spaco_scripts
  - Squidpy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# cluster-interlacement-graph-color-assignment

## Summary

Assigns colors to cell-type clusters in spatially resolved transcriptomics data by constructing a weighted cluster interlacement graph (CI-graph) from the Degree of Interlacement (DOI) metric, so that spatially adjacent clusters receive maximally contrasting colors. Implemented in the Spaco tool (Python) and SpacoR (R).

## When to use

Use when visualizing single-cell spatial (SRT) data where default palettes (e.g., Scanpy, Seurat, Squidpy, Giotto) cause perceptual ambiguity between neighboring, biologically distinct cell types that need contrasting colors.

## When NOT to use

- Data has no spatial coordinates — the CI-graph/DOI metric requires spatial relationships between cells; for purely non-spatial clusterings use standard palette tools
- Coloring hundreds of clusters or requiring CVD-supporting colorization, where the article notes visual contrast obtainable through color space is limited
- You need colors to follow literature conventions or user preferences rather than the spatial-distribution correspondence Spaco enforces

## Inputs

- Cell coordinates and region-localized cluster labels for a spatial dataset (e.g., STARmap mouse brain via Squidpy's dataset loader, handled with Scanpy)
- An AnnData/h5ad object with spatial coordinates and cluster annotations
- Optional: shared palette to constrain colorization, or an image from which to extract a color plate

## Outputs

- Color mapping (cluster-to-color assignments) optimized for spatial contrast, produced by spaco.colorize
- Benchmark statistics: CIEDE2000 color differences for adjacent cluster pairs and mean color difference per FOV, with Wilcoxon p-values (e.g., wilcoxon_pvalues.csv)

## How to apply

Run Spaco's three-module process: (1) cell cluster interlacement modeling — build a weighted CI-graph from cell coordinates and cluster labels using the Degree of Interlacement metric to quantify spatial relationships between cell-type pairs; (2) color palette selection — optionally use the adaptive palette selection (or palette extraction from images) to amplify chromatic distinctions; (3) cluster-color assignment — optimize the assignment so adjacent clusters on the CI-graph get high color difference (CIEDE2000). Call spaco.colorize on the cell coordinates and labels; for graph-guided mode use parameters such as radius=0.05 and n_neighbors=30, and pin numpy to 1.22.x or 1.23.x for reproducibility. Colors for specific cell types can be manually fixed and the remainder optimized. Benchmark correctness by computing CIEDE2000 color differences for anatomically adjacent cluster pairs and per-FOV means, then compare against Scanpy default, random shuffling, and Palo with one-sided Wilcoxon rank-sum tests.

## Related tools

- **Spaco** (Primary tool: builds the CI-graph from DOI and performs spatially aware cluster-color assignment via spaco.colorize) — github.com/BrainStOrmics/Spaco
- **Spaco_scripts** (Benchmark and vignette scripts to reproduce the paper's colorization analyses) — github.com/BrainStOrmics/Spaco_scripts
- **SpacoR** (R implementation for use with Seurat objects (partial feature coverage)) — github.com/BrainStOrmics/SpacoR
- **Squidpy** (Dataset loader for spatial datasets (e.g., sq.datasets.seqfish(), STARmap mouse brain))
- **Scanpy** (Handles the AnnData object and provides the default palette used as a baseline)

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git; color_mapping = spaco.colorize(adata, radius=0.05, n_neighbors=30)
```

## Evaluation signals

- One-sided Wilcoxon rank-sum tests show Spaco's adjacent-pair CIEDE2000 differences significantly exceed baselines (paper reference values: p = 1.45e-4 vs Scanpy, 2.32e-3 vs random, 4.51e-3 vs Palo)
- FOV mean color differences also significant (reference p-values: 8.42e-21, 9.16e-19, 8.87e-21)
- Graph-guided mode results reproduce when numpy is pinned to 1.22.x or 1.23.x
- Anatomically adjacent cluster pairs (e.g., 17 pairs defined via Allen Brain Atlas annotation of 10 STARmap clusters) receive visually distinct colors in rendered spatial plots
- Output color mapping covers all input cluster labels with no unassigned clusters

## Limitations

- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast through color space is limited when performing CVD-supporting colorization or coloring hundreds of clusters
- numpy version (1.22.x or 1.23.x) influences the result of graph-guided mode
- 3D colorization requires fine-tuning of parameters to differentiate 'sparse' from 'dense' cell types

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [intro] Three modules: "the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment"
- [methods] colorize invocation: "color_mapping = spaco.colorize("
- [methods] numpy pinning: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [discussion] Manual color fixing: "Spaco offers an option allowing users to manually fix or adjust colors for user-specified cell types and then optimize the color differences for the remaining cell types."
- [readme] pip install: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
