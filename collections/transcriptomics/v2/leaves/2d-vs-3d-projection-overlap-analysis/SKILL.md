---
name: 2d-vs-3d-projection-overlap-analysis
description: Use when you need to quantify whether cell types or clusters mix less
  in a 3D embedding (UMAP/tSNE) than in its 2D counterpart, e.g. choosing projection
  dimensionality for scRNA-seq data.
when_to_use_negative:
- Embeddings carry no cell-type/cluster labels — overlap is computed two populations
  at a time, so unlabeled coordinates cannot be scored.
- You need overlap measured by convex-hull area rather than entropy — that variant
  is the separate 2d_vs_3d python code processing ConvexHullGenerator.cs output from
  CellexalVR.
- Your goal is in-VR interaction (cell selection, heatmaps, RNA velocity visualization)
  rather than 2D-vs-3D overlap quantification.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_3308
tools:
- name: entropy_2dvs3d
  role: 'Core toolkit: calculates the Shannon index for 2d and 3d graphs two populations
    at a time; the rotating_grid notebook contains the calculations plus heatmap/violinplot
    tools'
  repo: https://github.com/sonejilab/entropy_2dvs3d
- name: 2d_vs_3d
  role: Companion python code for the convex-hull-area overlap variant, processing
    and visualising output from ConvexHullGenerator.cs in CellexalVR
  repo: https://github.com/sonejilab/2d_vs_3d
- name: CellexalVR
  role: Source of 3D embeddings and convex-hull output consumed by the companion 2d_vs_3d
    analysis; platform where the projections are visualized
  repo: https://github.com/sonejilab/cellexalvr
- name: R
  role: Analysis runtime (R 4.0.0+) in which the entropy_2dvs3d code is set up and
    executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:aef913c3b13067cd60082d23500e6a95d6646a0d8c7109cbe51499b048c1e89d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/entropy_2dvs3d
  tools:
  - 2d_vs_3d
  - CellexalVR
  - R
  - entropy_2dvs3d
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# 2d-vs-3d-projection-overlap-analysis

## Summary

Quantifies pairwise cell-type/cluster overlap in 2D versus 3D dimension-reduction embeddings (UMAP, tSNE) using Shannon entropy as a measure of mixing, to test whether 3D projections separate cell populations better than 2D.

## When to use

Use when you need to quantify whether cell types or clusters mix less in a 3D embedding (UMAP/tSNE) than in its 2D counterpart, e.g. choosing projection dimensionality for scRNA-seq data.

## When NOT to use

- Embeddings carry no cell-type/cluster labels — overlap is computed two populations at a time, so unlabeled coordinates cannot be scored.
- You need overlap measured by convex-hull area rather than entropy — that variant is the separate 2d_vs_3d python code processing ConvexHullGenerator.cs output from CellexalVR.
- Your goal is in-VR interaction (cell selection, heatmaps, RNA velocity visualization) rather than 2D-vs-3D overlap quantification.

## Inputs

- 2D and 3D UMAP and tSNE cell-coordinate embeddings (e.g. mouse gastrulation GSE87038, Pijuan-Sala et al. 2019)
- Per-cell cell-type/cluster labels defining the populations compared pairwise
- entropy_2dvs3d code (rotating_grid notebook) run in an R 4.0.0+ environment

## Outputs

- entropy_2d_vs_3d_results.csv: per cell type/cluster 2D vs 3D least-overlap Shannon entropy for both UMAP and tSNE
- Heatmaps and violinplots demonstrating the benefit of 3D graphs over 2D
- Validated conclusion on whether reduction to three dimensions yields less pairwise overlap than two dimensions

## How to apply

