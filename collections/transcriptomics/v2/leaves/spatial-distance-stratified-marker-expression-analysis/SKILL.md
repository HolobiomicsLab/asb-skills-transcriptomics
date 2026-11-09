---
name: spatial-distance-stratified-marker-expression-analysis
description: Use when you have a segmented cell-by-gene matrix from subcellular spatial
  data (e.g. Xenium) plus cell-type labels, and you need to test whether cell boundaries
  leak transcripts between adjacent cells.
when_to_use_negative:
- Input is raw Xenium transcripts or DAPI images rather than a segmentation-derived
  cell-by-gene matrix — run the BIDCell pipeline (preprocess/train/predict/make_cell_gene_mat)
  first.
- The tissue lacks adjacent, spatially intermingled cell types (e.g. no T cells near
  B cells), so distance-to-nearest-neighbour stratification is uninformative.
- The goal is nuclear-shape or boundary-quality assessment (e.g. CellSPA morphological
  categories) rather than transcript-level contamination between neighbours.
tools:
- name: BIDCell
  role: Provides the segmented cell-gene expression matrix (expr_mat.csv) whose neighbour-contamination
    properties are evaluated; marker files sc_breast_markers_pos.csv and sc_breast_markers_neg.csv
    are supplied with the package
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Source platform for the Xenium-BreastCancer1 dataset (Xenium Output Bundle
    In Situ Replicate 1) on which the distance-stratified analysis is run
- name: pandas
  role: Used to load expr_mat.csv and the single-cell reference/marker CSVs for the
    contamination computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:a807d6bccff2eb2df45a1ad73138e5ecbbcd2c06b8398077ca89b77a1e09c223
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spatial-distance-stratified-marker-expression-analysis

## Summary

Quantifies neighbour contamination in a segmentation of subcellular spatial transcriptomics data (e.g. BIDCell output on Xenium-BreastCancer1) by stratifying cells of one type (B cells) by distance to the nearest cell of another type (T cells) and computing the percentage expressing cross-lineage markers (CD4, CD8A, CD8B, CD3D, CD3E), mirroring Fig. 3e of the BIDCell paper.

## When to use

Use when you have a segmented cell-by-gene matrix from subcellular spatial data (e.g. Xenium) plus cell-type labels, and you need to test whether cell boundaries leak transcripts between adjacent cells.

## When NOT to use

- Input is raw Xenium transcripts or DAPI images rather than a segmentation-derived cell-by-gene matrix — run the BIDCell pipeline (preprocess/train/predict/make_cell_gene_mat) first.
- The tissue lacks adjacent, spatially intermingled cell types (e.g. no T cells near B cells), so distance-to-nearest-neighbour stratification is uninformative.
- The goal is nuclear-shape or boundary-quality assessment (e.g. CellSPA morphological categories) rather than transcript-level contamination between neighbours.

## Inputs

- BIDCell cell-gene expression matrix (expr_mat.csv)
- Breast cancer single-cell reference (sc_breast.csv, TISCH-BRCA / Chromium scFFPE-seq)
- Positive marker file (sc_breast_markers_pos.csv)
- Negative marker file (sc_breast_markers_neg.csv)
- Cell type labels for segmented cells (e.g. B cells, T cells)
- Spatial centroid coordinates of segmented cells

## Outputs

- Distance-stratified percentage of B cells expressing CD4, CD8A, CD8B, CD3D, CD3E per distance bin
- fig3_evaluation_metrics.csv containing correlation, purity F1 scores, and distance-stratified percentages

## How to apply

Load the segmentation-derived cell-gene expression matrix (expr_mat.csv produced by BIDCell's make_cell_gene_mat) with pandas, together with the breast cancer single-cell reference (sc_breast.csv from the TISCH-BRCA / Chromium scFFPE-seq pairing) and the positive/negative marker files (sc_breast_markers_pos.csv, sc_breast_markers_neg.csv). Identify B cells and T cells in the segmentation, compute each B cell's distance to the nearest T cell, and bin B cells by that distance. In each distance bin, compute the percentage of B cells expressing the T-cell markers CD4, CD8A, CD8B, CD3D, and CD3E; a distance-dependent decline in negative-marker expression indicates the segmentation reduces neighbour contamination. Write the distance-stratified percentages (alongside correlation and purity F1 metrics) to fig3_evaluation_metrics.csv and check them against the reported Fig. 3e relationship. The rationale is that a densely populated region such as Xenium-BreastCancer1 stress-tests whether transcripts are misassigned across cell boundaries.

## Related tools

- **BIDCell** (Provides the segmented cell-gene expression matrix (expr_mat.csv) whose neighbour-contamination properties are evaluated; marker files sc_breast_markers_pos.csv and sc_breast_markers_neg.csv are supplied with the package) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Source platform for the Xenium-BreastCancer1 dataset (Xenium Output Bundle In Situ Replicate 1) on which the distance-stratified analysis is run)
- **pandas** (Used to load expr_mat.csv and the single-cell reference/marker CSVs for the contamination computation)

## Examples

```
import pandas as pd; expr = pd.read_csv('expr_mat.csv', index_col=0); markers_neg = pd.read_csv('sc_breast_markers_neg.csv')
```

## Evaluation signals

- Percentage of B cells expressing negative T-cell markers (CD4, CD8A, CD8B, CD3D, CD3E) decreases monotonically with increasing distance to the nearest T cell, reproducing the distance-dependent pattern of Fig. 3e.
- BIDCell segmentation yields the smallest percentage of contaminated B cells (B cells expressing negative T-cell markers) compared with other segmentation methods on Xenium-BreastCancer1.
- Distance-stratified percentages are present in fig3_evaluation_metrics.csv alongside correlation and positive/negative purity F1 values consistent with Fig. 3a-d categories.
- Marker gene sets match the provided files (top/bottom 10 percentile per cell type, positive markers common to at least a third of cell types removed).

## Limitations

- Evaluation depends on the quality of the single-cell reference and marker definitions; markers were defined as genes in the highest/lowest 10 percentile per cell type, and manual curation or alternative marker-selection approaches may be needed.
- The contamination metric reflects transcript misassignment between neighbouring cells in a specific densely populated region (Xenium-BreastCancer1) and may not generalise to sparse tissues or other platforms (CosMx, MERSCOPE, Stereo-seq).
- Two-dimensional segmentation may achieve limited representation of cells' three-dimensional structure, affecting measured distances and apparent contamination.

## Evidence

- [methods] marker files supplied with package: "Please see the provided breast cancer single-cell reference and positive/negative marker files (`sc_breast.csv`, `sc_breast_markers_pos.csv`, and `sc_breast_markers_neg.csv`)"
- [readme] marker percentile definition: "The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample."
- [results] contamination finding: [evidence span withheld — anchor status: not_found]
- [readme] readme on marker curation: "We found that removing positive markers that were common to at least a third of cell types in each dataset was appropriate across various datasets."
- [discussion] 2D limitation: "Cells have a three-dimensional structure, thus analyses in a two-dimensional perspective may achieve limited representation."
