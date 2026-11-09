---
name: scrna-seq-marker-projection
description: Use when you have scRNA-seq data with cellular labels and a selected
  marker-gene panel (e.g., scGeneFit's 15-marker CBMC panel) and need to verify whether
  the reduced marker space preserves cell type distinctions via t-SNE.
when_to_use_negative:
- The goal is to select the marker panel itself rather than visualize an existing
  one — use scGeneFit's marker selection instead of a projection task
- The cell type classes are not linearly separable in the marker space or the analysis
  requires nonlinear dimension reduction — the article notes scGeneFit assumes linear
  separability and had not yet implemented nonlinear methods
- Labels are continuous manifold quantities (e.g., trajectory pseudotime or spatial
  location) rather than categorical cell type labels — the article states categorical
  labeling is a current limitation
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGeneFit
  role: Provides the selected marker gene panel that defines the reduced marker space
    onto which cells are projected for t-SNE visualization
  repo: github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:2439f5df2d8ec25d82369c688f99b72a8ba44fc8c5642dbad9fad363f272d83e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGeneFit
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# scrna-seq-marker-projection

## Summary

Visualizes single-cell RNA-seq data in a low-dimensional marker-gene space by subsetting the full expression matrix to an scGeneFit-selected marker panel (e.g., 15 markers for the 13-population CBMC dataset) and embedding the cells with t-SNE to check that cell type distinctions are preserved.

## When to use

Use when you have scRNA-seq data with cellular labels and a selected marker-gene panel (e.g., scGeneFit's 15-marker CBMC panel) and need to verify whether the reduced marker space preserves cell type distinctions via t-SNE.

## When NOT to use

- The goal is to select the marker panel itself rather than visualize an existing one — use scGeneFit's marker selection instead of a projection task
- The cell type classes are not linearly separable in the marker space or the analysis requires nonlinear dimension reduction — the article notes scGeneFit assumes linear separability and had not yet implemented nonlinear methods
- Labels are continuous manifold quantities (e.g., trajectory pseudotime or spatial location) rather than categorical cell type labels — the article states categorical labeling is a current limitation

## Inputs

- Single-cell scRNA-seq expression matrix (cells x genes) for the CBMC cord blood mononuclear cell dataset
- scGeneFit-selected marker gene panel (15 marker genes) shipped in the scGeneFit release data files (github.com/solevillar/scGeneFit)
- Cellular label annotations assigning each cell to one of the 13 cell type populations

## Outputs

- Two-dimensional t-SNE embedding of all cells computed on the 15-marker reduced expression matrix
- Scatter plot of the embedding colored by cell type label, saved as tsne_marker_projection.png (or .pdf)

## How to apply

Load the single-cell expression dataset and the marker panel shipped with the scGeneFit release data files (github.com/solevillar/scGeneFit). Subset the full expression matrix to the selected marker-gene columns (e.g., 15 scGeneFit markers for the CBMC cord blood mononuclear cell dataset with 13 cell type populations), forming a low-dimensional projection of each cell. Append the cellular label annotations so each projected cell retains its cell type identity. Run t-SNE on the reduced marker matrix to embed cells in two dimensions, then render a scatter plot colored by cell type label and save it (e.g., tsne_marker_projection.png). Rationale: scGeneFit selects markers that jointly optimize cell label recovery, so the panel is expected to retain the separations between the 13 CBMC populations even in this compressed space.

## Related tools

- **scGeneFit** (Provides the selected marker gene panel that defines the reduced marker space onto which cells are projected for t-SNE visualization) — github.com/solevillar/scGeneFit

## Examples

```
python run_tsne_marker_projection.py --input cbmc_expr_matrix --markers scgenefit_15_markers --labels cell_types --output tsne_marker_projection.png
```

## Evaluation signals

- The output figure file tsne_marker_projection.png (or .pdf) exists and is a valid PNG/PDF
- The scatter plot visually separates the 13 CBMC cell populations into distinct clusters
- The clustering pattern in the reduced marker space is consistent with the published t-SNE visualization of cells projected onto the 15-marker scGeneFit panel
- The expression matrix passed to t-SNE has exactly 15 columns (the selected markers), matching the panel size

## Limitations

- The projection assumes cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in the reduced marker space
- The underlying marker selection supports only categorical labeling; no support for placing unlabeled cells on trajectories or in spatial assays
- scGeneFit had not implemented nonlinear dimension reduction methods; the article's discussion flags this as planned future work

## Evidence

- [abstract] Marker selection optimizes joint label recovery: "Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification"
- [results] CBMC dataset with reduced marker space: "we applied scGeneFit to a cord blood mononuclear cell (CBMC) study containing isolated cells from cord blood"
- [discussion] Linear separability limitation: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
- [discussion] Categorical labeling limitation: "we envision relaxing the categorical labeling to a manifold constraint that will allow selection of markers to place unlabeled cells at, for example, time points along cell trajectories or locations"
