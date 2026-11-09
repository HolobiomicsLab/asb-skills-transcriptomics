---
name: merfish-spatial-transcriptomics-preprocessing
description: Use when handling an imaging-based spatial transcriptomics dataset (e.g.,
  MERFISH) that requires standard Seurat preprocessing and alignment of cell spatial
  coordinates to a anatomical reference before annotation or benchmarking.
when_to_use_negative:
- Input is a dissociated single-cell RNA-seq dataset with no spatial coordinates —
  use a standard scRNA-seq Seurat workflow instead
- Data are sequencing-based spatial transcriptomics (e.g., Visium) rather than imaging-based
  MERFISH, where the paper's standard imaging-based pre-processing pipeline does not
  directly apply
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Performs SCT-transform normalization, PCA, FindNeighbors/FindClusters (Louvain)
    clustering, and UMAP visualization of the MERFISH dataset
- name: Cell Marker Accordion
  role: Consumes the preprocessed, clustered Seurat object for downstream automatic
    cell type annotation via the accordion() function
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:23e948e348547688000fbcec88c57181f13abbe2473702aa2795b6b985705f92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - SCINA
  - ScType
  - Seurat
  - clustifyR
  - ontologySimilarity
  - scCATCH
  - scSorter
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# merfish-spatial-transcriptomics-preprocessing

## Summary

Preprocessing pipeline for imaging-based MERFISH spatial transcriptomics data using Seurat, covering SCT-transform normalization, PCA, Louvain clustering, UMAP visualization, and mapping of cell spatial coordinates to a reference atlas (Allen CCF 2020) prior to cell type annotation.

## When to use

Use when handling an imaging-based spatial transcriptomics dataset (e.g., MERFISH) that requires standard Seurat preprocessing and alignment of cell spatial coordinates to a anatomical reference before annotation or benchmarking.

## When NOT to use

- Input is a dissociated single-cell RNA-seq dataset with no spatial coordinates — use a standard scRNA-seq Seurat workflow instead
- Data are sequencing-based spatial transcriptomics (e.g., Visium) rather than imaging-based MERFISH, where the paper's standard imaging-based pre-processing pipeline does not directly apply

## Inputs

- MERFISH mouse brain spatial transcriptomics dataset (cell x gene count matrix with spatial coordinates)
- Allen CCF 2020 atlas NIfTI images with 0.01 mm voxel spacing

## Outputs

- Seurat object with SCT-normalized expression, PCA, Louvain clusters, and UMAP embeddings
- Per-cell spatial coordinates mapped to Allen CCF 2020 anatomical regions

## How to apply

Load the MERFISH dataset into Seurat (the paper used Seurat v5.0.3) and run a standard imaging-based spatial pipeline: SCT-transform normalization, PCA, then cell clustering with FindNeighbors/FindClusters (Louvain) and UMAP visualization. Select the section of interest — in the benchmark, coronal brain section 'Zhuang-ABCA-1.080' with 37,068 barcodes — and map cell spatial coordinates to the Allen CCF 2020 atlas using NIfTI images with 0.01 mm voxel spacing. The preprocessed, cluster-labeled Seurat object then serves as input for annotation tools such as the Cell Marker Accordion accordion() function. Rationale: a consistent standard pre-processing pipeline ensures cluster definitions and spatial mappings are comparable across tools being benchmarked on the same section.

## Related tools

- **Seurat** (Performs SCT-transform normalization, PCA, FindNeighbors/FindClusters (Louvain) clustering, and UMAP visualization of the MERFISH dataset)
- **Cell Marker Accordion** (Consumes the preprocessed, clustered Seurat object for downstream automatic cell type annotation via the accordion() function) — https://github.com/TebaldiLab/cellmarkeraccordion

## Examples

```
data <- SCTTransform(data); data <- RunPCA(data); data <- FindNeighbors(data, dims = 1:10); data <- FindClusters(data); data <- RunUMAP(data, dims = 1:10)
```

## Evaluation signals

- The selected section matches the expected barcode count (e.g., 'Zhuang-ABCA-1.080' yields 37,068 barcodes)
- Every cell in the Seurat object has both a Louvain cluster assignment and spatial coordinates mapped to an Allen CCF 2020 region
- UMAP embedding reproduces spatial structure consistent with the original MERFISH annotation
- Downstream annotation achieves high similarity to ground truth — the Cell Marker Accordion showed over 13% improvement with spatial mapping of high similarity to the original annotation

## Limitations

- The pipeline is tuned for imaging-based spatial datasets; the paper describes it specifically for 'The MERFISH mouse brain dataset ... using Seurat' and does not validate it on other spatial platforms
- Accurate CCF mapping depends on NIfTI images with 0.01 mm voxel spacing; different voxel resolutions or atlas versions would require re-verification

## Evidence

- [methods] The MERFISH mouse brain dataset was analyzed employing a standard pre-processing pipeline for imaging-based spatial dataset, using Seurat: "The MERFISH mouse brain dataset was analyzed employing a standard pre-processing pipeline for imaging-based spatial dataset, using Seurat"
- [other] Select the coronal brain section and map spatial coordinates to Allen CCF: [evidence span withheld — anchor status: not_found]
- [other] Annotation performance improved by over 13% and spatial mapping showed high similarity: [evidence span withheld — anchor status: not_found]
- [readme] README: process and cluster the data with Seurat: "data <- NormalizeData(data)
data <- FindVariableFeatures(data, selection.method = "vst", nfeatures = 2000)
data <- ScaleData(data)
data <- RunPCA(data, features = VariableFeatures(object = data))"
