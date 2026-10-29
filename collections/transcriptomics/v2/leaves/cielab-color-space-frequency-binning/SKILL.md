---
name: cielab-color-space-frequency-binning
description: Use when a coloring palette must be derived from a reference image (e.g.,
  a journal cover figure) so that visualizations preserve the image's theme while
  remaining discriminable against baselines like matplotlib tab20.
when_to_use_negative:
- When no reference image or thematic palette is needed and the goal is purely spatially
  aware coloring of adjacent cell types — use Spaco's graph-guided (Degree of Interlacement
  / CI-graph) mode instead
- When coloring hundreds of clusters or requiring CVD-supporting colorization, where
  visual contrast obtainable through color space is limited
- When the input is already a curated palette with adequate pairwise contrast — extraction
  would add no benefit
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: Provides the image-guided palette extraction (CIELab frequency binning, CIEDE2000
    frequency updating, FPS refinement) and spaco.colorize for applying the palette
    to cell-type clusters
  repo: https://github.com/BrainStOrmics/Spaco
- name: Squidpy
  role: Loads the spatial dataset (cell coordinates and cell-type labels, e.g., the
    Stereo-seq axolotl brain dataset) for colorization
- name: matplotlib
  role: Supplies the tab20 baseline palette against which pairwise CIEDE2000 color
    differences of the extracted palette are compared
- name: numpy
  role: Numerical backend for frequency/difference computations; note version sensitivity
    of Spaco modes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:f789192f6d33417d3e46265966afa7983b09780ad063c33172322c059db67810
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - Squidpy
  - matplotlib
  - numpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cielab-color-space-frequency-binning

## Summary

Image-guided palette extraction used by Spaco: colors are binned by frequency in CIELab space, frequencies are updated with the CIEDE2000 color-difference metric, and farthest point sampling (FPS) refines the set so the source image's theme color is maintained while color differentiation is maximized.

## When to use

Use when a coloring palette must be derived from a reference image (e.g., a journal cover figure) so that visualizations preserve the image's theme while remaining discriminable against baselines like matplotlib tab20.

## When NOT to use

- When no reference image or thematic palette is needed and the goal is purely spatially aware coloring of adjacent cell types — use Spaco's graph-guided (Degree of Interlacement / CI-graph) mode instead
- When coloring hundreds of clusters or requiring CVD-supporting colorization, where visual contrast obtainable through color space is limited
- When the input is already a curated palette with adequate pairwise contrast — extraction would add no benefit

## Inputs

- Reference image (e.g., journal cover) from which the color theme is extracted
- Spatially resolved single-cell dataset with cell coordinates and cell-type labels (e.g., Stereo-seq axolotl brain dataset loaded via Squidpy)
- Optional baseline palette for comparison (e.g., matplotlib tab20)

## Outputs

- Extracted, theme-preserving color palette (list of CIELab/RGB colors)
- Color mapping assigned to cell-type clusters via spaco.colorize
- Pairwise CIEDE2000 color-difference distributions for extracted vs. baseline palettes
- Wilcoxon signed-rank test p-value comparing the two distributions (reported: 1.20e-11)
- Comparison figure (e.g., color_difference_distribution.png)

## How to apply

Load the reference image (e.g., a journal cover) and, if colorizing data, the spatial dataset such as the Stereo-seq axolotl brain dataset with cell coordinates and cell-type labels via Squidpy. Run Spaco's palette extraction: bin image colors by frequency in CIELab space, update the frequencies using the CIEDE2000 color-difference metric, then refine the candidate colors with farthest point sampling (FPS) so the theme color is kept while differentiation between colors is maximized. Feed the extracted palette into spaco.colorize to assign colors to cell-type clusters. Validate perceptual quality by computing pairwise CIEDE2000 differences among the extracted colors and comparing the distribution to a standard palette such as matplotlib's tab20 with a Wilcoxon signed-rank test (Spaco vs tab20 reported p = 1.20e-11 on the axolotl dataset).

## Related tools

- **Spaco** (Provides the image-guided palette extraction (CIELab frequency binning, CIEDE2000 frequency updating, FPS refinement) and spaco.colorize for applying the palette to cell-type clusters) — https://github.com/BrainStOrmics/Spaco
- **Squidpy** (Loads the spatial dataset (cell coordinates and cell-type labels, e.g., the Stereo-seq axolotl brain dataset) for colorization)
- **matplotlib** (Supplies the tab20 baseline palette against which pairwise CIEDE2000 color differences of the extracted palette are compared)
- **numpy** (Numerical backend for frequency/difference computations; note version sensitivity of Spaco modes)

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git
```

## Evaluation signals

- Pairwise CIEDE2000 color differences of the extracted palette are comparable to matplotlib tab20 (Wilcoxon signed-rank test p = 1.20e-11 on the axolotl dataset)
- The dominant theme color of the source image is preserved in the extracted palette after FPS refinement
- spaco.colorize returns a color_mapping covering every cell type in the dataset without duplicates
- The comparison figure (color_difference_distribution.png) is produced and contains both pairwise-difference distributions
- Extraction runs reproducibly under numpy 1.22.x or 1.23.x, the versions the paper notes can influence Spaco results

## Limitations

- The numpy version (1.22.x or 1.23.x) can influence Spaco results, affecting reproducibility of the extracted/colorized output
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or coloring hundreds of clusters
- Extracted palettes enforce a correspondence between color differences and the source image/data that may not match user preferences or literature conventions

## Evidence

- [methods] Palette extraction from image while maximizing differentiation: "We provide a method for extracting color plates from images. While maintaining the theme color, the color differentiation is maximized."
- [readme] Spaco's spatially constrained color assignment purpose: "Spaco (**spa**tial **co**lorization) provides a spatially constrained approach that generates discriminate color assignments for visualizing single-cell spatial data in various scenarios."
- [methods] CIEDE2000 metric as the perceptual difference measure: "By quantifying the complex topology between cell type clusters, We optimized color assignment of to achieve better visual recognizability."
