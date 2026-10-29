---
name: count-matrix-normalization-scaling
description: Use when a raw or normalized count matrix (genes x cells) from scRNA-seq
  or spatial data must be normalized, scaled, and clustered prior to automatic cell-type
  annotation with consistency-weighted markers.
when_to_use_negative:
- Input is already a processed Seurat object with normalized and scaled assays and
  defined clusters — only the SE-on-marker-genes step or annotation is needed
- The task is marker-database curation (computing SPs/ECs weights) rather than expression-matrix
  preprocessing
- Input is variant-level single-cell data (e.g. FreeBayes/VarTrix VCF workflows),
  not gene-expression counts
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: NormalizeData/FindVariableFeatures(vst, nfeatures=2000)/ScaleData/RunPCA/FindNeighbors/FindClusters/RunUMAP
    preprocessing; also restricted scaling to marker genes for SE
- name: Cell Marker Accordion
  role: Consumes the normalized/scaled counts (SE on marker genes) to compute gene
    impact scores and annotate cells via accordion()
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: data.table
  role: Builds the (cell, cluster) table passed as cluster_info to accordion() when
    starting from a raw counts matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:c8bfc8eca2f4c7364e4c1589198327ae4e9f906c200ec8f6fcb00dd662e3fd3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - Seurat
  - data.table
  - ontologyIndex
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# count-matrix-normalization-scaling

## Summary

Standard Seurat preprocessing of raw single-cell counts — normalization, variable feature selection, scaling, PCA, neighbor/clustering, and UMAP — and the marker-gene-restricted scaling (SE, scaled expression) that feeds the Cell Marker Accordion impact-score computation.

## When to use

Use when a raw or normalized count matrix (genes x cells) from scRNA-seq or spatial data must be normalized, scaled, and clustered prior to automatic cell-type annotation with consistency-weighted markers.

## When NOT to use

- Input is already a processed Seurat object with normalized and scaled assays and defined clusters — only the SE-on-marker-genes step or annotation is needed
- The task is marker-database curation (computing SPs/ECs weights) rather than expression-matrix preprocessing
- Input is variant-level single-cell data (e.g. FreeBayes/VarTrix VCF workflows), not gene-expression counts

## Inputs

- raw or normalized count matrix (genes on rows, cells on columns), e.g. counts.rda PBMC example from 10X Genomics
- Seurat object from CreateSeuratObject (min.cells=3, min.features=200)
- cluster id table (cell, cluster) when annotating from a raw counts matrix

## Outputs

- Seurat object with normalized/scaled assays, PCA, neighbors, clusters (seurat_clusters) and UMAP
- scaled expression (SE) matrix restricted to the input marker genes for impact-score computation
- annotated Seurat object with accordion_per_cluster cell-type labels and detailed annotation info

## How to apply

Load the raw counts (e.g. the PBMC counts.rda from system.file('extdata', package='cellmarkeraccordion')) into a Seurat object with CreateSeuratObject(counts=..., min.cells=3, min.features=200). Run the standard Seurat pipeline: NormalizeData, FindVariableFeatures (selection.method='vst', nfeatures=2000), ScaleData, RunPCA on variable features, FindNeighbors (dims 1:10), FindClusters (resolution 0.4), RunUMAP (dims 1:10). For the Cell Marker Accordion scoring itself, the count matrix is additionally normalized and scaled restricted to the input marker genes to obtain scaled expression (SE), which is multiplied by ECs_reg x SPs_reg per cell type, marker gene, and cell to yield gene impact scores (G_IMs). The accordion() function accepts either a Seurat object or a raw/normalized count matrix with genes on rows and cells on columns, so this preprocessing can precede an annotation_resolution='cluster' call (default) or a per-cell call. Rationale: clustering gives per-cluster annotations while the SE on marker genes underpins the positive/negative marker impact-score aggregation (CT_IMs).

## Related tools

- **Seurat** (NormalizeData/FindVariableFeatures(vst, nfeatures=2000)/ScaleData/RunPCA/FindNeighbors/FindClusters/RunUMAP preprocessing; also restricted scaling to marker genes for SE)
- **Cell Marker Accordion** (Consumes the normalized/scaled counts (SE on marker genes) to compute gene impact scores and annotate cells via accordion()) — https://github.com/TebaldiLab/cellmarkeraccordion
- **data.table** (Builds the (cell, cluster) table passed as cluster_info to accordion() when starting from a raw counts matrix)

## Examples

```
data <- CreateSeuratObject(counts = counts, min.cells = 3, min.features = 200); data <- NormalizeData(data); data <- FindVariableFeatures(data, selection.method = "vst", nfeatures = 2000); data <- ScaleData(data); data <- RunPCA(data); data <- FindNeighbors(data, dims = 1:10); data <- FindClusters(data, resolution = 0.4); data <- RunUMAP(data, dims = 1:10); data <- accordion(data, assay="RNA", species="Human", tissue="blood", annotation_resolution="cluster")
```

## Evaluation signals

- CreateSeuratObject applied with min.cells=3 and min.features=200 yields a valid Seurat object with no cells lost to the normalization itself
- PCA and neighbors/UMAP run on dims 1:10 with 2000 vst variable features; FindClusters uses resolution 0.4 producing seurat_clusters for every cell
- SE matrix is computed only on the input marker genes, so every gene used in impact scoring has a scaled-expression value
- Running accordion() with annotation_resolution='cluster' assigns exactly one winning cell type per cluster (and per cell in per-cell mode) with no NA annotations
- Script runs end-to-end on the PBMC counts.rda example without errors

## Limitations

- Preprocessing is a standard Seurat pipeline; the article does not specify dataset-specific thresholds beyond min.cells=3, min.features=200, nfeatures=2000, dims 1:10, resolution 0.4, and 20% mitochondrial-count filtering
- Direct comparison of Cell Marker Accordion with non-marker-based annotation methods was not conducted in the study
- Downstream annotation quality depends on tissue/species coverage of the built-in database (tissue must exist, e.g. list_tissues(species='Human'))

## Evidence

- [methods] Single-cell datasets were analyzed employing a standard pre-processing pipeline for single-cell RNA-seq data, using Seurat: "Single-cell datasets were analyzed employing a standard pre-processing pipeline for single-cell RNA-seq data, using Seurat"
- [other] Normalize and scale the count matrix on the input marker genes: [evidence span withheld — anchor status: not_found]
- [methods] cells with less than 200 expressed genes and more than 6000 were filtered out: "To ensure the removal of empty droplets and potential doublets, cells with less than 200 expressed genes and more than 6000 were filtered out"
- [readme] It requires in input only a Seurat object or a raw or normalized count matrix with genes on rows and cells on columns: "It requires in input only a Seurat object or a raw or normalized count matrix with genes on rows and cells on columns"
