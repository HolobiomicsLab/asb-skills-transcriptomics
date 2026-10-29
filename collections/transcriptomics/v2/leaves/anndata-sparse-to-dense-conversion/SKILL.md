---
name: anndata-sparse-to-dense-conversion
description: Use when a scanpy-read .h5ad file (e.g. CITEseq.h5ad) returns a sparse
  matrix in adata.X and a downstream model such as MarkerMap's VAE expects a dense
  numpy array.
when_to_use_negative:
- The input adata.X is already a dense array (calling .toarray() would raise an AttributeError)
- Downstream tools (e.g. some scanpy workflows) natively handle sparse matrices and
  memory is constrained, since densification can explode memory for large gene-by-cell
  matrices
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scanpy
  role: Reads the CITE-seq .h5ad file and returns the AnnData object holding the sparse
    count matrix in adata.X
- name: anndata
  role: Provides the AnnData container whose adata.X sparse matrix is converted in
    place to a dense array
- name: numpy
  role: Supplies the ndarray format that adata.X is converted into via toarray() for
    dense handling
- name: MarkerMap
  role: Downstream VAE-based marker selection package whose models require dense input
    arrays
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:7b287ec96cf479b5d2e50cefbc9b76506250a50199b15111d036d6481cd350c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap
  tools:
  - MarkerMap
  - anndata
  - numpy
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# anndata-sparse-to-dense-conversion

## Summary

Converts the sparse count matrix stored in an AnnData object (adata.X) into a dense numpy array, as required before feeding CITE-seq scRNA-seq data into MarkerMap's variational autoencoder models, which expect dense input.

## When to use

Use when a scanpy-read .h5ad file (e.g. CITEseq.h5ad) returns a sparse matrix in adata.X and a downstream model such as MarkerMap's VAE expects a dense numpy array.

## When NOT to use

- The input adata.X is already a dense array (calling .toarray() would raise an AttributeError)
- Downstream tools (e.g. some scanpy workflows) natively handle sparse matrices and memory is constrained, since densification can explode memory for large gene-by-cell matrices

## Inputs

- AnnData object (.h5ad) with a sparse adata.X count matrix, e.g. data/cite_seq/CITEseq.h5ad read via scanpy
- Cell labels in adata.obs['names'] to be mapped to adata.obs['annotation']

## Outputs

- AnnData object with adata.X as a dense numpy array of gene counts
- Dense expression matrix ready for gene filtering, normalization, log2(1+x) transform, z-scaling, and MarkerMap training

## How to apply

Load the CITE-seq data (data/cite_seq/CITEseq.h5ad) with scanpy via sc.read_h5ad, which returns an AnnData object whose gene expression counts are stored in adata.X as a sparse array. Because MarkerMap models are built on dense tensors, convert the counts in place with adata.X = adata.X.toarray(). Perform this conversion immediately after loading and before any preprocessing steps such as housekeeping-gene removal, per-cell library-size normalization, log2(1+x) transformation, and gene-wise z-scaling. Cell labels should be stored in adata.obs['annotation'] (mapped from adata.obs['names']) so the dense matrix pairs correctly with labels for training, validation, and test splits.

## Related tools

- **scanpy** (Reads the CITE-seq .h5ad file and returns the AnnData object holding the sparse count matrix in adata.X)
- **anndata** (Provides the AnnData container whose adata.X sparse matrix is converted in place to a dense array)
- **numpy** (Supplies the ndarray format that adata.X is converted into via toarray() for dense handling)
- **MarkerMap** (Downstream VAE-based marker selection package whose models require dense input arrays) — https://github.com/Computational-Morphogenomics-Group/MarkerMap

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('data/cite_seq/CITEseq.h5ad'); adata.X = adata.X.toarray()
```

## Evaluation signals

- type(adata.X) is numpy.ndarray (not scipy.sparse.csr_matrix or similar) after conversion
- adata.X.shape is unchanged before and after conversion (same number of cells and genes)
- Count values are identical: adata.X[i, j] matches the original sparse matrix entries, with explicit zeros filling previously implicit zero entries
- Downstream MarkerMap.prepareData / MarkerMap model construction runs without sparse-input errors
- The preprocessed_adata.h5ad written after filtering, normalization, log2(1+x), and z-scaling retains the dense matrix

## Limitations

- Densifying large single-cell matrices can consume substantial memory since every implicit zero becomes an explicit stored value
- The conversion is only needed because MarkerMap models expect dense input; sparse-aware pipelines do not require this step
- For Google Colab installs, scanpy installs a newer matplotlib than the default and the runtime may need a restart after installation

## Evidence

- [methods] Convert sparse array data to dense array: "To convert, merely do `adata.X = adata.X.toarray()`"
- [readme] scanpy/numpy data handling in MarkerMap: "Data is handled with numpy and with scanpy which is used for many computational biology datasets."
- [readme] Reading the CITE-seq AnnData object: "We then read in the data using scanpy which returns an AnnData object. The gene data is stored in adata.X"
