---
name: scanpy-scrna-seq-data-handling
description: Use when raw scRNA-seq count data (e.g. 10X matrix.mtx with barcodes.tsv/genes.tsv)
  must be loaded into an AnnData object and pre-processed (normalize_per_cell, log1p,
  HVG selection) before dimensionality reduction or imputation.
when_to_use_negative:
- Input is already a preprocessed feature/embedding table (e.g. an existing .obsm
  projection or imputed matrix) rather than raw count data — no loading/filtering
  is needed
- The analysis target is Seurat-based feature selection of the top 5000 HVGs (as in
  the Klein imputation example, which uses R via klein_filterHVG.R) rather than Scanpy's
  cell_ranger dispersion filter
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Loads and pre-processes the scRNA-seq data (sc.read of matrix.mtx, normalize_per_cell,
    log1p, filter_genes_dispersion) and holds reduced representations in adata.obsm
  repo: https://github.com/theislab/scanpy
- name: AE-TPGG
  role: Downstream model consuming the Scanpy-preprocessed semi-continuous AnnData;
    writes its low-dimensional representation back into adata_filtered.obsm['X_ae_tpgg']
  repo: https://github.com/PUGEA/AE-TPGG
- name: Seurat
  role: Alternative feature-selection path used in the Klein example to select the
    top 5000 highly variable genes before Scanpy-side preprocessing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11704-022-2011-y
    title: 'AE-TPGG: a novel autoencoder-based approach for single-cell RNA-seq data
      imputation and dimensionality reduction'
  source_content_hash: sha256:30c52f6a4f536f6d721e1daaa6b74b906e7c0b8a6d5a9c5e6d7de4d2ef8ce401
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/theislab/scanpy
  tools:
  - AE-TPGG
  - Python 3
  - Scanpy
  - Seurat
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# scanpy-scrna-seq-data-handling

## Summary

Load, pre-process, and manipulate single-cell RNA-seq data with Scanpy — reading 10X matrix.mtx/genes.tsv/barcodes.tsv files, per-cell normalization, log1p transformation, and highly-variable-gene filtering — as the data-handling backbone for downstream AE-TPGG dimensionality reduction and imputation.

## When to use

Use when raw scRNA-seq count data (e.g. 10X matrix.mtx with barcodes.tsv/genes.tsv) must be loaded into an AnnData object and pre-processed (normalize_per_cell, log1p, HVG selection) before dimensionality reduction or imputation.

## When NOT to use

- Input is already a preprocessed feature/embedding table (e.g. an existing .obsm projection or imputed matrix) rather than raw count data — no loading/filtering is needed
- The analysis target is Seurat-based feature selection of the top 5000 HVGs (as in the Klein imputation example, which uses R via klein_filterHVG.R) rather than Scanpy's cell_ranger dispersion filter

## Inputs

- 10X-formatted scRNA-seq count matrix (matrix.mtx) with genes.tsv and barcodes.tsv, e.g. from the 68K PBMC dataset (10XGenomics/single-cell-3prime-paper)
- CSV count matrix (e.g. klein_filtered_data.csv) for the Klein dataset imputation example
- AnnData object (adata) with count matrix in .X

## Outputs

- AnnData object with per-cell-normalized, log1p-transformed .X and cell/barcode metadata
- filter_result from sc.pp.filter_genes_dispersion with gene_subset boolean mask of the top 1000 HVGs
- adata_filtered AnnData (HVG-subset, semi-continuous form) with reduced representation in .obsm['X_ae_tpgg']

## How to apply

Read the 10X-formatted count matrix with sc.read(..., cache=True) and transpose it so cells are observations, then assign adata.var_names from genes.tsv (column 2, tab-separated) and adata.obs_names from barcodes.tsv. Normalize with sc.pp.normalize_per_cell (total UMI count per cell), and for feature selection run sc.pp.filter_genes_dispersion with flavor='cell_ranger', n_top_genes=1000, log=False on a normalized copy, keeping filter_result.gene_subset as the HVG mask. For semi-continuous preprocessing before AE-TPGG, apply sc.pp.normalize_per_cell then sc.pp.log1p to the full adata and subset columns to the HVG subset (adata_filtered). Reduced representations produced by downstream models are stored in adata_filtered.obsm (e.g. 'X_ae_tpgg'), which is the interface for saving projections via np.savetxt or for clustering evaluation against the 10 purified PBMC subtype labels.

## Related tools

- **Scanpy** (Loads and pre-processes the scRNA-seq data (sc.read of matrix.mtx, normalize_per_cell, log1p, filter_genes_dispersion) and holds reduced representations in adata.obsm) — https://github.com/theislab/scanpy
- **AE-TPGG** (Downstream model consuming the Scanpy-preprocessed semi-continuous AnnData; writes its low-dimensional representation back into adata_filtered.obsm['X_ae_tpgg']) — https://github.com/PUGEA/AE-TPGG
- **Seurat** (Alternative feature-selection path used in the Klein example to select the top 5000 highly variable genes before Scanpy-side preprocessing)

## Examples

```
import scanpy as sc; adata = sc.read(path + 'matrix.mtx', cache=True).T; sc.pp.normalize_per_cell(adata); sc.pp.log1p(adata); adata_filtered = adata[:, filter_result.gene_subset]
```

## Evaluation signals

- adata.obs_names match barcodes.tsv entries and adata.var_names match genes.tsv column 2 after loading and transposing
- filter_result.gene_subset selects exactly 1000 genes when n_top_genes=1000 and flavor='cell_ranger'
- After preprocessing, adata_filtered.X is non-negative and contains many exact zeros consistent with a semi-continuous (zero-inflated, log1p-normalized) form
- adata_filtered.obsm['X_ae_tpgg'] exists with one row per filtered cell and can be written via np.savetxt('./pbmc68k_reduced_data', ...) without shape errors
- Downstream k-means clustering (k=10, matching the 10 purified PBMC cell subtypes) on the saved reduced data yields ACC/ARI/NMI/F1 values comparable to Figure 2 of the repository example

## Limitations

- Feature selection in the Scanpy path is limited to the top 1000 highly variable genes via filter_genes_dispersion; the 5000-HVG Seurat path requires an external R step (klein_filterHVG.R)
- README code (import scanpy.api as sc) targets an older Scanpy API; current Scanpy versions have removed scanpy.api
- Cell-subtype labels for evaluation come from 10 purified PBMC subtypes, so clustering metrics are only meaningful when those labels are available

## Evidence
