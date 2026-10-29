---
name: 3d-projection-angle-flattening-analysis
description: Use when comparing whether 3D single-cell dimension-reduction projections
  (UMAP/tSNE) separate annotated cell types or clusters better than 2D, quantified
  by pairwise Shannon-entropy mixing or convex-hull area overlap across swept viewing
  angles.
when_to_use_negative:
- Input is raw scRNA-seq expression counts rather than computed 2D/3D embedding coordinates
  with cell-type/cluster labels — dimension reduction must be performed first.
- The goal is interactive, immersive visualization or analysis of the 3D projection;
  use CellexalVR itself instead of this batch overlap-quantification analysis.
- You need per-cell differential-expression, heatmap marker, or pseudotime/slingshot
  trajectory analysis rather than population-overlap quantification.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_4043
- http://edamontology.org/topic_3307
tools:
- name: entropy_2dvs3d
  role: Provides the rotating_grid notebook that computes the Shannon Diversity index
    of pairwise population overlap for 2D graphs and for 3D graphs rotated and flattened
    across viewing angles.
  repo: https://github.com/sonejilab/entropy_2dvs3d
- name: 2d_vs_3d
  role: Python code that processes and visualises the convex-hull overlap output from
    CellexalVR's ConvexHullGenerator.cs, sweeping viewing angles and writing hull_overlap_results.csv.
  repo: https://github.com/sonejilab/2d_vs_3d
- name: CellexalVR
  role: Source platform whose ConvexHullGenerator.cs (ConvexHullOverlap branch) computes
    population convex hulls whose overlap the 2d_vs_3d code analyses.
  repo: https://github.com/sonejilab/cellexalvr
- name: R
  role: Runtime (R 4.0.0+) in which the entropy_2dvs3d analysis is set up and executed.
- name: Python
  role: Language of the 2d_vs_3d convex-hull overlap processing and visualisation
    code.
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:d75613a5e2bf83ea9213c3f678aa533889927a1421037b217103dfb983ef2227
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/entropy_2dvs3d
  tools:
  - 2d_vs_3d
  - CellexalVR
  - Python
  - R
  - entropy_2dvs3d
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_4043
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# 3d-projection-angle-flattening-analysis

## Summary

Quantifies whether 3D single-cell dimension-reduction projections (UMAP/tSNE) separate annotated cell populations better than 2D by systematically rotating the 3D embedding, flattening each rotation to 2D, and scoring pairwise cell-type/cluster overlap with Shannon-entropy mixing or Quickhull convex-hull area.

## When to use

Use when comparing whether 3D single-cell dimension-reduction projections (UMAP/tSNE) separate annotated cell types or clusters better than 2D, quantified by pairwise Shannon-entropy mixing or convex-hull area overlap across swept viewing angles.

## When NOT to use

- Input is raw scRNA-seq expression counts rather than computed 2D/3D embedding coordinates with cell-type/cluster labels — dimension reduction must be performed first.
- The goal is interactive, immersive visualization or analysis of the 3D projection; use CellexalVR itself instead of this batch overlap-quantification analysis.
- You need per-cell differential-expression, heatmap marker, or pseudotime/slingshot trajectory analysis rather than population-overlap quantification.

## Inputs

- Precomputed 2D and 3D UMAP/tSNE cell-coordinate embeddings (e.g. from the mouse gastrulation scRNA-seq dataset, GEO GSE87038)
- Per-cell cell-type/cluster annotations used to define pairwise populations
- CellexalVR ConvexHullGenerator.cs output (for the convex-hull overlap route)
- Viewing-angle grid (2048 angles) for flattening the 3D embedding

## Outputs

- entropy_2d_vs_3d_results.csv: 2D versus 3D least-overlap Shannon entropy per cell type/cluster for both UMAP and tSNE
- hull_overlap_results.csv: table with method (2D/3D), angle, and overlap percentage columns
- Best viewing angle minimizing hull overlap from the exhaustive 2048-angle search
- Heatmaps and violin plots demonstrating the benefit of 3D over 2D

## How to apply

