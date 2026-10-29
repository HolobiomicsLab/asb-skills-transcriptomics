---
name: benchmark-distribution-visualization
description: Use when you must show that a spatially aware colorization method (e.g.
  Spaco's graph-guided palette) yields significantly more discernible colors than
  a default palette, using pairwise color-difference distributions.
when_to_use_negative:
- Data has no spatial coordinates or cell-type adjacency information, so the CI-graph
  restriction to spatially adjacent pairs cannot be constructed
- The goal is qualitative palette inspection or manual color fixing for specific cell
  types rather than quantitative benchmarking against a baseline
- Coloring hundreds of clusters, where the article notes visual contrast obtainable
  through color space is limited and pairwise-difference benchmarks lose meaning
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: Generates the spatially aware adaptive palette under test via CI-graph-guided
    selection in CIELab space
  repo: https://github.com/BrainStOrmics/Spaco
- name: Scanpy
  role: Loads the labeled spatial dataset and supplies the default baseline palette
    for comparison
- name: Squidpy
  role: Loads the Stereo-seq spatial data with cell spatial coordinates used to build
    adjacency
- name: matplotlib
  role: Renders the distribution comparison plot of pairwise color differences
- name: Python
  role: Implementation environment for the benchmark scripts (Spaco_scripts reproduction
    scripts)
  repo: https://github.com/BrainStOrmics/Spaco_scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:351647950c263b680abc0a0a59ece980476f053a9ae77f3f419a8bf12eacb45b
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# benchmark-distribution-visualization

## Summary

Benchmark a colorization method by quantifying the distribution of pairwise color differences (CIEDE2000 in CIELab) it produces versus a baseline palette, testing the shift with a one-sided Wilcoxon signed-rank test and rendering the distribution comparison with matplotlib.

## When to use

Use when you must show that a spatially aware colorization method (e.g. Spaco's graph-guided palette) yields significantly more discernible colors than a default palette, using pairwise color-difference distributions.

## When NOT to use

- Data has no spatial coordinates or cell-type adjacency information, so the CI-graph restriction to spatially adjacent pairs cannot be constructed
- The goal is qualitative palette inspection or manual color fixing for specific cell types rather than quantitative benchmarking against a baseline
- Coloring hundreds of clusters, where the article notes visual contrast obtainable through color space is limited and pairwise-difference benchmarks lose meaning

## Inputs

- Cell-by-coordinate spatial transcriptomics data with cell-type labels (e.g. Stereo-seq axolotol brain dataset with 16 cell types, loaded via Squidpy/Scanpy)
- Two color palettes to compare: the method's adaptively selected palette and a baseline default palette (e.g. Scanpy's)
- Cluster interlacement (CI-) graph of spatially adjacent cell-type pairs built from the Degree of Interlacement metric

## Outputs

- Pairwise color-difference table (CIEDE2000 values per cell-type pair, per palette), e.g. axolotl_colordiff_results.csv
- Distribution comparison plot, e.g. axolotl_colordiff_comparison.png rendered with matplotlib
- One-sided Wilcoxon signed-rank test p-value comparing the paired color-difference distributions

## How to apply

Run the colorization method under test (e.g. Spaco's graph-guided adaptive palette selection, which embeds the cluster interlacement (CI-) graph into CIELab space via UMAP) and a baseline (e.g. the Scanpy default palette) on the same labeled data. For each palette compute pairwise color differences using CIEDE2000 in CIELab, restricted to spatially adjacent cell-type pairs as defined by the Degree of Interlacement-based CI-graph. Perform a one-sided Wilcoxon signed-rank test comparing the paired color-difference distributions (in the axolotl case, Spaco's discernibility was significantly higher, p = 1.27e-11 versus the Scanpy default). Write the pairwise color-difference table to CSV and render the distribution comparison plot (e.g. with matplotlib) so the upward shift of the method's distribution is visually verifiable.

## Related tools

- **Spaco** (Generates the spatially aware adaptive palette under test via CI-graph-guided selection in CIELab space) — https://github.com/BrainStOrmics/Spaco
- **Scanpy** (Loads the labeled spatial dataset and supplies the default baseline palette for comparison)
- **Squidpy** (Loads the Stereo-seq spatial data with cell spatial coordinates used to build adjacency)
- **matplotlib** (Renders the distribution comparison plot of pairwise color differences)
- **Python** (Implementation environment for the benchmark scripts (Spaco_scripts reproduction scripts)) — https://github.com/BrainStOrmics/Spaco_scripts

## Examples

```
color_mapping = spaco.colorize(adata, color_basis='cluster', mode='graph-guided')
```

## Evaluation signals

- Reproduced one-sided Wilcoxon signed-rank p-value matches the reference value 1.27e-11 within numerical tolerance
- Spaco's pairwise color-difference distribution is shifted upward relative to the Scanpy default palette distribution
- Pairwise differences are computed with CIEDE2000 in CIELab and restricted to spatially adjacent cell-type pairs from the CI-graph
- Output files axolotl_colordiff_results.csv (pairwise table) and axolotl_colordiff_comparison.png (plot) exist with expected schema
- Number of paired comparisons matches the number of spatially adjacent cell-type pairs (16 cell types in the axolotl dataset)

## Limitations

- numpy version (1.22.x or 1.23.x) can influence the result of the graph-guided mode of Spaco and must be fixed for exact reproducibility
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast obtainable through color space is limited with CVD-supporting colorization or when coloring hundreds of clusters

## Evidence

- [abstract] Spaco incorporates an adaptive palette selection approach to amplify chromatic distinctions: "Spaco incorporates an adaptive palette selection approach to amplify chromatic distinctions"
- [intro] CI-graph construction via Degree of Interlacement: "Spaco introduces the Degree of Interlacement (DOI) metric to evaluate the spatial relationship between individual cell-type pairs."
- [methods] numpy version caveat: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [methods] colorize API call: "color_mapping = spaco.colorize("
- [methods] reproduction scripts repository: "Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts](https://github.com/BrainStOrmics/Spaco_scripts)"
- [intro] Spaco colorization modules: "the Spaco colorization process is composed of three primary modules: (1) cell cluster interlacement modeling, (2) color palette selection, and (3) cluster-color assignment"
