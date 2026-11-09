---
name: scanpy-single-cell-data-preprocessing
description: Use when a raw single-cell RNA-seq count matrix (AnnData/h5ad, e.g. the
  NEURIPS bone marrow dataset) must be filtered, normalized to 10,000 counts, log-transformed,
  and reduced to ~3500 highly variable genes before integration or benchmarking.
when_to_use_negative:
- Input is already preprocessed (e.g. latent embeddings from CODAL/scVI/scANVI used
  for silhouette/ASW computation) — applying count filtering or normalization again
  would be wrong.
- Raw ATAC-seq accessibility counts destined for CODAL's encoder, which are transformed
  using deviance residuals rather than the RNA-seq normalize_total/log1p workflow.
- Non-count data (e.g. already log-normalized or scaled matrices) where min-count
  filtering and depth normalization would distort the values.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scanpy
  role: Provides filter_cells/filter_genes, normalize_total, log1p, and highly_variable_genes
    used to preprocess the NEURIPS bone marrow expression data
- name: anndata
  role: Data container format (AnnData) that MIRA and scanpy operate on; each MIRA
    function takes an AnnData object as its first positional argument
- name: MIRA/CODAL
  role: Downstream consumer of the preprocessed data; CODAL topic-model inference
    is used for batch-effect correction on the normalized, HVG-reduced matrix
  repo: github.com/cistrome/MIRA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:44436f7ae23e24ec3aa933d8eaec9c1a8c3fc5a35816a14cf55f65ec3ebfb6d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastMNN
  - Harmony
  - MIRA
  - MIRA/CODAL
  - Scanorama
  - Seurat
  - anndata
  - scANVI
  - scVI
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# scanpy-single-cell-data-preprocessing

## Summary

Preprocess raw single-cell RNA-seq count data with the standard scanpy workflow — cell/gene count filtering, depth normalization, log+1 transformation, and highly variable gene selection — so batches are ready for latent-space integration methods like CODAL, scVI, or scANVI.

## When to use

Use when a raw single-cell RNA-seq count matrix (AnnData/h5ad, e.g. the NEURIPS bone marrow dataset) must be filtered, normalized to 10,000 counts, log-transformed, and reduced to ~3500 highly variable genes before integration or benchmarking.

## When NOT to use

- Input is already preprocessed (e.g. latent embeddings from CODAL/scVI/scANVI used for silhouette/ASW computation) — applying count filtering or normalization again would be wrong.
- Raw ATAC-seq accessibility counts destined for CODAL's encoder, which are transformed using deviance residuals rather than the RNA-seq normalize_total/log1p workflow.
- Non-count data (e.g. already log-normalized or scaled matrices) where min-count filtering and depth normalization would distort the values.

## Inputs

- Raw single-cell RNA-seq count matrix as an AnnData (.h5ad) object
- Expert-annotated cell type labels per cell
- Batch covariates (site/donor of origin) per cell

## Outputs

- Preprocessed AnnData object: cells with fewer than 400 counts and genes with fewer than 30 counts removed, normalized to target_sum 10000, log+1 transformed
- Matrix restricted to ~3500 highly variable genes (minimum dispersion threshold 0.3)
- Integration-ready feature matrix for downstream latent-space methods (CODAL, scVI, scANVI, etc.)

## How to apply

Load the raw gene expression counts into an AnnData object alongside cell type labels and batch (site/donor) covariates. First filter cells with fewer than 400 counts and genes with fewer than 30 counts. Then normalize read depths using scanpy's normalize_total function with target_sum set to 10000, followed by a log+1 transformation. Finally select highly variable genes with a minimum dispersion threshold of 0.3, which yields 3500 highly variable genes for the NEURIPS bone marrow benchmark. The rationale is that this standardized scanpy preprocessing matches the CODAL paper's data preparation, ensuring latent embeddings computed downstream (e.g. silhouette-based ASW benchmarks) are comparable across CODAL, scVI, scANVI, Harmony, Scanorama, MIRA, PCA, Seurat v3, and FastMNN.

## Related tools

- **scanpy** (Provides filter_cells/filter_genes, normalize_total, log1p, and highly_variable_genes used to preprocess the NEURIPS bone marrow expression data)
- **anndata** (Data container format (AnnData) that MIRA and scanpy operate on; each MIRA function takes an AnnData object as its first positional argument)
- **MIRA/CODAL** (Downstream consumer of the preprocessed data; CODAL topic-model inference is used for batch-effect correction on the normalized, HVG-reduced matrix) — github.com/cistrome/MIRA

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('neurips_bm_raw.h5ad'); sc.pp.filter_cells(adata, min_counts=400); sc.pp.filter_genes(adata, min_counts=30); sc.pp.normalize_total(adata, target_sum=10000); sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata, min_disp=0.3)
```

## Evaluation signals

- Cell and gene filters applied: no cells with fewer than 400 counts and no genes with fewer than 30 counts remain in the processed AnnData.
- Depth normalization check: library-size distribution of the normalized matrix is centered on target_sum 10000 before log1p.
- Feature-space check: highly variable gene selection with minimum dispersion 0.3 yields ~3500 genes for the NEURIPS bone marrow dataset.
- Downstream validity: the preprocessed matrix, joined with expert-annotated cell type labels and batch covariates, supports per-method ASW benchmark tables where cell type silhouette (higher better) and batch silhouette (lower better) are computed for all methods including CODAL and scANVI.

## Limitations

- Results from batch-confounded analyses require orthogonal biological validation to rule out under-correction of technical effects.
- Preprocessing choices are dataset-specific; the 400-count cell / 30-count gene / dispersion 0.3 / 3500-HVG thresholds were defined for the NEURIPS bone marrow benchmark and may need re-tuning elsewhere.
- Authors caution that batch correction does not remove the need for good experimental design and appropriate controls (e.g. control cells in both treatment and control batches).

## Evidence

- [methods] cells with fewer than 400 counts and genes with fewer than 30 counts filtered out: "cells with fewer than 400 counts and genes with fewer than 30 counts filtered out"
- [methods] normalize_total target_sum 10000: "cell read depths normalized using the normalize_total function with target_sum set to 10000"
- [methods] highly variable genes minimum dispersion 0.3: "highly variable genes found with a minimum dispersion threshold of 0.3, yielding 3500 highly variable genes"
- [readme] MIRA/scanpy AnnData convention: "In general, the first positional argument of a MIRA function is an AnnData object"
