---
name: ci-graph-merging-across-sections
description: Use when visualizing two or more spatially resolved tissue sections with
  shared cell-type labels (e.g., Stereo-seq axolotl regeneration time points) and
  default per-section colorization mismatches or visually confuses corresponding cell
  types across sections.
when_to_use_negative:
- When only a single section is being colored — per-section colorization without CI-graph
  merging is sufficient and simpler
- When the goal is batch correction or label transfer between sections — Spaco only
  assigns colors and does not correct or align biological content
- When coloring hundreds of clusters or requiring CVD-supporting palettes, where obtainable
  visual contrast through color space is limited
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: Builds per-section CI-graphs via the Degree of Interlacement metric and performs
    greedy correspondence mapping and integrative cluster-color assignment (spaco.colorize)
  repo: https://github.com/BrainStOrmics/Spaco
- name: Squidpy
  role: Loads the Stereo-seq spatial sections (spatial coordinates and cell-type labels)
    as AnnData objects
- name: Scanpy
  role: Renders the colorized spatial maps via sc.pl.spatial (spot_size=0.035)
- name: numpy
  role: Must be pinned to 1.22.x or 1.23.x because the graph-guided mode result is
    version-dependent
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:9f83bb10ecb59cc5ee98b6689fa2548132ca549a7befc5729da0eeef139062f9
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
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# ci-graph-merging-across-sections

## Summary

Spaco's integrative colorization merges cluster interlacement (CI)-graphs built independently from multiple tissue sections so that cell-type colors are contrast-optimized jointly, resolving cross-section ambiguities such as telencephalon neuroblasts (TLNBLs) at axolotl DPI-10 being confused with Sfrp+ ependymoglial cells (sfrpEGCs) at DPI-15.

## When to use

Use when visualizing two or more spatially resolved tissue sections with shared cell-type labels (e.g., Stereo-seq axolotl regeneration time points) and default per-section colorization mismatches or visually confuses corresponding cell types across sections.

## When NOT to use

- When only a single section is being colored — per-section colorization without CI-graph merging is sufficient and simpler
- When the goal is batch correction or label transfer between sections — Spaco only assigns colors and does not correct or align biological content
- When coloring hundreds of clusters or requiring CVD-supporting palettes, where obtainable visual contrast through color space is limited

## Inputs

- Per-section cell spatial coordinate tables with cell-type/cluster annotations (e.g., Stereo-seq axolotl DPI-10 and DPI-15 sections loaded as AnnData via Squidpy)
- Shared cell-type label set spanning all sections to be integrated
- A shared default color palette

## Outputs

- Merged CI-graph spanning all input sections
- A color_mapping dictionary assigning jointly contrast-optimized colors to each cell type, consistent across sections
- Colorized spatial maps for each section (e.g., rendered with sc.pl.spatial, spot_size=0.035)

## How to apply

Load each section's spatial coordinates and cell-type labels with Squidpy, then build a per-section weighted CI-graph using Spaco's Degree of Interlacement metric via spaco.colorize with neighborhood parameters radius=0.05 and n_neighbors=30. Merge the CI-graphs (e.g., DPI-10 and DPI-15 axolotl sections) with Spaco's greedy correspondence mapping of clusters based on overlap/expression similarity across sections. Define a shared default palette and run integrative cluster-color assignment with spaco.colorize so colors are contrast-optimized jointly, maximizing CIEDE2000 color differences for interlaced cell-type pairs. Pin numpy to 1.22.x or 1.23.x because the graph-guided mode result is version-dependent. Reorder the resulting color_mapping dictionary to the categorical order of the cell-type labels before rendering spatial maps (e.g., sc.pl.spatial with spot_size=0.035).

## Related tools

- **Spaco** (Builds per-section CI-graphs via the Degree of Interlacement metric and performs greedy correspondence mapping and integrative cluster-color assignment (spaco.colorize)) — https://github.com/BrainStOrmics/Spaco
- **Squidpy** (Loads the Stereo-seq spatial sections (spatial coordinates and cell-type labels) as AnnData objects)
- **Scanpy** (Renders the colorized spatial maps via sc.pl.spatial (spot_size=0.035))
- **numpy** (Must be pinned to 1.22.x or 1.23.x because the graph-guided mode result is version-dependent)

## Examples

```
color_mapping = spaco.colorize(aDATA_dpi10, aDATA_dpi15, radius=0.05, n_neighbors=30)
```

## Evaluation signals

- sfrpEGCs at DPI-15 and TLNBLs at DPI-10 receive perceptually distinct colors, with maximized CIEDE2000 color difference under the shared palette
- Corresponding cell types across sections are assigned identical colors via the greedy correspondence mapping
- The color_mapping dictionary covers every cell-type label and can be reordered to the categorical label order without missing keys
- Output spatial maps match the reference Spaco_scripts vignette (Vignette/Demo.ipynb) result when numpy is pinned to 1.22.x or 1.23.x
- The biological signal (increase in sfrpEGCs during wound healing) is visible in the integrated rendering, unlike under Scanpy's default visualization

## Limitations

- numpy version (1.22.x or 1.23.x) influences the result of the graph-guided mode of Spaco, so results may not be reproducible across environments
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or when coloring hundreds of clusters

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [methods] colorize in methods: "color_mapping = spaco.colorize("
- [methods] numpy version dependency: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [discussion] manual color fixing option: "Spaco offers an option allowing users to manually fix or adjust colors for user-specified cell types and then optimize the color differences for the remaining cell types."
- [readme] Scripts to reproduce benchmarking and analytic results: "Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts](https://github.com/BrainStOrmics/Spaco_scripts)"
