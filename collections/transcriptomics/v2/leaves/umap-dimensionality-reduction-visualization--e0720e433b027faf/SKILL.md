---
name: umap-dimensionality-reduction-visualization
description: Use when annotating cells at a later timepoint (t1) against known labels
  from an initial timepoint (t0) and you need a 2D UMAP projection to inspect whether
  novel-cell calls or annotation assignments form coherent clusters.
when_to_use_negative:
- The analysis goal is temporal DEG detection (FindTemporalDEGs) rather than annotation/novel-cell
  visualization — UMAP plots are not part of that step's evidence
- Data comes from a single timepoint only; CASi's cross-timepoint UMAP workflow requires
  an initial timepoint (t0) plus at least one following timepoint (t1)
- You need quantitative recovery metrics (e.g., ARI comparisons against CHETAH, scPred,
  scmap) rather than a qualitative cluster visualization
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CASi
  role: Provides DetectNovelCells with DrawUMAP=TRUE to compute and render the correlation/label-colored
    UMAP
  repo: https://github.com/yizhuo-wang/CASi
- name: Seurat
  role: Underlying single-cell object framework for the CASi list and expression matrices
- name: umap
  role: UMAP embedding computation for the 2D projection
- name: ggplot2
  role: Figure rendering for the UMAP plots
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:454359b9a0a4a23262cfe4cb32ce1dba8a70eae4c5fa580619097129d1707722
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yizhuo-wang/CASi
  tools:
  - CASi
  - LinQ-View
  - R
  - Seurat
  - ggplot2
  - umap
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# umap-dimensionality-reduction-visualization

## Summary

Generate UMAP embeddings of scRNA-seq timepoint data and render them as plots colored by cell-type-specific correlation scores and cell labels, used in CASi's novel-cell detection to visually confirm that cells flagged as novel (correlation ≤ 0.4, p < 0.01) form a distinct cluster rather than being misassigned to known t0 labels.

## When to use

Use when annotating cells at a later timepoint (t1) against known labels from an initial timepoint (t0) and you need a 2D UMAP projection to inspect whether novel-cell calls or annotation assignments form coherent clusters.

## When NOT to use

- The analysis goal is temporal DEG detection (FindTemporalDEGs) rather than annotation/novel-cell visualization — UMAP plots are not part of that step's evidence
- Data comes from a single timepoint only; CASi's cross-timepoint UMAP workflow requires an initial timepoint (t0) plus at least one following timepoint (t1)
- You need quantitative recovery metrics (e.g., ARI comparisons against CHETAH, scPred, scmap) rather than a qualitative cluster visualization

## Inputs

- CASi list built by CreateCASiList from t0 and t1 gene expression matrices plus t0 cell labels (default nfeature=2000)
- DetectNovelCells output with corr.cutoff=0.4 and p.cutoff=0.01 flags per t1 cell

## Outputs

- novel_cells_umap.png — UMAP plot colored by cell-type-specific correlation and cell labels
- UMAP embedding coordinates of the t1 cells (2D projection used for the figure)

## How to apply

After running DetectNovelCells on a CASi list (built with CreateCASiList from the t0 and t1 expression matrices and t0 cell labels, nfeature=2000), set DrawUMAP=TRUE with corr.cutoff=0.4 and p.cutoff=0.01 so the function computes a UMAP embedding of the t1 cells and draws it colored by both the cell-type-specific correlation values and the assigned cell labels. The plot (saved as novel_cells_umap.png) serves as the visual validation that cells flagged novel by the correlation/p-value filters form a separate cluster (e.g., B cells in simulation scenario 3) instead of being absorbed into known t0 types. Rendering uses the umap package for the embedding and ggplot2 for the figures; Seurat objects underlie the CASi list structure.

## Related tools

- **CASi** (Provides DetectNovelCells with DrawUMAP=TRUE to compute and render the correlation/label-colored UMAP) — https://github.com/yizhuo-wang/CASi
- **Seurat** (Underlying single-cell object framework for the CASi list and expression matrices)
- **umap** (UMAP embedding computation for the 2D projection)
- **ggplot2** (Figure rendering for the UMAP plots)

## Examples

```
DrawUMAP=TRUE within DetectNovelCells: `R -e 'require(keras); require(CASi); cl <- CreateCASiList(t0_expr, t1_expr, labels_t0, nfeature=2000); cl <- DetectNovelCells(cl, corr.cutoff=0.4, p.cutoff=0.01, DrawUMAP=TRUE)'`
```

## Evaluation signals

- The analysis script runs without error and novel_cells_umap.png exists as a valid figure file
- The UMAP shows the novel-cell cluster recovered by the correlation/p-value flags rather than cells misassigned to known t0 labels
- In simulation scenario 3, flagged novel cells (correlation ≤ 0.4, t-test p < 0.01) visually coincide with the ground-truth novel B-cell population at t1 and t2
- Plots are colored by both cell-type-specific correlation and cell labels, matching the CASi DrawUMAP output specification

## Limitations

- Novel cell detection (and thus its UMAP confirmation) may not work well when the novel cell type is transcriptomically similar to known cell types
- If the initial timepoint's data is dramatically different from later timepoint data, the neural network cannot be well trained, undermining annotation and any visualization built on it
- The visualization is qualitative; quantitative comparison (ARI) against methods like CHETAH, scPred, scmap was only reported in supplementary Figure 11

## Evidence

- [results] UMAP correlation plots & fish plots: [evidence span withheld — anchor status: not_found]
- [methods] DrawUMAP parameter: [evidence span withheld — anchor status: not_found]
- [results] workflow outputs: [evidence span withheld — anchor status: not_found]
- [abstract] package scope: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data"
- [intro] dimension reduction context: "the downstream analysis of scRNA-seq data includes dimension reduction, unsupervised clustering, differential expression analysis and so on"
