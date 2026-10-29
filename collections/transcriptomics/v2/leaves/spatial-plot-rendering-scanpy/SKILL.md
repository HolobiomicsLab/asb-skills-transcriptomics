---
name: spatial-plot-rendering-scanpy
description: Use when an AnnData object with cell spatial coordinates and cell-type
  labels needs to be visualized as a spatial map with a custom cluster-to-color mapping,
  e.g., after Spaco colorization.
when_to_use_negative:
- Data lacks per-cell spatial coordinates (e.g., a dissociated scRNA-seq object),
  since sc.pl.spatial requires spatial coordinates to place spots
- The goal is default library colorization rather than a spatially aware, contrast-optimized
  mapping — Scanpy/Squidpy defaults draw colors from matplotlib palettes and can ambiguously
  color adjacent distinct cell types
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Scanpy
  role: Provides sc.pl.spatial used to render the colorized spatial maps at spot_size=0.035
- name: Squidpy
  role: Loads the Stereo-seq axolotl sections (cell spatial coordinates and cell-type
    labels) as AnnData for plotting
- name: Spaco
  role: Supplies the contrast-optimized color_mapping dictionary that is reordered
    and applied to the spatial plot
  repo: https://github.com/BrainStOrmics/Spaco
- name: matplotlib
  role: Underlying plotting backend used by sc.pl.spatial to produce the figures
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:6082bb9f30ffa6138708fe605098d60758fc73027709b62bd554dd04b62a186d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Scanpy
  - Spaco
  - Squidpy
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# spatial-plot-rendering-scanpy

## Summary

Renders colorized single-cell spatial maps of spatially resolved transcriptomics data with Scanpy's sc.pl.spatial, applying a color_mapping dictionary (e.g., produced by Spaco) to cell-type labels at a specified spot_size so clusters are visually distinguishable in space.

## When to use

Use when an AnnData object with cell spatial coordinates and cell-type labels needs to be visualized as a spatial map with a custom cluster-to-color mapping, e.g., after Spaco colorization.

## When NOT to use

- Data lacks per-cell spatial coordinates (e.g., a dissociated scRNA-seq object), since sc.pl.spatial requires spatial coordinates to place spots
- The goal is default library colorization rather than a spatially aware, contrast-optimized mapping — Scanpy/Squidpy defaults draw colors from matplotlib palettes and can ambiguously color adjacent distinct cell types

## Inputs

- AnnData object with spatial coordinates (spatial slot) and categorical cell-type labels, e.g., Stereo-seq axolotl DPI-10 or DPI-15 sections
- color_mapping dictionary from Spaco colorization (cluster label -> hex color)

## Outputs

- Spatial scatter plots of cells colored by cell type (matplotlib figures via sc.pl.spatial) for each section
- Reordered color_mapping dictionary aligned to the categorical order of cell-type labels

## How to apply

Load the spatial data (e.g., the Stereo-seq axolotl DPI-10 and DPI-15 sections with cell coordinates and cell-type labels) via Squidpy, then obtain a color_mapping dictionary from Spaco's colorize output. Reorder the dictionary to match the categorical order of the cell-type labels so each category receives its intended color. Render with sc.pl.spatial using spot_size=0.035 (the value used in the Spaco_scripts vignette for Stereo-seq axolotl sections), passing the reordered mapping so each cell type is plotted with its contrast-optimized color. Verify the plot visually reproduces the reference vignette result, e.g., sfrpEGCs at DPI-15 and TLNBLs at DPI-10 appearing in perceptually distinct colors.

## Related tools

- **Scanpy** (Provides sc.pl.spatial used to render the colorized spatial maps at spot_size=0.035)
- **Squidpy** (Loads the Stereo-seq axolotl sections (cell spatial coordinates and cell-type labels) as AnnData for plotting)
- **Spaco** (Supplies the contrast-optimized color_mapping dictionary that is reordered and applied to the spatial plot) — https://github.com/BrainStOrmics/Spaco
- **matplotlib** (Underlying plotting backend used by sc.pl.spatial to produce the figures)

## Examples

```
sc.pl.spatial(adata, color='cluster', spot_size=0.035, palette=reordered_color_mapping, save='dpi15_colorized.png')
```

## Evaluation signals

- Each cell type in the plot matches the intended hex color from the color_mapping dictionary (check legend order matches reordered mapping)
- sfrpEGCs at DPI-15 and TLNBLs at DPI-10 appear in perceptually distinct colors (maximized CIEDE2000 color difference under the shared palette)
- Output spatial map matches the reference Spaco_scripts Vignette/Demo.ipynb result for the same section and parameters
- spot_size=0.035 renders cells at the same scale as the vignette; cells neither overlap into a solid mask nor appear as sparse isolated points

## Limitations

- Scanpy's default visualization (without Spaco colors) obscured the increase in sfrpEGCs during wound healing because TLNBLs at DPI-10 were incorrectly matched with sfrpEGCs at DPI-15
- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco, indirectly changing the colors fed to the plot

## Evidence

- [other] Reorder the resulting color_mapping dictionary to the categorical order of the cell-type labels and render the colorized spatial maps for both sections with sc.pl.spatial (spot_size=0.035).: [evidence span withheld — anchor status: not_found]
- [other] Scanpy's default visualization obscured it because TLNBLs at DPI-10 were incorrectly matched with sfrpEGCs at DPI-15: [evidence span withheld — anchor status: not_found]
- [intro] popular data analysis frameworks like Seurat,14 Giotto,15 Scanpy,16 and Squidpy17 derive their colorization approaches from widely used data visualization libraries: "popular data analysis frameworks like Seurat,14 Giotto,15 Scanpy,16 and Squidpy17 derive their colorization approaches from widely used data visualization libraries"
- [methods] color_mapping = spaco.colorize(: "color_mapping = spaco.colorize("
