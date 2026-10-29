---
name: spatial-transcriptomics-colorization
description: Use when visualizing cell-type clusters from single-cell-resolution SRT
  data (STARmap, Stereo-seq, seqFISH, IMC) where default palettes (Scanpy/matplotlib)
  cause perceptual ambiguity between spatially adjacent or interlaced cell types.
when_to_use_negative:
- The input is bulk or non-spatial data where cell-type spatial adjacency is meaningless
  — Spaco's optimization is driven by spatial interlacement.
- Hundreds of clusters must be colored or strict CVD-supporting colorization is required
  — the article notes visual contrast obtainable through color space is limited in
  these cases.
- The user must follow literature-conventional colors rather than spatial-contrast-driven
  ones — Spaco enforces a correspondence between color differences and spatial distribution
  that may not match user preferences.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3608
tools:
- name: Spaco
  role: 'Core tool: builds the CI-graph, selects palettes (adaptive or image-guided),
    and assigns colors via spaco.colorize'
  repo: https://github.com/BrainStOrmics/Spaco
- name: Squidpy
  role: Loads SRT datasets (cell coordinates and labels) such as sq.datasets.seqfish()
    for colorization input
- name: Scanpy
  role: Handles AnnData objects and provides the default baseline palette for comparison
    and sc.pl.spatial rendering
- name: SpacoR
  role: R implementation for Seurat-based workflows; several functions and features
    not yet implemented
  repo: https://github.com/BrainStOrmics/SpacoR
- name: Spaco_scripts
  role: Benchmark and vignette scripts to reproduce the paper's colorization results
    (e.g., Vignette/Demo.ipynb)
  repo: https://github.com/BrainStOrmics/Spaco_scripts
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_004
  - task_003
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:3599d596990d03f5a00151dc9cb63c26c9faf961b17dc39e99b8a8f81b88fe95
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
  - matplotlib
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3608
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7727272727272727
n_claims: 22
---

# spatial-transcriptomics-colorization

## Summary

Spatially aware colorization of single-cell-resolution spatially resolved transcriptomics (SRT) data using Spaco, which builds a weighted cluster interlacement (CI-) graph via the Degree of Interlacement (DOI) metric and optimizes cluster-color assignment so adjacent cell types receive maximally contrasting colors (CIEDE2000).

## When to use

Use when visualizing cell-type clusters from single-cell-resolution SRT data (STARmap, Stereo-seq, seqFISH, IMC) where default palettes (Scanpy/matplotlib) cause perceptual ambiguity between spatially adjacent or interlaced cell types.

## When NOT to use

- The input is bulk or non-spatial data where cell-type spatial adjacency is meaningless — Spaco's optimization is driven by spatial interlacement.
- Hundreds of clusters must be colored or strict CVD-supporting colorization is required — the article notes visual contrast obtainable through color space is limited in these cases.
- The user must follow literature-conventional colors rather than spatial-contrast-driven ones — Spaco enforces a correspondence between color differences and spatial distribution that may not match user preferences.

## Inputs

- Cell spatial coordinates and cell-type/cluster labels (e.g., AnnData loaded via Squidpy, e.g., sq.datasets.seqfish())
- Optional pre-defined or shared color palette (e.g., Scanpy default, matplotlib tab20)
- Optional source image for image-guided palette extraction (e.g., a journal cover image)

## Outputs

- color_mapping dictionary assigning a color to each cell-type cluster
- Colorized spatial maps rendered with sc.pl.spatial (e.g., spot_size=0.035)
- Pairwise CIEDE2000 color-difference tables and Wilcoxon test p-values (e.g., wilcoxon_pvalues.csv)
- Palette-distribution comparison figures (e.g., color_difference_distribution.png)

## How to apply

