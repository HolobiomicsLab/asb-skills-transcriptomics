---
name: scrna-seq-anndata-preprocessing
description: Use when raw scRNA-seq matrices or downloaded dataset files (e.g. the
  5800-cell Setty CD34+ bone-marrow data from the VIA Datasets repo) must be loaded
  as AnnData with PCA precomputed before running VIA.
when_to_use_negative:
- The AnnData object already contains PCA in obsm['X_pca'] and a computed embedding,
  i.e. preprocessing is complete and via_wrapper can be called directly
- The analysis goal is downstream gene-trend plotting or F1-score computation on an
  already-inferred VIA trajectory, not data loading/normalization
- The input is simulated toy trajectory data (e.g. dyntoy 4-leaf or disconnected datasets)
  where ground-truth pseudotime already defines the structure and minimal preparation
  is needed
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: VIA
  role: Downstream consumer of the preprocessed AnnData; its wrapper requires PCA
    in adata.obsm['X_pca'] and parameters such as knn, ncomps and jac_std_global
  repo: https://github.com/ShobiStassen/VIA
- name: scanpy
  role: Construct the AnnData object, attach PCA to obsm['X_pca'], and compute UMAP
    embedding for visualization
- name: pandas
  role: Load expression matrices and lineage annotation tables from the VIA Datasets
    repository before packaging into AnnData
- name: UMAP
  role: Provides the 2D embedding used for single-cell level visualization alongside
    the trajectory computation on PCA space
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:6fb3381462bfb63d491ebebe73b2267371ca32bfaa3cd1d8ea1108d885539077
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - UMAP
  - VIA
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# scrna-seq-anndata-preprocessing

## Summary

Prepare scRNA-seq expression data into an AnnData object with PCA stored in adata.obsm['X_pca'] and a low-dimensional embedding so it can be consumed directly by VIA's via_wrapper for trajectory inference on datasets such as the Setty CD34+ hematopoiesis study.

## When to use

Use when raw scRNA-seq matrices or downloaded dataset files (e.g. the 5800-cell Setty CD34+ bone-marrow data from the VIA Datasets repo) must be loaded as AnnData with PCA precomputed before running VIA.

## When NOT to use

- The AnnData object already contains PCA in obsm['X_pca'] and a computed embedding, i.e. preprocessing is complete and via_wrapper can be called directly
- The analysis goal is downstream gene-trend plotting or F1-score computation on an already-inferred VIA trajectory, not data loading/normalization
- The input is simulated toy trajectory data (e.g. dyntoy 4-leaf or disconnected datasets) where ground-truth pseudotime already defines the structure and minimal preparation is needed

## Inputs

- scRNA-seq gene-expression matrix (genes x cells) e.g. the Setty 5800-cell CD34+ bone-marrow dataset
- Ground-truth lineage/cell-type annotations (per-cell labels such as mDC, pDC, megakaryocyte, erythroid, monocyte, B-cell)
- Precomputed PCA coordinates to be stored in adata.obsm['X_pca']
- Dataset files from github.com/ShobiStassen/VIA/tree/master/Datasets (AnnData/h5ad or linked files)

## Outputs

- AnnData object with expression, annotations, and PCA in adata.obsm['X_pca'] ready for via_wrapper
- 2D embedding (UMAP or PHate) attached for single-cell visualization
- Preprocessed object supporting downstream terminal-state detection and marker-gene trend extraction (IRF8, CSF1R, CD123)

## How to apply

Load the gene-expression matrix and ground-truth lineage annotations (scanpy, pandas) and construct an AnnData object in which a precomputed principal-component representation is assigned to adata.obsm['X_pca'], exactly as VIA's wrapper expects before calling via_wrapper with parameters like knn=30, ncomps=10-20 and jac_std_global=0.15. Compute a 2D embedding (UMAP or PHate) for single-cell-level visualization, since VIA's plotting functions require it but the trajectory computation runs on the PCA space. For the hematopoiesis case, fetch the CD34+ scRNA-seq AnnData from the VIA repository's Datasets folder or linked Google Drive, keeping the 5800-cell (5780-cell per README) structure and annotation labels intact so terminal-state F1-scores can later be matched to cDC, pDC, megakaryocyte, erythroid, monocyte and B-cell fates. The rationale is that VIA deliberately operates on a cluster-graph built over the high-dimensional reduced representation, so the PCA must be stored in the documented obsm slot rather than regenerated internally.

## Related tools

- **VIA** (Downstream consumer of the preprocessed AnnData; its wrapper requires PCA in adata.obsm['X_pca'] and parameters such as knn, ncomps and jac_std_global) — https://github.com/ShobiStassen/VIA
- **scanpy** (Construct the AnnData object, attach PCA to obsm['X_pca'], and compute UMAP embedding for visualization)
- **pandas** (Load expression matrices and lineage annotation tables from the VIA Datasets repository before packaging into AnnData)
- **UMAP** (Provides the 2D embedding used for single-cell level visualization alongside the trajectory computation on PCA space)

## Examples

```
adata = sc.AnnData(data_genes); adata.obsm['X_pca'] = data.values
```

## Evaluation signals

- adata.obsm['X_pca'] exists and matches VIA's documented input format before via_wrapper is called
- Cell count matches the source dataset (5800 cells reported in the task card; 5780 cells in the README's hematopoiesis tutorial entry)
- All per-cell lineage annotations (cDC, pDC, megakaryocyte, erythroid, monocyte, B-cell) are preserved and alignable with predicted terminal clusters for later F1 computation
- A 2D embedding (UMAP/PHate) is available for visualization, matching VIA's requirement that the embedding be supplied to the wrapper
- The preprocessed object carries the marker genes needed for later trend extraction: IRF8, CSF1R and CD123

## Limitations

- The paper's discussion gives no accession identifiers for the hematopoiesis dataset, so acquisition depends on the VIA repository Datasets folder or linked Google Drive files
- Methods requiring an embedding such as UMAP or t-SNE before analysis can be highly susceptible to user-defined input parameters, so the 2D embedding should be used for visualization rather than as the trajectory basis
- Larger datasets are distributed separately from the repository (Google Drive), and the README notes smaller files are in the Datasets folder while larger ones are external

## Evidence

- [methods] Load data as AnnData with PCA stored in adata.obsm['X_pca']: "Load data as AnnData with PCA stored in adata.obsm['X_pca']  [section=methods; evidence='adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values']"
- [methods] Compute UMAP embedding for single-cell visualization: "# use UMAP or PHate to obtain embedding that is used for single-cell level visualization"
- [methods] VIA wrapper call signature: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [readme] Hematopoiesis dataset entry in README: [evidence span withheld — anchor status: external_doc]
- [readme] Datasets folder location: "Dataset are available in the [Datasets folder](https://github.com/ShobiStassen/VIA/tree/master/Datasets) (smaller files)"
- [discussion] Embedding-dependence caveat: "We showed that methods which require UMAP (or t-SNE) before parsing MOCA are highly susceptible to user defined input parameters"
