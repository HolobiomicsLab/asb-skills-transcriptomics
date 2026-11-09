---
name: spatial-data-visualization
description: Use when visualizing single-cell spatial omics data (e.g., Stereo-seq
  axolotl brain, STARmap, seqFISH, IMC datasets) and default color assignments from
  Seurat, Scanpy, Giotto, Squidpy, matplotlib, or ggplot2 produce perceptual ambiguity
  between neighboring, biologically distinct cell types.
when_to_use_negative:
- Data is non-spatial (no spatial coordinates) — Spaco's DOI/CI-graph modeling relies
  on spatial interlacement between cell types
- Coloring hundreds of clusters or requiring strict CVD-supporting colorization —
  visual contrast obtainable through color space is limited in these cases per the
  paper's discussion
- User requires color differences to follow literature conventions rather than spatial
  distribution — Spaco enforces correspondence between color differences and spatial
  distribution
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: 'Core colorization tool: DOI-based interlacement modeling, palette selection,
    and cluster-color assignment (spaco.colorize)'
  repo: https://github.com/BrainStOrmics/Spaco
- name: Squidpy
  role: Loads spatial SRT datasets (e.g., sq.datasets.seqfish(), Stereo-seq axolotl
    brain) as AnnData for colorization
- name: Scanpy
  role: Analysis framework whose default colorization Spaco replaces with spatially
    aware assignments
- name: SpacoR
  role: R implementation for use with Seurat objects (partially feature-complete)
  repo: https://github.com/BrainStOrmics/SpacoR
- name: Spaco_scripts
  role: Reproduction scripts and vignettes (e.g., Demo.ipynb) for benchmarking and
    analytic results
  repo: https://github.com/BrainStOrmics/Spaco_scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:b4be0c9343311469039ded0c95c50a6ba65f2285aac9f31dea3eab663a64d59b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Scanpy
  - Spaco
  - SpacoR
  - Spaco_scripts
  - Squidpy
  - numpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# spatial-data-visualization

## Summary

Spatially aware colorization of cellular-resolution spatially resolved transcriptomics (SRT) data using Spaco, which constructs a Degree of Interlacement (DOI)-weighted cluster interlacement graph to assign perceptually contrasting colors to adjacent cell types, with adaptive palette selection (including image-guided palette extraction) and CVD-supporting palettes.

## When to use

Use when visualizing single-cell spatial omics data (e.g., Stereo-seq axolotl brain, STARmap, seqFISH, IMC datasets) and default color assignments from Seurat, Scanpy, Giotto, Squidpy, matplotlib, or ggplot2 produce perceptual ambiguity between neighboring, biologically distinct cell types.

## When NOT to use

- Data is non-spatial (no spatial coordinates) — Spaco's DOI/CI-graph modeling relies on spatial interlacement between cell types
- Coloring hundreds of clusters or requiring strict CVD-supporting colorization — visual contrast obtainable through color space is limited in these cases per the paper's discussion
- User requires color differences to follow literature conventions rather than spatial distribution — Spaco enforces correspondence between color differences and spatial distribution

## Inputs

- AnnData object with spatial coordinates and cell-type/cluster labels (e.g., loaded via Squidpy from Stereo-seq, STARmap, seqFISH, or IMC data)
- Cell-type label assignment (cluster column) for each cell
- Optional theme image (e.g., journal cover image) for image-guided palette extraction
- Optional user-fixed color mappings for selected cell types

## Outputs

- Color mapping dictionary (cluster-to-color assignments) applied to spatial plot of cells
- Extracted color palette (when image-guided palette extraction is used)
- Visualized spatial map with contrasting, spatially aware cell-type colors

## How to apply

Run Spaco's three-module pipeline: (1) cell cluster interlacement modeling — compute the Degree of Interlacement metric between cell-type pairs from cell coordinates and labels and build the weighted cluster interlacement (CI-) graph; (2) color palette selection — either use an adaptive palette approach to amplify chromatic distinctions or extract a palette from a theme image by binning colors in CIELab space, updating frequencies with the CIEDE2000 color-difference metric, and refining with farthest point sampling (FPS) to preserve the theme color while maximizing color differentiation; (3) cluster-color assignment via spaco.colorize, which maps contrasting palette entries onto highly interlaced cell-type pairs. Inputs are AnnData objects with spatial coordinates and cell-type labels loaded via Squidpy (e.g., sq.datasets.seqfish()). R users can use SpacoR with Seurat; users may also manually fix colors for specified cell types and let Spaco optimize the rest.

## Related tools

- **Spaco** (Core colorization tool: DOI-based interlacement modeling, palette selection, and cluster-color assignment (spaco.colorize)) — https://github.com/BrainStOrmics/Spaco
- **Squidpy** (Loads spatial SRT datasets (e.g., sq.datasets.seqfish(), Stereo-seq axolotl brain) as AnnData for colorization)
- **Scanpy** (Analysis framework whose default colorization Spaco replaces with spatially aware assignments)
- **SpacoR** (R implementation for use with Seurat objects (partially feature-complete)) — https://github.com/BrainStOrmics/SpacoR
- **Spaco_scripts** (Reproduction scripts and vignettes (e.g., Demo.ipynb) for benchmarking and analytic results) — https://github.com/BrainStOrmics/Spaco_scripts

## Examples

```
import squidpy as sq, spaco; adata_cell = sq.datasets.seqfish(); color_mapping = spaco.colorize(adata=adata_cell, cluster_key='cluster')
```

## Evaluation signals

- Pairwise CIEDE2000 color differences among Spaco-assigned colors should be comparable to or better than reference palettes like matplotlib tab20 (reproduced Wilcoxon signed-rank test p = 1.20e-11 on the axolotl dataset)
- Adjacent cell-type pairs with high Degree of Interlacement receive perceptually contrasting colors in the rendered spatial plot
- Image-guided palette extraction preserves the theme color of the source image while maximizing color differentiation (FPS refinement output)
- Spacified spatial plot is rendered without perceptual ambiguity between neighboring biologically distinct cell types
- Reproduction via Spaco_scripts matches benchmarking figures in the paper

## Limitations

- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or when coloring hundreds of clusters
- 3D colorization requires fine-tuning of parameters to differentiate 'sparse' from 'dense' cell types
- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco

## Evidence

- [abstract] DOI metric drives coloring: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [intro] three modules: "the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment"
- [methods] palette extraction rationale: "We provide a method for extracting color plates from images. While maintaining the theme color, the color differentiation is maximized."
- [discussion] manual color fixing: "Spaco offers an option allowing users to manually fix or adjust colors for user-specified cell types and then optimize the color differences for the remaining cell types."
- [methods] workflow usage of colorize: "color_mapping = spaco.colorize("
- [methods] spatial data loading in demo: "adata_cell = sq.datasets.seqfish()"
