---
name: cell-type-scatter-plot-visualization
description: Use when a small marker-gene panel has been selected and you need a visual
  check that the reduced-dimensional projection separates known cell type labels into
  distinct clusters, e.g., reproducing a published t-SNE of cells projected onto 15
  markers.
when_to_use_negative:
- Marker selection has not yet been performed and no reduced marker set is available
  — run scGeneFit marker selection first
- The goal is quantitative label-recovery accuracy rather than a visual separability
  check
- Cell labels are continuous (trajectories, spatial locations) rather than categorical
  — the paper notes categorical labeling is a current limitation
- Input is already a 2-D embedding; this skill is about building the embedding from
  a marker-gene projection
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGeneFit
  role: Source of the selected marker-gene panel whose reduced space is projected
    and visualized; the marker panel is loaded from the scGeneFit release data files
  repo: github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:d812c1ac12142420c989c05f7275057d14eb15896497459b7f4eb55b34cf3b10
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGeneFit
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cell-type-scatter-plot-visualization

## Summary

Visualize single cells in a low-dimensional marker space by embedding the reduced marker-gene projection with t-SNE and rendering a 2-D scatter plot colored by cell type label, to verify that the selected marker panel (e.g., scGeneFit's 15 markers on the CBMC dataset) preserves distinctions between cell populations.

## When to use

Use when a small marker-gene panel has been selected and you need a visual check that the reduced-dimensional projection separates known cell type labels into distinct clusters, e.g., reproducing a published t-SNE of cells projected onto 15 markers.

## When NOT to use

- Marker selection has not yet been performed and no reduced marker set is available — run scGeneFit marker selection first
- The goal is quantitative label-recovery accuracy rather than a visual separability check
- Cell labels are continuous (trajectories, spatial locations) rather than categorical — the paper notes categorical labeling is a current limitation
- Input is already a 2-D embedding; this skill is about building the embedding from a marker-gene projection

## Inputs

- Single-cell RNA-seq expression matrix (e.g., CBMC cord blood mononuclear cell dataset) with cells x genes
- Selected marker-gene panel (e.g., scGeneFit's 15 selected marker genes from the release data files at github.com/solevillar/scGeneFit)
- Per-cell cell type label annotations (e.g., 13 CBMC cell populations)

## Outputs

- 2-D t-SNE embedding of cells in the reduced marker space
- Scatter plot with points colored by cell type label, saved as tsne_marker_projection.png (or .pdf)

## How to apply

Load the single-cell expression dataset and the marker panel (e.g., the 15-marker scGeneFit panel shipped in the scGeneFit release data files), then subset the full expression matrix to the selected marker-gene columns to form the low-dimensional projection of each cell. Append the cellular label annotations so each projected cell retains its cell type identity. Run t-SNE on the 15-dimensional marker projection to embed the cells in two dimensions, and render the 2-D embedding as a scatter plot with points colored by cell type label. Save the figure as tsne_marker_projection.png (or .pdf). Rationale: scGeneFit's markers jointly optimize cell label recovery, so the t-SNE in marker space should visually reproduce the separation of the 13 CBMC cell populations seen with the full label set.

## Related tools

- **scGeneFit** (Source of the selected marker-gene panel whose reduced space is projected and visualized; the marker panel is loaded from the scGeneFit release data files) — github.com/solevillar/scGeneFit

## Examples

```
X_markers = X[:, marker_idx]; emb = TSNE(n_components=2).fit_transform(X_markers); scatter(emb[:,0], emb[:,1], c=labels); savefig('tsne_marker_projection.png')
```

## Evaluation signals

- Output figure file exists and is a valid PNG/PDF (tsne_marker_projection.png or .pdf)
- Scatter plot visually separates the cell types into distinct clusters consistent with the published t-SNE visualization
- All 13 CBMC cell type populations are represented by distinguishable clusters/labels in the plot
- The embedding was computed from exactly the 15 selected marker-gene columns, not the full expression matrix
- Each plotted cell retains its original cell type label identity (labels correctly joined to projected cells)

## Limitations

- Visual cluster separation in t-SNE is qualitative; the underlying marker selection assumes cell type classes are linearly separable, which may not hold in lower dimensions
- The paper notes nonlinear dimension reduction methods were not implemented in scGeneFit itself; t-SNE here is a downstream visualization, not part of marker optimization
- Categorical labeling only — the method does not support placing unlabeled cells on trajectories or in spatial assays
- t-SNE embeddings are stochastic; peripherally separated points may reflect t-SNE parameters rather than marker-panel quality

## Evidence

- [results] 15-marker CBMC t-SNE: "we applied scGeneFit to a cord blood mononuclear cell (CBMC) study containing isolated cells from cord blood"
- [abstract] joint marker optimization rationale: "Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery"
- [discussion] linear separability assumption: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable"
- [discussion] categorical labeling limitation: "we envision relaxing the categorical labeling to a manifold constraint that will allow selection of markers to place unlabeled cells at, for example, time points"
- [abstract] one-vs-all redundancy contrast: "This results in a substantially more robust and less redundant set of markers than existing methods, most of which identify markers that separate each cell label from the rest."
