---
name: cell-population-evolution-visualization
description: Use when analyzing multi-timepoint scRNA-seq (t0/t1 expression matrices
  with t0 labels) and you need to display how annotated and novel cell populations
  evolve, e.g., emerging tumor subclones.
when_to_use_negative:
- Data come from a single timepoint with no t0/t1 structure to compare, so cross-timepoint
  population evolution is undefined
- The follow-up timepoint data differ dramatically from the initial timepoint, so
  the t0-trained neural network cannot borrow useful information for annotation
- The goal is temporal differential expression (tDEGs) rather than cell population
  evolution, which is handled by FindTemporalDEGs instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CASi
  role: Provides CreateCASiList, CrossTimeAnnotation (DrawUMAP=TRUE), DetectNovelCells,
    and fish-plot visualization of cell population evolution
  repo: github.com/yizhuo-wang/CASi
- name: Seurat
  role: Supports UMAP visualization of annotated and novel cell populations
- name: umap
  role: Computes UMAP embeddings for correlation- and label-colored plots
- name: ggplot2
  role: Renders the UMAP and fish plot figures (e.g., novel_cells_umap.png)
- name: keras
  role: Backend for the CrossTimeAnnotation neural network whose outputs feed the
    visualizations
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:10373d8af01961ff2bcc3d8e72ce2618e6356ff11d8787896b1cf5e3f401ae2e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMLU
  - CASi
  - CHETAH
  - Seurat
  - ggplot2
  - keras
  - scmap
  - singleR
  - umap
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# cell-population-evolution-visualization

## Summary

Visualize how cell populations change across timepoints in multi-timepoint scRNA-seq using CASi's UMAP plots (colored by cell-type-specific correlation and cell labels, via DrawUMAP=TRUE) and fish plots of annotated vs. novel population evolution.

## When to use

Use when analyzing multi-timepoint scRNA-seq (t0/t1 expression matrices with t0 labels) and you need to display how annotated and novel cell populations evolve, e.g., emerging tumor subclones.

## When NOT to use

- Data come from a single timepoint with no t0/t1 structure to compare, so cross-timepoint population evolution is undefined
- The follow-up timepoint data differ dramatically from the initial timepoint, so the t0-trained neural network cannot borrow useful information for annotation
- The goal is temporal differential expression (tDEGs) rather than cell population evolution, which is handled by FindTemporalDEGs instead

## Inputs

- t0 (initial timepoint) gene expression count matrix
- t1 (follow-up timepoint) gene expression count matrix
- t0 cell type labels
- Optional sample metadata, e.g., Group (pre/post treatment) and Time for the 21-specimen mantle cell lymphoma dataset

## Outputs

- UMAP plots colored by cell-type-specific correlation and cell labels (e.g., novel_cells_umap.png)
- Fish plot (cell population evolution) summarizing annotated and novel populations over timepoints
- Correlation-defined tumor-cell clusters (e.g., exactly two clusters in the MCL application)

## How to apply

Build a CASi list with CreateCASiList passing the t0 expression matrix, t1 expression matrix, and t0 cell labels (default nfeature=2000 top highly variable genes). Run CrossTimeAnnotation with DrawUMAP=TRUE to train the Keras/TensorFlow neural network classifier on labeled t0 cells and generate UMAPs colored by cell-type-specific correlation and cell labels. Run DetectNovelCells with corr.cutoff=0.4 and p.cutoff=0.01 so cells with correlation <= 0.4 to known types and t-test p < 0.01 are flagged as novel and appear as distinct clusters in the UMAP. Finally, generate the cell population evolution (fish plot) summarizing how annotated and novel populations change over time.

## Related tools

- **CASi** (Provides CreateCASiList, CrossTimeAnnotation (DrawUMAP=TRUE), DetectNovelCells, and fish-plot visualization of cell population evolution) — github.com/yizhuo-wang/CASi
- **Seurat** (Supports UMAP visualization of annotated and novel cell populations)
- **umap** (Computes UMAP embeddings for correlation- and label-colored plots)
- **ggplot2** (Renders the UMAP and fish plot figures (e.g., novel_cells_umap.png))
- **keras** (Backend for the CrossTimeAnnotation neural network whose outputs feed the visualizations)

## Examples

```
require(keras); require(CASi); casilist <- CreateCASiList(t0_count, t1_count, t0_label); CrossTimeAnnotation(casilist, DrawUMAP=TRUE); DetectNovelCells(casilist, corr.cutoff=0.4, p.cutoff=0.01)
```

## Evaluation signals

- The analysis script runs without error and novel_cells_umap.png exists as a valid figure file
- The UMAP shows the novel-cell cluster recovered by correlation/p-value flags (corr <= 0.4, p < 0.01) rather than cells misassigned to known t0 labels
- In the MCL application, tumor cells separate into exactly two correlation-defined clusters in the UMAP as reported
- The fish plot reflects the expected population evolution, e.g., emergent tumor subclones growing across pre/post-treatment specimens

## Limitations

- Novel cell detection may fail when the novel cells are transcriptomically similar to the known cell types
- The current cell type annotation step pools data from all samples together for training and applying the neural network, which may be inappropriate when cell populations differ across conditions
- If the initial timepoint's data is dramatically different from later timepoints, the neural network cannot borrow useful information and the downstream visualizations inherit that failure

## Evidence

- [abstract] CASi capabilities including visualization: "cross-timepoint cell annotation, (2) detection of potentially novel cell types emerged over time, (3) visualization of cell population evolution"
- [readme] README description of CASi package scope: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data, which provides users with: (1) cross-time points cell annotation"
- [intro] Neural network classifier underlying annotated populations: "Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting."
- [methods] Demo data lazy-loaded with package: "The demo pbmc data, "pbmc_demo", has been lazy-loaded with the package."