Download precomputed 2D and 3D UMAP and tSNE cell-coordinate embeddings with per-cell cluster/cell-type labels (the paper uses the mouse gastrulation scRNA-seq dataset, Pijuan-Sala et al. 2019, GEO GSE87038), and set up the authors' entropy code from github.com/sonejilab/entropy_2dvs3d in an R 4.0.0+ environment. For each embedding, compute the Shannon Diversity index quantifying pairwise overlap of cell populations — the entropy_2dvs3d code considers populations two by two, and its main rotating_grid notebook contains both the plotting functions and the actual calculations. Systematically rotate the 3D embedding, flatten each rotation to 2D, compute the entropy for each rotated projection, and retain the minimum (least-overlap) entropy across rotations, giving the 3D projection its best case against the fixed 2D view. Tabulate the 2D versus 3D least-overlap entropy per cell type/cluster for both UMAP and tSNE and write the table to entropy_2d_vs_3d_results.csv, summarizing with the repo's heatmap and violinplot tools. Validate that the least-overlap 3D entropy is lower than the 2D entropy for both UMAP and tSNE, reproducing the paper's trend (Figures 2C and 2D); the rationale is that 3D projections are less prone to merging and flattening similar cell-types/clusters together.

## Related tools

- **entropy_2dvs3d** (Core toolkit: calculates the Shannon index for 2d and 3d graphs two populations at a time; the rotating_grid notebook contains the calculations plus heatmap/violinplot tools) — https://github.com/sonejilab/entropy_2dvs3d
- **2d_vs_3d** (Companion python code for the convex-hull-area overlap variant, processing and visualising output from ConvexHullGenerator.cs in CellexalVR) — https://github.com/sonejilab/2d_vs_3d
- **CellexalVR** (Source of 3D embeddings and convex-hull output consumed by the companion 2d_vs_3d analysis; platform where the projections are visualized) — https://github.com/sonejilab/cellexalvr
- **R** (Analysis runtime (R 4.0.0+) in which the entropy_2dvs3d code is set up and executed)

## Examples

```
git clone https://github.com/sonejilab/entropy_2dvs3d  # then run the rotating_grid notebook (main calculations) in R 4.0.0+
```

## Evaluation signals

- For every cell type/cluster, the retained least-overlap entropy across rotated 3D projections is lower than the original 2D entropy, for both UMAP and tSNE (Figures 2C/2D trend).
- entropy_2d_vs_3d_results.csv contains one 2D-vs-3D entropy row per cell type/cluster for each embedding (UMAP and tSNE).
- Each population overlap is computed two by two with the Shannon Diversity index, matching the entropy_2dvs3d README definition.
- Every systematic rotation of the 3D embedding yields a recorded entropy before the minimum is retained, confirming the rotate–flatten–entropy loop completed.
- The lower-overlap-in-3D trend replicates across both embeddings and, as reported, across two further large datasets using convex hulls as a second measure of overlap.

## Limitations

- Demonstrated on large datasets (mouse gastrulation GSE87038 plus two further large datasets); the article frames the 3D benefit for larger and complex datasets, so small datasets are untested here.
- The 3D score depends on the systematic rotation scheme in the rotating_grid notebook — the minimum-overlap rotation is retained, so results hinge on how rotations are sampled.
- Requires precomputed embeddings and population labels; the skill does not cover generating UMAP/tSNE projections or assigning clusters.

## Evidence

- [results] for cell-types and clusters (Figures 2C and 2D respectively) reduction to three-dimensions resulted in less overlap than in two-dimensions.: "for cell-types and clusters (Figures 2C and 2D respectively) reduction to three-dimensions resulted in less overlap than in two-dimensions."
- [abstract] Three-dimensional projections can be more informative for larger and complex datasets because they are less prone to merging and flattening similar cell-types/clusters together.: "Three-dimensional projections can be more informative for larger and complex datasets because they are less prone to merging and flattening similar cell-types/clusters together."
- [readme] Calculating Shannon index for 2d and 3d graphs. Tools to create heatmaps and violinplots to demonstrate benefit of 3d graphs over 2d.: [evidence span withheld — anchor status: external_doc]
- [readme] The main notebook is the rotating_grid one which contains both the plotting functions and the actual calculations.: [evidence span withheld — anchor status: external_doc]
- [discussion] Code for this method can be obtained at https://github.com/sonejilab/entropy_2dvs3d: "Code for this method can be obtained at https://github.com/sonejilab/entropy_2dvs3d"
- [results] We saw this trend consistently when applying this procedure to two further large datasets where we also used convex hulls as a second measure of overlap: "We saw this trend consistently when applying this procedure to two further large datasets where we also used convex hulls as a second measure of overlap"
