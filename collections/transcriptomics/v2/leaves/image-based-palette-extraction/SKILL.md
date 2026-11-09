---
name: image-based-palette-extraction
description: Use when the goal is to adopt a coloring theme from a publication figure
  or cover image and apply it to spatial transcriptomics cell-type plots while keeping
  palette discernibility comparable to standard palettes like matplotlib tab20.
when_to_use_negative:
- When the palette is already predefined (e.g., matplotlib tab20 or ggplot2 defaults)
  and only cluster-to-color assignment needs spatial optimization — use Spaco's graph-guided
  colorization instead of palette extraction
- When user-specified cell types must retain fixed colors per literature conventions
  — Spaco's enforcement of color-difference/spatial-distribution correspondence may
  not match such preferences
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: Performs image-guided palette extraction (CIELab frequency binning, CIEDE2000
    frequency updates, FPS refinement) and colorizes clusters via spaco.colorize
  repo: https://github.com/BrainStOrmics/Spaco
- name: Squidpy
  role: Loads the spatial dataset (cell coordinates and cell-type labels) and example
    image data
- name: Scanpy
  role: Handles the single-cell AnnData object holding cell-type labels
- name: matplotlib
  role: Provides the tab20 reference palette used as the discernibility baseline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:d79fd1ab526bde7635b848b68c7306d6b8cb70623990341639b8baf02f0ee599
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# image-based-palette-extraction

## Summary

Extract a coloring palette from a theme image (e.g., a journal cover) using Spaco's image-guided palette extraction, which bins colors by frequency in CIELab space, updates frequencies with the CIEDE2000 color-difference metric, and refines with farthest point sampling so the theme color is preserved while color differentiation is maximized.

## When to use

Use when the goal is to adopt a coloring theme from a publication figure or cover image and apply it to spatial transcriptomics cell-type plots while keeping palette discernibility comparable to standard palettes like matplotlib tab20.

## When NOT to use

- When the palette is already predefined (e.g., matplotlib tab20 or ggplot2 defaults) and only cluster-to-color assignment needs spatial optimization — use Spaco's graph-guided colorization instead of palette extraction
- When user-specified cell types must retain fixed colors per literature conventions — Spaco's enforcement of color-difference/spatial-distribution correspondence may not match such preferences

## Inputs

- Theme image (e.g., a journal cover image) from which colors are extracted
- Spatially resolved dataset with cell coordinates and cell-type labels (e.g., Stereo-seq axolotl brain dataset loaded via Squidpy)
- Number of clusters / desired palette size

## Outputs

- Extracted color palette (list of RGB/hex colors) preserving the image's theme
- Colored cell-type cluster plot produced by spaco.colorize
- Pairwise CIEDE2000 color-difference distributions for the extracted palette vs. tab20, plotted as color_difference_distribution.png with Wilcoxon signed-rank test p-value

## How to apply

Load the theme image and the spatial dataset (e.g., the Stereo-seq axolotl brain dataset with cell coordinates and cell-type labels) using Squidpy. Run Spaco's image-guided palette extraction on the image: bin colors by frequency in CIELab space, update frequencies using the CIEDE2000 color-difference metric, then refine the candidate colors with farthest point sampling (FPS) so the theme color is maintained while color differentiation is maximized. Colorize the cell-type clusters with spaco.colorize using the extracted palette. To validate discernibility, compute pairwise CIEDE2000 color differences among the Spaco-assigned colors and among matplotlib tab20 colors, and compare the two distributions with a Wilcoxon signed-rank test; the paper reports p = 1.20e-11 for Spaco versus tab20 on the axolotl dataset.

## Related tools

- **Spaco** (Performs image-guided palette extraction (CIELab frequency binning, CIEDE2000 frequency updates, FPS refinement) and colorizes clusters via spaco.colorize) — https://github.com/BrainStOrmics/Spaco
- **Squidpy** (Loads the spatial dataset (cell coordinates and cell-type labels) and example image data)
- **Scanpy** (Handles the single-cell AnnData object holding cell-type labels)
- **matplotlib** (Provides the tab20 reference palette used as the discernibility baseline)

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git; color_mapping = spaco.colorize(adata, palette=extracted_palette)
```

## Evaluation signals

- Reproduced Wilcoxon signed-rank test p-value matches the paper's reported value of 1.20e-11 for Spaco-extracted palette vs. matplotlib tab20
- color_difference_distribution.png exists and shows pairwise CIEDE2000 distributions for both palettes
- Extracted palette retains the theme color of the source image while pairwise color differences remain comparable to tab20
- spaco.colorize returns a color mapping covering all cell-type clusters in the dataset

## Limitations

- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or when coloring hundreds of clusters
- numpy version (1.22.x or 1.23.x) could influence the result of the graph-guided mode of Spaco

## Evidence

- [methods] Palette extraction preserves theme while maximizing differentiation: "We provide a method for extracting color plates from images. While maintaining the theme color, the color differentiation is maximized."
- [methods] spaco.colorize API entry point: "color_mapping = spaco.colorize("
- [abstract] Spaco accommodates CVD and open code: "Spaco ensures broad accessibility by accommodating color vision deficiency and offering open-accessible code in both Python and R"
- [methods] Installation from GitHub repository: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