Obtain the authors' code — entropy_2dvs3d (Shannon-index route) with its companion 2d_vs_3d (convex-hull route) — and load the precomputed 2D and 3D UMAP/tSNE cell coordinates with cell-type/cluster annotations for the mouse gastrulation dataset (Pijuan-Sala et al., 2019, GEO GSE87038) into an R 4.0.0+ environment. Baseline the 2D projection: the entropy_2dvs3d rotating_grid notebook computes the Shannon Diversity index for cell populations considered two by two, while the 2d_vs_3d Python code builds Quickhull convex hulls per population and compares pairwise overlap area (processing CellexalVR's ConvexHullGenerator.cs output). Systematically rotate the 3D embedding, flatten each rotation to 2D, and recompute the overlap measure at each of 2048 viewing angles in an exhaustive search, retaining the minimum (least-overlap) value and the angle that produced it. Tabulate 2D versus best-angle 3D overlap per cell type/cluster for both UMAP and tSNE, writing entropy_2d_vs_3d_results.csv and hull_overlap_results.csv with method (2D/3D), angle, and overlap percentage columns. Validate that the least-overlap 3D entropy/hull percentage is lower than the 2D value for both UMAP and tSNE, reproducing the trend of Figures 2C and 2D, and optionally generate the accompanying heatmaps and violin plots.

## Related tools

- **entropy_2dvs3d** (Provides the rotating_grid notebook that computes the Shannon Diversity index of pairwise population overlap for 2D graphs and for 3D graphs rotated and flattened across viewing angles.) — https://github.com/sonejilab/entropy_2dvs3d
- **2d_vs_3d** (Python code that processes and visualises the convex-hull overlap output from CellexalVR's ConvexHullGenerator.cs, sweeping viewing angles and writing hull_overlap_results.csv.) — https://github.com/sonejilab/2d_vs_3d
- **CellexalVR** (Source platform whose ConvexHullGenerator.cs (ConvexHullOverlap branch) computes population convex hulls whose overlap the 2d_vs_3d code analyses.) — https://github.com/sonejilab/cellexalvr
- **R** (Runtime (R 4.0.0+) in which the entropy_2dvs3d analysis is set up and executed.)
- **Python** (Language of the 2d_vs_3d convex-hull overlap processing and visualisation code.)

## Examples

```
jupyter notebook rotating_grid.ipynb
```

## Evaluation signals

- The least-overlap 3D Shannon entropy must be lower than the 2D entropy for both UMAP and tSNE, reproducing the trend of Figures 2C and 2D.
- 3D convex-hull overlap percentages at the best of the 2048 angles must be lower than the corresponding 2D percentages.
- entropy_2d_vs_3d_results.csv must contain one entry per cell type/cluster per embedding (UMAP and tSNE) with 2D and 3D least-overlap entropy values; hull_overlap_results.csv must carry method (2D/3D), angle, and overlap percentage columns.
- The angle search must be exhaustive over all 2048 viewing angles, with the retained minimum tied to a recorded angle.
- The lower-overlap-in-3D trend should hold across both overlap measures (Shannon entropy and convex hulls), as the paper confirmed on two further large datasets.

## Limitations

- Overlap is minimized over a discrete grid of 2048 viewing angles, so the true optimum angle is not guaranteed to be sampled.
- Results depend on the quality and choice of the precomputed dimension-reduction embeddings and on the granularity of cell-type/cluster annotations; the trend was demonstrated on the mouse gastrulation dataset plus two further large datasets.
- Overlap calculations scale with the number of cells and population groups selected, so large datasets take longer to complete.

## Evidence

- [results] 3D reduction gives less pairwise overlap than 2D for both UMAP and tSNE (Figs 2C/2D): "for cell-types and clusters (Figures 2C and 2D respectively) reduction to three-dimensions resulted in less overlap than in two-dimensions."
- [results] Trend confirmed on two further large datasets with convex hulls as a second overlap measure: "We saw this trend consistently when applying this procedure to two further large datasets where we also used convex hulls as a second measure of overlap"
- [results] Motivating observation: two mesoderm subtypes overlap in 2D UMAP but separate in 3D: "Two mesoderm types during mouse organogenesis overlap in a UMAP when projected onto two dimensions, where, in (B) they are transcriptionally distinct when projected onto three"
- [abstract] Rationale: 3D projections are less prone to merging similar cell types/clusters: "Three-dimensional projections can be more informative for larger and complex datasets because they are less prone to merging and flattening similar cell-types/clusters together."
- [readme] Shannon index computation plus heatmaps and violin plots for 2D vs 3D graphs: [evidence span withheld — anchor status: external_doc]
- [readme] rotating_grid notebook contains the plotting functions and the overlap calculations: [evidence span withheld — anchor status: external_doc]
- [readme] Convex-hull area-based approach for pairwise population overlap in 3D vs 2D: [evidence span withheld — anchor status: external_doc]
- [discussion] Code availability for the entropy 2D-vs-3D method: "Code for this method can be obtained at https://github.com/sonejilab/entropy_2dvs3d"
