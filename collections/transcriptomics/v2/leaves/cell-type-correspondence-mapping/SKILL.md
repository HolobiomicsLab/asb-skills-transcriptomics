---
name: cell-type-correspondence-mapping
description: Use when visualizing two or more cellular-resolution SRT sections jointly
  (e.g., Stereo-seq axolotl DPI-10 vs DPI-15) and cell-type labels differ across sections,
  so naive default palettes misalign colors between biologically related or confusable
  cell types.
when_to_use_negative:
- Only one tissue section is being visualized — single-section colorization needs
  no cross-section correspondence mapping
- Cell-type labels are already consistent and identically named across sections and
  the default Scanpy/Seurat/Squidpy palette suffices
- Data is not single-cell spatial (SRT) data — e.g., scRNA-seq without spatial coordinates,
  since the Degree of Interlacement requires spatial relationships between cells
edam_operation: http://edamontology.org/operation_0337
tools:
- name: Spaco
  role: Builds per-section CI-graphs via Degree of Interlacement, performs greedy
    cross-section cluster correspondence mapping, and runs integrative cluster-color
    assignment
  repo: https://github.com/BrainStOrmics/Spaco
- name: Squidpy
  role: Loads the Stereo-seq axolotl sections (spatial coordinates and cell-type labels)
    as AnnData before colorization
- name: Scanpy
  role: Renders the colorized spatial maps via sc.pl.spatial and provides the default
    visualization baseline being improved upon
- name: numpy
  role: Dependency whose version (1.22.x or 1.23.x) must be pinned because it influences
    graph-guided mode results
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:84b1e36b87ab07bd4b78c77db8514ed6966aeb9a475f058f14151c90198b588c
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cell-type-correspondence-mapping

## Summary

Maps cell-type clusters across multiple spatially resolved transcriptomics tissue sections (e.g., Spaco's greedy correspondence mapping over merged cluster-interlacement graphs) so that corresponding cell types receive a shared, contrast-optimized color palette in integrative visualizations.

## When to use

Use when visualizing two or more cellular-resolution SRT sections jointly (e.g., Stereo-seq axolotl DPI-10 vs DPI-15) and cell-type labels differ across sections, so naive default palettes misalign colors between biologically related or confusable cell types.

## When NOT to use

- Only one tissue section is being visualized — single-section colorization needs no cross-section correspondence mapping
- Cell-type labels are already consistent and identically named across sections and the default Scanpy/Seurat/Squidpy palette suffices
- Data is not single-cell spatial (SRT) data — e.g., scRNA-seq without spatial coordinates, since the Degree of Interlacement requires spatial relationships between cells

## Inputs

- Cell spatial coordinates and cell-type labels per section (Squidpy/Scanpy AnnData objects, e.g., Stereo-seq axolotl DPI-10 and DPI-15 sections)
- Per-section CI-graphs built with Spaco's Degree of Interlacement metric
- A shared default color palette (Python list/dict) covering the union of cell types across sections

## Outputs

- A merged cross-section CI-graph with greedy cluster correspondence mappings
- An integrative color_mapping dictionary assigning perceptually distinct colors to each cell type consistently across sections
- Colorized spatial maps rendered per section (e.g., via sc.pl.spatial with spot_size=0.035)

## How to apply

Load each section's cell spatial coordinates and cell-type labels (e.g., Stereo-seq axolotl sections via Squidpy as in the Spaco_scripts vignette), pinning numpy to 1.22.x or 1.23.x since the graph-guided mode result is version-dependent. Build a per-section weighted cluster interlacement (CI) graph using Spaco's Degree of Interlacement metric, calling spaco.colorize with neighborhood parameters radius=0.05 and n_neighbors=30. Merge the per-section CI-graphs using Spaco's greedy correspondence mapping, which matches clusters across sections based on overlap/expression similarity (e.g., matching sfrpEGCs at DPI-15 against TLNBLs at DPI-10 so they are not conflated). Define a shared default palette and run integrative cluster-color assignment so colors are contrast-optimized jointly (maximized CIEDE2000 color difference) across both sections, then reorder the resulting color_mapping dictionary to the categorical order of cell-type labels before rendering with sc.pl.spatial (spot_size=0.035).

## Related tools

- **Spaco** (Builds per-section CI-graphs via Degree of Interlacement, performs greedy cross-section cluster correspondence mapping, and runs integrative cluster-color assignment) — https://github.com/BrainStOrmics/Spaco
- **Squidpy** (Loads the Stereo-seq axolotl sections (spatial coordinates and cell-type labels) as AnnData before colorization)
- **Scanpy** (Renders the colorized spatial maps via sc.pl.spatial and provides the default visualization baseline being improved upon)
- **numpy** (Dependency whose version (1.22.x or 1.23.x) must be pinned because it influences graph-guided mode results)

## Examples

```
import spaco; color_mapping = spaco.colorize(adata, radius=0.05, n_neighbors=30)
```

## Evaluation signals

- sfrpEGCs at DPI-15 and TLNBLs at DPI-10 receive perceptually distinct colors (maximized CIEDE2000 color difference under the shared palette)
- The increase in sfrpEGCs during wound healing becomes visible across the jointly colorized DPI-10/DPI-15 maps, unlike Scanpy's default visualization
- color_mapping dictionary keys cover the union of cell-type labels across both sections and can be reordered to the categorical label order without missing entries
- Output matches the reference Spaco_scripts vignette (Vignette/Demo.ipynb) result when numpy is pinned to 1.22.x or 1.23.x
- Each cell type is assigned the same color in both sections (correspondence mapping succeeded) with no duplicate color assignments within a section

## Limitations

- numpy version (1.22.x or 1.23.x) influences the result of Spaco's graph-guided mode, so results are version-dependent and must be pinned for reproducibility
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast through color space is limited when performing CVD-supporting colorization or coloring hundreds of clusters

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [methods] numpy version influences graph-guided mode results: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [methods] Spaco install command from methods: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
- [readme] README pointing to the Spaco_scripts vignette demo: "[Quick Example](https://github.com/BrainStOrmics/Spaco_scripts/blob/main/Vignette/Demo.ipynb)"
