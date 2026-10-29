---
name: visium-spot-data-preprocessing
description: Use when preparing a 10x Visium slice (e.g. the 2518-spot, 17,943-gene
  human breast cancer slice) and a scRNA-seq reference so that both are normalized,
  scaled, and share 2000 common HVGs before coupled-VAE/Minibatch-UOT integration.
when_to_use_negative:
- When the data are already normalized/scaled or an integrated embedding exists —
  re-running filter_data/batch_scale would distort values
- When no paired scRNA-seq reference is available, since common HVG identification
  across datasets is the core of this preprocessing
- When integrating data profiled from the same cells (vertical integration) rather
  than spatial-to-reference deconvolution
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: uniPort
  role: Provides load_file, filter_data, and batch_scale preprocessing functions to
    normalize, filter, and MaxAbs-scale both Visium and scRNA datasets before Run()
  repo: https://github.com/caokai1073/uniport
- name: scanpy
  role: Used for per-cell total-count normalization (scanpy.pp.normalize_total) and
    log1p normalization with offset 1
- name: Seurat
  role: Loads the 10x Visium slice from the cellranger folder via Load10X_Spatial,
    then NormalizeData and ScaleData
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:fb684c0c7ce1c8944bb1ac360084a0c1582ddbbcd6b5b453bba03a7b42a6a302
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - Seurat
  - scanpy
  - scatterpie
  - uniPort
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# visium-spot-data-preprocessing

## Summary

Preprocess 10x Visium spatial spot data and its scRNA reference for uniPort-based spatial deconvolution: load raw count matrices, normalize (scanpy.pp.normalize_total, log1p with offset 1), scale per-dataset with MaxAbsScaler, and select k=2000 highly variable common genes plus kx=ky=2000 dataset-specific HVGs.

## When to use

Use when preparing a 10x Visium slice (e.g. the 2518-spot, 17,943-gene human breast cancer slice) and a scRNA-seq reference so that both are normalized, scaled, and share 2000 common HVGs before coupled-VAE/Minibatch-UOT integration.

## When NOT to use

- When the data are already normalized/scaled or an integrated embedding exists — re-running filter_data/batch_scale would distort values
- When no paired scRNA-seq reference is available, since common HVG identification across datasets is the core of this preprocessing
- When integrating data profiled from the same cells (vertical integration) rather than spatial-to-reference deconvolution

## Inputs

- 10x Visium spatial slice (cellranger output folder, e.g. 'st', or AnnData .h5ad)
- Reference scRNA-seq AnnData with cell-type annotations (e.g. Whole_miniatlas_meta.csv metadata with celltype_major labels)
- Number of highly variable common genes (k=2000) and dataset-specific HVGs (kx=ky=2000)

## Outputs

- Normalized/scaled Visium AnnData with dataset-specific HVGs
- Normalized/scaled scRNA reference AnnData with its specific HVGs
- adata_cm: AnnData restricted to the 2000 highly variable common genes

## How to apply

Load the Visium data (e.g. from the cellranger 'st' folder via Seurat's Load10X_Spatial, or .h5ad via uniPort's load_file) and normalize total counts per cell with scanpy.pp.normalize_total, then log-normalize with an offset of 1 using scanpy.pp.log1p. Filter data and scale gene values to the 0-1 range within each dataset using uniPort's filter_data and batch_scale (MaxAbsScaler). Identify k=2000 highly variable common genes across all datasets into an adata_cm object, and kx=ky=2000 dataset-specific HVGs for each dataset; these common-specific gene splits let uniPort handle heterogeneity between the spatial and reference datasets. Note that per the paper, cell/gene filtering (e.g. <200 genes/cell, <3 cells/gene) was applied only to PBMC data and not to the spatial slices. The uniPort spatial helper functions (stClusterPie, mapCluster) are stored in 'spatial_function.R' in the repository's R process/spot directory.

## Related tools

- **uniPort** (Provides load_file, filter_data, and batch_scale preprocessing functions to normalize, filter, and MaxAbs-scale both Visium and scRNA datasets before Run()) — https://github.com/caokai1073/uniport
- **scanpy** (Used for per-cell total-count normalization (scanpy.pp.normalize_total) and log1p normalization with offset 1)
- **Seurat** (Loads the 10x Visium slice from the cellranger folder via Load10X_Spatial, then NormalizeData and ScaleData)

## Examples

```
import scanpy as sc; sc.pp.normalize_total(adata); sc.pp.log1p(adata)
```

## Evaluation signals

- Both AnnData objects carry log1p-normalized values (offset 1) and gene values scaled to the 0-1 range via MaxAbsScaler within each dataset
- adata_cm contains exactly 2000 highly variable common genes; each dataset object contains 2000 dataset-specific HVGs (kx=ky=2000)
- Spot count of the Visium slice matches the source data (2518 spots, 17,943 genes for the human breast cancer slice)
- No cells or genes were filtered from the spatial data (per the paper, filtering applied only to PBMC data)
- Downstream up.Run(adatas=[adata1, adata2], adata_cm=adata_cm) converges and can output a global OT plan with save_OT=True

## Limitations

- Storing a dense global OT plan T (save_OT=True) requires substantial memory; verify memory capacity before use
- Which hyperparameter combination from the searched grids (λ ∈ {0.5, 1.0, 5.0}, γ ∈ {0.5, 1.0}, λs ∈ {0.5, 1.0}) was selected per dataset is not stated
- Filtering thresholds (<200 genes/cell, <3 cells/gene) were only defined for PBMC data; other datasets, including the spatial slices, had no filtering applied

## Evidence

- [discussion] highly variable common genes: "We identified k = 2000 highly variable common genes across cells of all datasets, and identified kx = ky = 2000 highly variable genes for each dataset"
- [discussion] log-normalization: "We performed log-normalization of all datasets with an offset of 1 using the scanpy.pp.log1p function"
- [discussion] MaxAbs scaling: "We normalized values of each gene to the range of 0-1 within each dataset by the MaxAbsScaler function"
- [discussion] filtering policy: "We filtered out cells with fewer than 200 genes and filtered out genes observed in fewer than 3 cells for PBMC data. No cells or genes were filtered out for other data"
- [methods] preprocessing functions: "remove AnnData returns in *filter_data()* and *batch_scale()* functions."
- [methods] spatial helper script: "The 'spatial_function.R' is stored `here <https://github.com/caokai1073/uniPort/tree/main/R%20process/spot/>`_."