Run the three Spaco modules: (1) cell cluster interlacement modeling, which computes the Degree of Interlacement between every cell-type pair from cell spatial coordinates and labels and constructs a weighted CI-graph; (2) color palette selection, either adaptive (embedding the CI-graph into CIELab via UMAP and selecting colors that amplify chromatic distinctions) or image-guided (extracting a theme palette from an image by binning CIELab colors and refining with farthest point sampling); and (3) cluster-color assignment via spaco.colorize. For graph-guided mode on the STARmap benchmark, use radius=0.05 and n_neighbors=30, and pin numpy to 1.22.x or 1.23.x since the graph-guided result is version-dependent. For multi-section data (e.g., axolotl DPI-10 and DPI-15), merge per-section CI-graphs by greedy correspondence mapping of clusters so colors are contrast-optimized jointly across sections. Evaluate by computing CIEDE2000 color differences for anatomically adjacent cell-type pairs and comparing against Scanpy default, random, and Palo baselines with one-sided Wilcoxon tests, and optionally regress realized color differences on DOI scores.

## Related tools

- **Spaco** (Core tool: builds the CI-graph, selects palettes (adaptive or image-guided), and assigns colors via spaco.colorize) — https://github.com/BrainStOrmics/Spaco
- **Squidpy** (Loads SRT datasets (cell coordinates and labels) such as sq.datasets.seqfish() for colorization input)
- **Scanpy** (Handles AnnData objects and provides the default baseline palette for comparison and sc.pl.spatial rendering)
- **SpacoR** (R implementation for Seurat-based workflows; several functions and features not yet implemented) — https://github.com/BrainStOrmics/SpacoR
- **Spaco_scripts** (Benchmark and vignette scripts to reproduce the paper's colorization results (e.g., Vignette/Demo.ipynb)) — https://github.com/BrainStOrmics/Spaco_scripts

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git; color_mapping = spaco.colorize(adata, radius=0.05, n_neighbors=30)
```

## Evaluation signals

- STARmap benchmark: one-sided Wilcoxon p-values for adjacent-pair color differences of 1.45e-4 vs Scanpy, 2.32e-3 vs random, 4.51e-3 vs Palo (reference values; FOV means 8.42e-21, 9.16e-19, 8.87e-21).
- Axolotl adaptive palette: one-sided Wilcoxon p = 1.27e-11 vs pre-defined palette, with Spaco's pairwise CIEDE2000 distribution shifted upward.
- DOI-to-color-difference linear regression reproduces R-square = 0.45 with p = 4e-61 (accept R-square within 0.40–0.50 and p of the same order).
- Integrative colorization: TLNBLs (DPI-10) and sfrpEGCs (DPI-15) receive perceptually distinct colors (maximized CIEDE2000 under the shared palette), resolving the confusion seen with Scanpy defaults.
- Runtime: full colorization pipeline completes in under 30 s for simulated datasets of up to 100,000 cells.

## Limitations

- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco — pin the version for reproducibility.
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions.
- Visual contrast is limited when performing CVD-supporting colorization or coloring hundreds of clusters.
- 3D colorization requires fine-tuning of parameters to differentiate 'sparse' from 'dense' cell types.
- The pure-R SpacoR implementation is incomplete: several functions and features are not yet implemented.

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [intro] the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment: "the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment"
- [methods] pip install git+https://github.com/BrainStOrmics/Spaco.git: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
- [methods] numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [methods] color_mapping = spaco.colorize(: "color_mapping = spaco.colorize("
- [methods] Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts]: "Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts](https://github.com/BrainStOrmics/Spaco_scripts)"
- [readme] Spaco (spatial colorization) provides a spatially constrained approach that generates discriminate color assignments for visualizing single-cell spatial data: "Spaco (**spa**tial **co**lorization) provides a spatially constrained approach that generates discriminate color assignments for visualizing single-cell spatial data in various scenarios."
- [discussion] One major limitation of Spaco is its enforcement of a correspondence between color differences and the spatial distribution of cells.: "One major limitation of Spaco is its enforcement of a correspondence between color differences and the spatial distribution of cells."
