---
name: color-difference-optimization-cielab
description: Use when spatially resolved data (e.g., Stereo-seq, STARmap, seqFISH,
  IMC) has many cell types whose default Scanpy/Seurat/Squidpy colors look similar
  in adjacent regions, causing perceptual ambiguity between biologically distinct
  neighboring clusters.
when_to_use_negative:
- 'Data is non-spatial (e.g., standard scRNA-seq): the CI-graph depends on spatial
  interlacement and there is no adjacency structure to optimize against'
- Hundreds of clusters must be colored, where visual contrast achievable through color
  space is limited regardless of assignment
- Colors must follow literature conventions rather than the spatial distribution of
  cells, since Spaco enforces a correspondence between color differences and spatial
  distribution
tools:
- name: Spaco
  role: 'Core tool: builds the CI-graph via Degree of Interlacement, performs adaptive
    palette selection and CIELab/CIEDE2000 color-difference optimization via spaco.colorize'
  repo: https://github.com/BrainStOrmics/Spaco
- name: SpacoR
  role: R/Seurat implementation of the same spatial colorization workflow
  repo: https://github.com/BrainStOrmics/SpacoR
- name: Squidpy
  role: Loads spatial transcriptomics datasets (e.g., sq.datasets.seqfish()) and provides
    the AnnData with spatial coordinates used as spaco.colorize input
- name: Scanpy
  role: Default palettes serve as the baseline being compared against; sc.pl.spatial
    renders the colorized maps
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:b83b049f694cec23513bab55f02d750752e85646ba6446dcc793da87695e8eac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Scanpy
  - Spaco
  - SpacoR
  - Squidpy
  - matplotlib
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# color-difference-optimization-cielab

## Summary

Spatially aware cluster-color assignment for single-cell spatial transcriptomics: Spaco builds a weighted cluster interlacement (CI-) graph from the Degree of Interlacement between cell types, optionally selects an adaptive palette, and assigns colors so that spatially adjacent cell types receive maximally distinct (CIEDE2000-measured) colors.

## When to use

Use when spatially resolved data (e.g., Stereo-seq, STARmap, seqFISH, IMC) has many cell types whose default Scanpy/Seurat/Squidpy colors look similar in adjacent regions, causing perceptual ambiguity between biologically distinct neighboring clusters.

## When NOT to use

- Data is non-spatial (e.g., standard scRNA-seq): the CI-graph depends on spatial interlacement and there is no adjacency structure to optimize against
- Hundreds of clusters must be colored, where visual contrast achievable through color space is limited regardless of assignment
- Colors must follow literature conventions rather than the spatial distribution of cells, since Spaco enforces a correspondence between color differences and spatial distribution

## Inputs

- AnnData object with cell spatial coordinates and cell-type/cluster labels (loaded via Squidpy/Scanpy)
- Neighborhood parameters for spaco.colorize (e.g., radius=0.05, n_neighbors=30)
- Optional pre-defined palette or fixed-color assignments for user-specified cell types
- For integrative mode: cell spatial coordinates and cell-type labels from multiple tissue sections (e.g., axolotl DPI-10 and DPI-15)

## Outputs

- color_mapping dictionary of cell type to optimized color
- Colorized spatial maps rendered with sc.pl.spatial (e.g., spot_size=0.035)
- Pairwise CIEDE2000 color-difference table for adjacent cell-type pairs (e.g., axolotl_colordiff_results.csv)
- Palette discernibility comparison plot (e.g., axolotl_colordiff_comparison.png)

## How to apply

Load the spatial data (cell coordinates plus cell-type labels) via Squidpy/Scanpy into an AnnData object and pin numpy to 1.22.x or 1.23.x, since the numpy version can change graph-guided mode results. Run spaco.colorize with neighborhood parameters radius=0.05 and n_neighbors=30 to build the per-section CI-graph using the Degree of Interlacement metric. Choose the palette mode: pre-defined, adaptive/graph-guided (which embeds the CI-graph into CIELab via UMAP and amplifies chromatic distinctions), or extract a palette from an image while maximizing color differentiation. Colors are assigned to maximize CIEDE2000 color differences between spatially adjacent cell-type pairs, with support for manually fixing colors of user-specified cell types and optimizing the rest, plus color-vision-deficiency accommodation. Reorder the returned color_mapping dictionary to the categorical label order before rendering spatial maps with sc.pl.spatial. For integrative colorization across multiple sections, merge CI-graphs via Spaco's greedy correspondence mapping so one shared palette covers all sections (e.g., resolving TLNBL vs sfrpEGC confusion between axolotl DPI-10 and DPI-15 sections).

## Related tools

- **Spaco** (Core tool: builds the CI-graph via Degree of Interlacement, performs adaptive palette selection and CIELab/CIEDE2000 color-difference optimization via spaco.colorize) — https://github.com/BrainStOrmics/Spaco
- **SpacoR** (R/Seurat implementation of the same spatial colorization workflow) — https://github.com/BrainStOrmics/SpacoR
- **Squidpy** (Loads spatial transcriptomics datasets (e.g., sq.datasets.seqfish()) and provides the AnnData with spatial coordinates used as spaco.colorize input)
- **Scanpy** (Default palettes serve as the baseline being compared against; sc.pl.spatial renders the colorized maps)

## Examples

```
color_mapping = spaco.colorize(adata, radius=0.05, n_neighbors=30)
```

## Evaluation signals

- Pairwise CIEDE2000 color-difference distribution for the adaptive palette is significantly higher than the pre-defined/Scanpy default palette (one-sided Wilcoxon signed-rank test; paper reports p = 1.27e-11 on the 16-type axolotl dataset)
- Spatially adjacent, biologically distinct cell-type pairs receive perceptually distinct colors under the shared palette (maximized CIEDE2000 difference)
- In integrative multi-section mode, corresponding clusters across sections (e.g., merged DPI-10/DPI-15 CI-graphs) share colors and formerly confused pairs (TLNBL vs sfrpEGC) are visually separable
- Reproduced results match the reference vignette (Spaco_scripts Vignette/Demo.ipynb) after reordering color_mapping to the categorical label order
- Reproducibility check: rerun with numpy pinned to 1.22.x or 1.23.x and confirm graph-guided mode output is stable

## Limitations

- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco and must be pinned for exact reproducibility
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or coloring hundreds of clusters
- 3D colorization requires fine-tuning of parameters to differentiate 'sparse' from 'dense' cell types
- The R version (SpacoR) is still in development and several functions/features are not yet implemented

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [abstract] Adaptive palette selection: "Spaco incorporates an adaptive palette selection approach to amplify chromatic distinctions"
- [intro] Colorization problem definition: "given a limited palette, how can we assign more contrasting colors to adjacent cell-type pairs?"
- [methods] numpy version caveat: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [methods] colorize API: "color_mapping = spaco.colorize("
- [readme] README description of Spaco: "Spaco (**spa**tial **co**lorization) provides a spatially constrained approach that generates discriminate color assignments for visualizing single-cell spatial data in various scenarios."
