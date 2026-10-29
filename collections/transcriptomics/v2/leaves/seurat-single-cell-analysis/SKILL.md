---
name: seurat-single-cell-analysis
description: Use when raw single-cell RNA-seq counts (e.g., 10x PBMC, bone marrow,
  MERFISH spatial) must be normalized, clustered, and embedded in Seurat before cell-type
  annotation with accordion() or accordion_disease().
when_to_use_negative:
- Input is already an annotated feature table or per-cell score matrix — Seurat clustering
  would not add information
- The goal is CNV-based malignant cell calling (use SCEVAN or CopyKAT instead, which
  operate on their own pipelines)
- Data is bulk RNA-seq or non-single-cell; the Accordion/Seurat workflow targets single-cell
  and spatial omics
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Preprocessing, clustering, UMAP embedding, and FeaturePlot/DimPlot visualization
    of Seurat objects that serve as input to the Cell Marker Accordion functions
- name: cellmarkeraccordion
  role: Annotates the Seurat-preprocessed object via accordion()/accordion_disease(),
    returning per-cluster annotations or per-cell disease scores
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: ggplot2
  role: Used for grouped score distribution visualization in R
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:c09eb1fb942828ef2bc4b5d03a9a382ab35fba6a988f09b0c08032060a162b0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - CopyKAT
  - R
  - SCEVAN
  - Seurat
  - cellmarkeraccordion
  - ggplot2
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# seurat-single-cell-analysis

## Summary

Standard Seurat preprocessing, clustering, dimensionality reduction, and visualization pipeline used to prepare scRNA-seq and spatial data (Seurat objects or count matrices) for Cell Marker Accordion annotation and downstream per-cell disease scoring.

## When to use

Use when raw single-cell RNA-seq counts (e.g., 10x PBMC, bone marrow, MERFISH spatial) must be normalized, clustered, and embedded in Seurat before cell-type annotation with accordion() or accordion_disease().

## When NOT to use

- Input is already an annotated feature table or per-cell score matrix — Seurat clustering would not add information
- The goal is CNV-based malignant cell calling (use SCEVAN or CopyKAT instead, which operate on their own pipelines)
- Data is bulk RNA-seq or non-single-cell; the Accordion/Seurat workflow targets single-cell and spatial omics

## Inputs

- Raw single-cell RNA-seq count matrix (genes x cells, e.g., counts.rda from 10x PBMC) or preprocessed Seurat object
- MERFISH imaging-based spatial transcriptomics dataset
- Cluster ID per cell (data.table of cell and cluster columns) when running accordion() on a raw counts matrix
- Condition metadata (e.g., healthy vs AML) for split FeaturePlot visualization

## Outputs

- Processed Seurat object with PCA/UMAP embeddings, seurat_clusters, and accordion annotations in metadata
- Per-cell disease scores in object metadata (e.g., LHSC_per_cell_score, Neoplastic_monocyte_per_cell_score)
- UMAP/FeaturePlot PNG visualizations with quantile cutoffs (q10-q95)
- Exported score tables such as aml_disease_scores.csv

## How to apply

Create a Seurat object with CreateSeuratObject(counts, min.cells = 3, min.features = 200), then run NormalizeData, FindVariableFeatures (method 'vst', nfeatures = 2000), ScaleData, RunPCA, FindNeighbors (dims = 1:10), FindClusters (resolution = 0.4) and RunUMAP (dims = 1:10). Filter low-quality cells before or during QC, removing cells with fewer than 200 or more than 6000 expressed genes and those with 20% or more mitochondrial-derived counts, to eliminate empty droplets and doublets. The resulting Seurat object (or raw/normalized matrix plus cluster IDs via cluster_info) is the direct input to accordion() and accordion_disease(), which return per-cluster annotations (accordion_per_cluster) or per-cell scores (e.g., LHSC_per_cell_score, Neoplastic_monocyte_per_cell_score) added to the object metadata. Visualize scores with FeaturePlot split by condition using quantile cutoffs such as min.cutoff='q15'/max.cutoff='q95'. The Rationale: the Accordion functions accept Seurat objects directly, so the Seurat pipeline defines the clustering resolution (cell vs cluster) that determines annotation granularity.

## Related tools

- **Seurat** (Preprocessing, clustering, UMAP embedding, and FeaturePlot/DimPlot visualization of Seurat objects that serve as input to the Cell Marker Accordion functions)
- **cellmarkeraccordion** (Annotates the Seurat-preprocessed object via accordion()/accordion_disease(), returning per-cluster annotations or per-cell disease scores) — https://github.com/TebaldiLab/cellmarkeraccordion
- **ggplot2** (Used for grouped score distribution visualization in R)

## Examples

```
data <- CreateSeuratObject(counts = counts, min.cells = 3, min.features = 200); data <- NormalizeData(data); data <- FindVariableFeatures(data, selection.method = "vst", nfeatures = 2000); data <- RunPCA(data); data <- FindNeighbors(data, dims = 1:10); data <- FindClusters(data, resolution = 0.4); data <- RunUMAP(data, dims = 1:10)
```

## Evaluation signals

- After FindClusters at resolution 0.4 and RunUMAP on dims 1:10, DimPlot(group.by = 'accordion_per_cluster') should show coherent annotation clusters
- Cell-level QC should have removed cells with <200 or >6000 expressed genes and cells with 20% or more mitochondrial counts
- LHSC and neoplastic monocyte per-cell scores should be enriched in AML patient cells relative to healthy controls, reproducing Fig. 5C-H patterns
- Exported aml_disease_scores.csv must contain per-cell scores alongside healthy-vs-AML condition metadata for every cell
- FeaturePlots rendered with min.cutoff='q15'/max.cutoff='q95' (LHSC) and 'q10'/'q90' (neoplastic monocyte) should show distinguishable per-cell score gradients

## Limitations

- The disease benchmark covered only two liquid and two solid tumors, so performance in other contexts is unverified
- Ground-truth datasets of high quality are scarce, limiting benchmark expansion and validation of annotations
- Single-cell MDS data came from a small patient cohort, constraining generalizability of hematopoietic findings

## Evidence

- [methods] Single-cell datasets were analyzed employing a standard pre-processing pipeline for single-cell RNA-seq data, using Seurat: "Single-cell datasets were analyzed employing a standard pre-processing pipeline for single-cell RNA-seq data, using Seurat"
- [methods] cells with less than 200 expressed genes and more than 6000 were filtered out: "To ensure the removal of empty droplets and potential doublets, cells with less than 200 expressed genes and more than 6000 were filtered out"
- [readme] data <- CreateSeuratObject(counts = counts, min.cells = 3, min.features = 200): "data <- CreateSeuratObject(counts = counts, min.cells = 3, min.features = 200)"
- [methods] The MERFISH mouse brain dataset was analyzed employing a standard pre-processing pipeline for imaging-based spatial dataset, using Seurat: "The MERFISH mouse brain dataset was analyzed employing a standard pre-processing pipeline for imaging-based spatial dataset, using Seurat"
- [methods] Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
