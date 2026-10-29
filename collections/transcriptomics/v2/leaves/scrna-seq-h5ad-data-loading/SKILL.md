---
name: scrna-seq-h5ad-data-loading
description: Use when the input is a single-cell RNA-seq dataset in .h5ad format that
  must be read with Scanpy and preprocessed (HVG selection + PCA into obsm['X_pca'])
  before DV computation or sketching.
when_to_use_negative:
- adata.obsm['X_pca'] already exists (e.g. PCA was computed previously) — loading/re-running
  PCA is unnecessary and SCValue can be invoked directly
- The input is not an AnnData/.h5ad object (e.g. a raw count matrix or CSV) — Scanpy
  h5ad loading does not apply
- The goal is uniform subsampling in Seurat/Scanpy pipelines, which does not require
  the X_pca representation at all
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Scanpy
  role: Reads the .h5ad file and performs HVG selection (n_top_genes=3000) and PCA
    (n_comps=50) to populate adata.obsm['X_pca']
- name: scValue
  role: Consumes the prepared AnnData (X_pca representation + cell_type labels) as
    its adata input for value-based subsampling
  repo: https://github.com/LHBCB/scvalue
- name: AnnData
  role: In-memory container holding the .h5ad data, obsm['X_pca'] representation,
    and obs['cell_type'] labels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:d7011fd0de40e218afd548245cc9f6fa7f4cb76381abfab38ce3048905c40fe6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/LHBCB/scvalue
  repo_url: https://github.com/LHBCB/scvalue
  tools:
  - AnnData
  - Scanpy
  - scValue
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# scrna-seq-h5ad-data-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load a log-normalised scRNA-seq dataset in .h5ad format with Scanpy/AnnData and compute a PCA representation on the top 3,000 highly variable genes, storing 50 principal components in adata.obsm['X_pca'] as the feature matrix for downstream value-based subsampling with scValue.

## When to use

Use when the input is a single-cell RNA-seq dataset in .h5ad format that must be read with Scanpy and preprocessed (HVG selection + PCA into obsm['X_pca']) before DV computation or sketching.

## When NOT to use

- adata.obsm['X_pca'] already exists (e.g. PCA was computed previously) — loading/re-running PCA is unnecessary and SCValue can be invoked directly
- The input is not an AnnData/.h5ad object (e.g. a raw count matrix or CSV) — Scanpy h5ad loading does not apply
- The goal is uniform subsampling in Seurat/Scanpy pipelines, which does not require the X_pca representation at all

## Inputs

- scRNA-seq dataset in .h5ad format (log-normalised), read via Scanpy into an AnnData object
- Cell type labels in adata.obs['cell_type'] (column name configurable via cell_type_key)

## Outputs

- AnnData object with adata.var['highly_variable'] flags for the top 3,000 HVGs
- adata.obsm['X_pca'] containing 50 principal components used as the feature matrix X for random-forest DV computation

## How to apply

Read the .h5ad file with sc.read_h5ad into an AnnData object; the example mTC dataset (89,429 mouse T cells) is already log-normalised, so no re-normalisation is needed. Select the top 3,000 highly variable genes with sc.pp.highly_variable_genes(adata, n_top_genes=3000), then run sc.pp.pca(adata, n_comps=50), which stores the PCs in adata.obsm['X_pca']. This representation is the default feature matrix scValue expects (use_rep='X_pca'), with cell type labels taken from adata.obs['cell_type'] via cell_type_key. If 'X_pca' is absent, scValue cannot compute random-forest data values, so verify the key exists before invoking SCValue; alternatively pass use_rep='X' to use adata.X, noting that sparse matrices may be densified internally.

## Related tools

- **Scanpy** (Reads the .h5ad file and performs HVG selection (n_top_genes=3000) and PCA (n_comps=50) to populate adata.obsm['X_pca'])
- **scValue** (Consumes the prepared AnnData (X_pca representation + cell_type labels) as its adata input for value-based subsampling) — https://github.com/LHBCB/scvalue
- **AnnData** (In-memory container holding the .h5ad data, obsm['X_pca'] representation, and obs['cell_type'] labels)

## Examples

```
adata = sc.read_h5ad('mTC.h5ad'); sc.pp.highly_variable_genes(adata, n_top_genes=3000); sc.pp.pca(adata, n_comps=50)
```

## Evaluation signals

- adata.obsm['X_pca'] exists and has shape (n_cells, 50) after sc.pp.pca(adata, n_comps=50)
- adata.var['highly_variable'] contains exactly 3,000 True entries after sc.pp.highly_variable_genes(adata, n_top_genes=3000)
- The .h5ad expression matrix is log-normalised (the example mTC dataset ships in this state), matching scValue's expected input
- adata.obs['cell_type'] is present and non-null for all cells, since scValue uses it via cell_type_key for labels Y
- Instantiating SCValue with the default use_rep='X_pca' runs without error, confirming the representation key is correctly populated

## Limitations

- Using use_rep='X' on a sparse adata.X may trigger internal densification, which is memory-intensive for large datasets
- The example dataset is already log-normalised; datasets requiring normalisation or QC (duplicate removal, batch correction, imputation) need those steps before this loading skill yields a suitable input
- PCA on the top 3,000 HVGs is the paper's concrete preprocessing choice; other feature representations are not validated in the scValue workflow

## Evidence

- [other] Load the cell-typed scRNA-seq dataset in .h5ad format with Scanpy and take the adata.obsm['X_pca'] representation (PCA computed on the top 3,000 highly variable genes) as the feature matrix X: [evidence span withheld — anchor status: not_found]
