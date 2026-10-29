---
name: single-cell-count-matrix-preprocessing
description: Use when a raw scRNA-seq count matrix must be converted into a normalized,
  feature-selected expression matrix (anndata/Scanpy) as input to a graph-based clustering
  pipeline such as scMGCA.
when_to_use_negative:
- Input is already a normalized, feature-selected expression matrix produced by a
  previous preprocessing run — re-filtering could remove informative genes.
- The downstream analysis requires full-gene-resolution data (e.g. marker-gene differential
  expression on all genes) rather than the top d highly variable genes.
- Input is spatial or bulk RNA-seq data rather than single-cell count data.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCANPY
  role: Toolkit used to normalize the scRNA-seq gene expression matrix, filter non-expressed
    genes, and select highly variable genes
- name: anndata
  role: Data container for loading and passing the preprocessed cell-by-gene matrix
- name: scMGCA
  role: Downstream consumer of the preprocessed matrix for cell-graph construction
    and clustering
  repo: https://pypi.org/project/scMGCA
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:3545174859c4999c9274a2ad9f884f9153bdf5948aed5a1734c436f32e0eed28
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCANPY
  - anndata
  - numpy
  - pandas
  - scMGCA
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-count-matrix-preprocessing

## Summary

Preprocess raw scRNA-seq count matrices for scMGCA-style graph clustering by filtering out genes not expressed in any cell, normalizing the expression matrix, and retaining the top d highly variable genes with SCANPY. This reduces dimensionality and the impact of high dropout rates before cell-graph construction.

## When to use

Use when a raw scRNA-seq count matrix must be converted into a normalized, feature-selected expression matrix (anndata/Scanpy) as input to a graph-based clustering pipeline such as scMGCA.

## When NOT to use

- Input is already a normalized, feature-selected expression matrix produced by a previous preprocessing run — re-filtering could remove informative genes.
- The downstream analysis requires full-gene-resolution data (e.g. marker-gene differential expression on all genes) rather than the top d highly variable genes.
- Input is spatial or bulk RNA-seq data rather than single-cell count data.

## Inputs

- Raw scRNA-seq gene expression count matrix (cell x gene), e.g. from public repositories
- anndata/AnnData object readable by SCANPY
- Cell-type label annotations (for benchmark datasets used in evaluation)

## Outputs

- Normalized expression matrix restricted to the top d highly variable genes
- Preprocessed AnnData object ready for KNN graph construction in scMGCA

## How to apply

Load the scRNA-seq count matrix into an anndata object and, as the article's first preprocessing step, filter out genes that are not expressed in any cells to reduce the impact of high dropout rates in scRNA-seq data. Normalize the gene expression matrix in SCANPY and retain the top d highly variable genes that carry high-level information. Keep the cell-by-gene matrix organized so downstream steps (KNN graph construction, random surfing, PPMI computation) can consume it directly. The value of d is dataset-dependent and the paper does not give a single numeric value, so choose it per dataset (a common SCANPY default is 2000, but any documented choice should be recorded). Validate that the output matrix is nonnegative, has no all-zero genes, and contains exactly the selected highly variable genes.

## Related tools

- **SCANPY** (Toolkit used to normalize the scRNA-seq gene expression matrix, filter non-expressed genes, and select highly variable genes)
- **anndata** (Data container for loading and passing the preprocessed cell-by-gene matrix)
- **scMGCA** (Downstream consumer of the preprocessed matrix for cell-graph construction and clustering) — https://pypi.org/project/scMGCA

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('raw_counts.h5ad'); sc.pp.filter_genes(adata, min_cells=1); sc.pp.normalize_total(adata); sc.pp.highly_variable_genes(adata, n_top_genes=d); adata = adata[:, adata.var.highly_variable]
```

## Evaluation signals

- No all-zero genes remain in the output matrix (genes not expressed in cells were filtered out)
- Output matrix values are nonnegative and consistent with library normalization
- Number of retained genes equals the chosen top d highly variable genes
- Downstream scMGCA run on the preprocessed matrix (e.g. the demo dataset) executes end-to-end without errors
- Clustering metrics (ARI, NMI) computed on benchmark datasets fall in the expected ranges reported against the paper's baseline

## Limitations

- The number of highly variable genes d is not given a numeric value in the paper, so it must be chosen per dataset
- Software version numbers (Python, TensorFlow, SCANPY) are not reported, which can complicate exact reproduction
- Sample data were only described as obtained from public repositories, with no specific dataset accessions provided

## Evidence

- [discussion] Filter out genes not expressed in cells: "The first step is to filter out genes that are not expressed in cells, reducing the impact of the high dropout rates in scRNA-seq data."
- [discussion] Keep top d highly variable genes: "the top d highly variable genes with high-level information are used in the study"
- [results] Normalize and select HVGs: "We first normalized the scRNA-seq gene expression matrix and selected the highly variable genes"
- [readme] SCANPY as the analysis toolkit: "SCANPY is a scalable toolkit for analyzing single-cell gene expression data built jointly with anndata"
- [discussion] Dataset accessions missing: "Sample data were obtained from public repositories."
