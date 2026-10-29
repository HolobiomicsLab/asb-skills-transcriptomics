---
name: degree-of-interlacement-graph-construction
description: Use when single-cell spatial transcriptomics data (e.g., Stereo-seq axolotl
  brain with 16 cell types) with per-cell coordinates and cluster labels must be summarized
  as a weighted cell-type-by-cell-type graph of spatial interlacement before spatially
  aware colorization.
when_to_use_negative:
- Input is non-spatial data (e.g., dissociated scRNA-seq without coordinates) — the
  DOI metric is undefined without spatial relationships
- Data are not at single-cell resolution or cell-type labels/clusters are absent,
  so no cell-type pairs can be evaluated
- You already have a user-specified color mapping and do not need spatially constrained
  palette refinement
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Spaco
  role: Computes the Degree of Interlacement metric and constructs the weighted CI-graph
    from cell-type spatial relationships
  repo: github.com/BrainStOrmics/Spaco
- name: Squidpy
  role: Loads the spatial dataset (coordinates plus annotations) used to build the
    CI-graph
- name: Scanpy
  role: Provides the AnnData object and cell-type labels whose spatial relationships
    the CI-graph models
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:9ee7f345d05300103b19e1ad8b7275b9a045d54881a32843a97a4d5878f09594
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Scanpy
  - Spaco
  - Squidpy
  - matplotlib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# degree-of-interlacement-graph-construction

## Summary

Construct Spaco's weighted cluster interlacement (CI-) graph by computing the Degree of Interlacement (DOI) metric between every pair of cell types from single-cell spatial coordinates. This graph globally models spatial relationships among cell clusters and drives Spaco's graph-guided palette selection.

## When to use

Use when single-cell spatial transcriptomics data (e.g., Stereo-seq axolotl brain with 16 cell types) with per-cell coordinates and cluster labels must be summarized as a weighted cell-type-by-cell-type graph of spatial interlacement before spatially aware colorization.

## When NOT to use

- Input is non-spatial data (e.g., dissociated scRNA-seq without coordinates) — the DOI metric is undefined without spatial relationships
- Data are not at single-cell resolution or cell-type labels/clusters are absent, so no cell-type pairs can be evaluated
- You already have a user-specified color mapping and do not need spatially constrained palette refinement

## Inputs

- Single-cell spatial transcriptomics data with per-cell spatial coordinates (e.g., Stereo-seq axolotl brain dataset, STARmap mouse brain, seqFISH mouse embryo, IMC human breast cancer)
- Cell-type/cluster labels per cell, loaded via Squidpy/Scanpy (e.g., sq.datasets.seqfish())
- AnnData object (h5ad) holding coordinates and annotations

## Outputs

- Weighted cluster interlacement (CI-) graph: a cell-type-by-cell-type matrix of Degree of Interlacement values modeling pairwise spatial relationships
- CI-graph consumed downstream by graph-guided palette selection (UMAP embedding in CIELab space) to produce a spatially aware color assignment

## How to apply

Load the spatially resolved dataset (cell spatial coordinates plus cell-type labels) via Squidpy/Scanpy, e.g., the Stereo-seq axolotl brain dataset with 16 cell types. For each cell-type pair, compute Spaco's Degree of Interlacement metric, which quantifies how isolated, adjacent, partially interlaced, or entirely intertwined the two cell types are across space — resolving the challenge of summarizing proximity for cell types comprising thousands of intricately distributed cells. Assemble the pairwise DOI values into a weighted cluster interlacement (CI-) graph whose nodes are cell types and whose edge weights encode spatial mixing. The CI-graph is then embedded (e.g., UMAP into CIELab space) during graph-guided adaptive palette selection, so the graph must faithfully reflect which cell-type pairs are spatial neighbors. Pin numpy to 1.22.x or 1.23.x, since the numpy version influences the result of the graph-guided mode downstream.

## Related tools

- **Spaco** (Computes the Degree of Interlacement metric and constructs the weighted CI-graph from cell-type spatial relationships) — github.com/BrainStOrmics/Spaco
- **Squidpy** (Loads the spatial dataset (coordinates plus annotations) used to build the CI-graph)
- **Scanpy** (Provides the AnnData object and cell-type labels whose spatial relationships the CI-graph models)

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git
```

## Evaluation signals

- The CI-graph is weighted and symmetric, with one node per cell type (e.g., 16 nodes for the axolotl brain dataset) and edge weights reflecting Degree of Interlacement
- Cell-type pairs that are spatially intertwined/adjacent receive higher interlacement weights than isolated pairs, matching known tissue topology
- Downstream graph-guided palette from this CI-graph yields pairwise CIEDE2000 color differences significantly higher than the Scanpy default palette (one-sided Wilcoxon p = 1.27e-11 on the axolotl dataset)
- numpy version is fixed at 1.22.x or 1.23.x, since it influences the graph-guided mode result
- Reproduction scripts from the BrainStOrmics/Spaco_scripts repository run end-to-end on the loaded Squidpy/Scanpy object

## Limitations

- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- The numpy version (1.22.x or 1.23.x) can change the result of the graph-guided mode, so exact reproducibility requires pinning the environment
- Determining overall proximity is nontrivial when a cell type comprises thousands of cells intricately distributed across space — the DOI metric is the paper's proposed resolution but is sensitive to how interlacement is summarized

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [intro] Spaco introduces the Degree of Interlacement (DOI) metric to evaluate the spatial relationship between individual cell-type pairs.: "Spaco introduces the Degree of Interlacement (DOI) metric to evaluate the spatial relationship between individual cell-type pairs."
- [intro] a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships.: "a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships."
- [intro] This poses the challenge of determining the overall proximity of one cell type, which may comprise thousands of cells and is intricately distributed across space, to another.: "This poses the challenge of determining the overall proximity of one cell type, which may comprise thousands of cells and is intricately distributed across space, to another."
- [methods] numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
