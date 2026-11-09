---
name: quickhull-computational-geometry
description: Use when single-cell 2D/3D dimension-reduction embeddings (UMAP/tSNE)
  with cell-type or cluster annotations are available and you need an area-based,
  quantitative measure of population overlap between cell types or clusters.
when_to_use_negative:
- Input is raw single-cell expression data without precomputed dimension-reduction
  coordinates — run UMAP/tSNE first
- The goal is a Shannon-index (entropy) overlap measure rather than area-based convex
  hull overlap — use the entropy_2dvs3d code instead
- Cell populations are highly non-convex or multipolar, where a single convex hull
  per population poorly represents its footprint
tools:
- name: CellexalVR
  role: Generates the convex hulls of cell populations in 2D and 3D graphs via ConvexHullGenerator.cs;
    the 2d_vs_3d Python code processes and visualises its output
  repo: https://github.com/sonejilab/cellexalvr
- name: Python
  role: Processes and visualises CellexalVR convex hull output and computes pairwise
    hull overlap areas for 2D and 3D projections
  repo: https://github.com/sonejilab/2d_vs_3d
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:600a53801021da0f4f784a9f4d04a9ef06ceeda97c8d02f8cde1a06b6da7e7c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvr
  tools:
  - CellexalVR
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# quickhull-computational-geometry

## Summary

Quantifies pairwise overlap between cell-type/cluster populations in dimension-reduction plots by computing Quickhull convex hulls on 2D and 3D UMAP/tSNE projections and comparing hull-overlap areas, as used in the CellexalVR 2D-vs-3D analysis.

## When to use

Use when single-cell 2D/3D dimension-reduction embeddings (UMAP/tSNE) with cell-type or cluster annotations are available and you need an area-based, quantitative measure of population overlap between cell types or clusters.

## When NOT to use

- Input is raw single-cell expression data without precomputed dimension-reduction coordinates — run UMAP/tSNE first
- The goal is a Shannon-index (entropy) overlap measure rather than area-based convex hull overlap — use the entropy_2dvs3d code instead
- Cell populations are highly non-convex or multipolar, where a single convex hull per population poorly represents its footprint

## Inputs

- 2D UMAP or tSNE embedding coordinates per cell
- 3D UMAP or tSNE embedding coordinates per cell
- Per-cell cell-type or cluster annotations (e.g. mouse gastrulation dataset, GEO: GSE87038)
- List of 2048 viewing angles for 3D projection flattening

## Outputs

- hull_overlap_results.csv with columns for method (2D/3D), viewing angle, and overlap percentage
- Pairwise cell-type/cluster convex hull overlap percentage matrices (2D and best-angle 3D)

## How to apply

Load the single-cell dimension-reduction coordinates (2D and 3D UMAP/tSNE embeddings) with cell-type/cluster annotations, e.g. for the mouse gastrulation dataset (Pijuan-Sala et al., 2019). Compute per-population convex hulls on the 2D projection with the Quickhull algorithm and quantify pairwise hull overlap between cell types as a percentage of hull area. For 3D, flatten the projection over 2048 viewing angles, recomputing Quickhull convex hulls and pairwise overlap at each angle, then exhaustively search the angles for the one minimizing overlap. Compare 2D overlap percentages against best-angle 3D percentages, writing method (2D/3D), angle, and overlap percentage columns to hull_overlap_results.csv. Validation: 3D overlap must be lower than 2D, consistent with the finding that reduction to three dimensions resulted in less cell-type overlap for both UMAP and tSNE.

## Related tools

- **CellexalVR** (Generates the convex hulls of cell populations in 2D and 3D graphs via ConvexHullGenerator.cs; the 2d_vs_3d Python code processes and visualises its output) — https://github.com/sonejilab/cellexalvr
- **Python** (Processes and visualises CellexalVR convex hull output and computes pairwise hull overlap areas for 2D and 3D projections) — https://github.com/sonejilab/2d_vs_3d

## Examples

```
from scipy.spatial import ConvexHull; h = ConvexHull(points_2d)  # Quickhull; repeat per population and compare pairwise overlap areas
```

## Evaluation signals

- Reproduced overlap percentages must show lower overlap in three dimensions than in two dimensions, for both UMAP and tSNE
- hull_overlap_results.csv contains the required columns: method (2D/3D), angle, and overlap percentage
- Each pairwise overlap percentage lies within [0, 100]
- The reported 3D percentage corresponds to the minimum over the exhaustively searched 2048 viewing angles
- Lower-overlap-in-3D trend should be reproducible on two further large datasets when convex hulls are used as the second overlap measure

## Limitations

- 3D overlap depends on viewing angle; only 2048 sampled angles are evaluated, so the true minimum may be missed
- Convex hulls assume convex population footprints, which can overstate overlap for concave, banana-shaped, or nested cell populations
- The article validated the lower-overlap-in-3D trend on the mouse gastrulation dataset plus two further large datasets; results are not guaranteed for smaller or sparsely annotated datasets

## Evidence

- [readme] Convex hull method choice and area-based overlap: [evidence span withheld — anchor status: external_doc]
- [readme] Python code links to CellexalVR ConvexHullGenerator.cs: [evidence span withheld — anchor status: not_found]
- [methods] 2048-angle flattening and Quickhull recompute: [evidence span withheld — anchor status: not_found]
- [results] 3D less overlap than 2D for UMAP and tSNE: "for cell-types and clusters (Figures 2C and 2D respectively) reduction to three-dimensions resulted in less overlap than in two-dimensions."
- [results] Convex hulls as second measure confirmed on two further datasets: "We saw this trend consistently when applying this procedure to two further large datasets where we also used convex hulls as a second measure of overlap"
