---
name: tsne-dimensionality-reduction
description: Use when you have cells projected onto a small selected marker-gene panel
  (e.g. scGeneFit's 15 CBMC markers) plus cell type labels, and need a 2-D visualization
  to check that cell populations remain separated.
when_to_use_negative:
- The task is to select the marker genes themselves rather than visualize an existing
  panel — use scGeneFit marker selection, not t-SNE plotting.
- The cells lack cell type labels, so the scatter plot cannot be colored or evaluated
  for separation of the intended populations.
- A quantitative clustering or label-recovery metric is required — t-SNE 2-D distances
  are qualitative and should not be used as the sole accuracy measure.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scGeneFit
  role: Provides the 15-marker gene panel that defines the low-dimensional projection
    space of the CBMC cells prior to t-SNE embedding
  repo: github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:c79e8085ce686b83f20f7f96e31a69a322b9a63a941df69260ea916ad059a651
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGeneFit
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# tsne-dimensionality-reduction

## Summary

Visualize single-cell RNA-seq cells in a reduced scGeneFit-selected marker space by running t-SNE on the 15-marker CBMC projection and rendering a scatter plot colored by cell type, to show that 13 cell populations remain distinguishable with few markers.

## When to use

Use when you have cells projected onto a small selected marker-gene panel (e.g. scGeneFit's 15 CBMC markers) plus cell type labels, and need a 2-D visualization to check that cell populations remain separated.

## When NOT to use

- The task is to select the marker genes themselves rather than visualize an existing panel — use scGeneFit marker selection, not t-SNE plotting.
- The cells lack cell type labels, so the scatter plot cannot be colored or evaluated for separation of the intended populations.
- A quantitative clustering or label-recovery metric is required — t-SNE 2-D distances are qualitative and should not be used as the sole accuracy measure.

## Inputs

- CBMC scRNA-seq expression matrix (cells x genes) with cell type labels for 13 populations
- 15-marker scGeneFit panel (list of selected marker genes) shipped in the scGeneFit release data files

## Outputs

- 2-D t-SNE embedding of cells in the 15-marker projection space
- Scatter plot colored by cell type label, saved as tsne_marker_projection.png (or .pdf)

## How to apply

Load the single-cell expression dataset and the 15-marker scGeneFit panel from the scGeneFit release data (github.com/solevillar/scGeneFit), then subset the full expression matrix to the 15 selected marker-gene columns to form the low-dimensional projection of each cell. Append the cellular label annotations so each projected cell retains its cell type identity through the embedding. Run t-SNE on the 15-dimensional marker projection to embed the cells in two dimensions. Render the 2-D embedding as a scatter plot with points colored by cell type label, reproducing the published figure for the 13 CBMC cell populations, and save it as tsne_marker_projection.png (or .pdf). The rationale is that scGeneFit's markers jointly optimize cell label recovery, so cluster separation in the t-SNE view is direct qualitative evidence the panel discriminates cell types.

## Related tools

- **scGeneFit** (Provides the 15-marker gene panel that defines the low-dimensional projection space of the CBMC cells prior to t-SNE embedding) — github.com/solevillar/scGeneFit

## Evaluation signals

- The output figure file tsne_marker_projection.png (or .pdf) exists and is a valid PNG/PDF.
- The scatter plot visually separates the 13 CBMC cell types into distinct clusters consistent with the published t-SNE visualization.
- The input to t-SNE has exactly 15 marker-gene columns matching the scGeneFit panel.
- Each point in the embedding retains its cell type label so cluster coloring is complete (no unlabeled cells).
- The embedding is 2-D, matching the published figure format.

## Limitations

- t-SNE is a post-hoc visualization; the underlying scGeneFit markers assume cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in lower dimensions.
- The 2-D embedding is qualitative — cluster separation in the plot does not by itself quantify marker-panel label-recovery performance.
- scGeneFit's pipeline did not implement nonlinear dimension reduction methods in marker selection; the authors planned to relax this assumption in future work.

## Evidence

- [abstract] scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods.: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [results] CBMC application: "we applied scGeneFit to a cord blood mononuclear cell (CBMC) study containing isolated cells from cord blood"
- [discussion] Linear separability assumption: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable"
- [discussion] Future nonlinear methods: "In future work, we plan to depart from this assumption by considering a more general class of nonlinear dimension reduction methods"
