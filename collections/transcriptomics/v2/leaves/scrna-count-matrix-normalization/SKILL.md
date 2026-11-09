---
name: scrna-count-matrix-normalization
description: Use when raw scRNA-seq/CITE-seq counts (e.g., a sparse adata.X from a
  .h5ad file) must be converted into a dense, normalized, scaled matrix before training
  a marker-selection or generative model.
when_to_use_negative:
- The input matrix is already normalized, log-transformed, and scaled (e.g., a preprocessed_adata.h5ad)
  — reapplying filters/scaling would distort the data
- The model accepts sparse input directly, so dense conversion is unnecessary or memory-prohibitive
  for very large datasets
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Reads the .h5ad file into an AnnData object whose adata.X count matrix is
    preprocessed
- name: numpy
  role: Handles the count matrix data and applies the log2(1+x) transformation to
    normalized counts
- name: anndata
  role: Container format storing the preprocessed gene-by-cell matrix and cell labels
    for downstream MarkerMap steps
- name: MarkerMap
  role: Downstream model that expects the dense, preprocessed matrix; its quick-start
    workflow defines the preprocessing requirements
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:0de4f81f73782857b284333bcbf49f1a252a5c860080b9a685e9b7f23a13cd11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
  tools:
  - MarkerMap
  - anndata
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# scrna-count-matrix-normalization

## Summary

Preprocess raw single-cell RNA-seq count matrices (e.g., CITE-seq .h5ad data) so they are suitable for marker selection models like MarkerMap: drop low-information/housekeeping genes, normalize per-cell library size, apply log2(1+x), and z-scale per gene.

## When to use

Use when raw scRNA-seq/CITE-seq counts (e.g., a sparse adata.X from a .h5ad file) must be converted into a dense, normalized, scaled matrix before training a marker-selection or generative model.

## When NOT to use

- The input matrix is already normalized, log-transformed, and scaled (e.g., a preprocessed_adata.h5ad) — reapplying filters/scaling would distort the data
- The model accepts sparse input directly, so dense conversion is unnecessary or memory-prohibitive for very large datasets

## Inputs

- Raw scRNA-seq or CITE-seq count matrix as AnnData (.h5ad), e.g., data/cite_seq/CITEseq.h5ad
- Sparse gene-by-cell count matrix in adata.X (converted to dense via toarray())
- Cell labels in adata.obs (e.g., 'names' copied to 'annotation') used to set group_by

## Outputs

- Dense, library-normalized, log2(1+x)-transformed, z-scaled gene expression matrix in an AnnData object
- Saved preprocessed AnnData file, e.g., preprocessed_adata.h5ad

## How to apply

Load the data with scanpy into an AnnData object (e.g., data/cite_seq/CITEseq.h5ad) and convert adata.X from sparse to dense with toarray(), since MarkerMap VAE models expect dense input. Remove general/housekeeping genes and apply presence/absence gene-fraction filters, dropping genes expressed in too few or too many cells. Normalize counts per cell so each cell has a comparable library total, then apply the log2(1+x) transform with numpy, and finally center and scale each gene to zero mean and unit variance. Save the result as an AnnData object (e.g., preprocessed_adata.h5ad) so the downstream train/validation/test split (70%/10%/20%) and MarkerMap.prepareData operate on comparable, scaled values.

## Related tools

- **scanpy** (Reads the .h5ad file into an AnnData object whose adata.X count matrix is preprocessed)
- **numpy** (Handles the count matrix data and applies the log2(1+x) transformation to normalized counts)
- **anndata** (Container format storing the preprocessed gene-by-cell matrix and cell labels for downstream MarkerMap steps)
- **MarkerMap** (Downstream model that expects the dense, preprocessed matrix; its quick-start workflow defines the preprocessing requirements) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('data/cite_seq/CITEseq.h5ad'); adata.X = adata.X.toarray(); adata.write('preprocessed_adata.h5ad')
```

## Evaluation signals

- adata.X is a dense ndarray after calling toarray() (no sparse dtype remains)
- Each cell's total counts are comparable after per-cell library-size normalization
- After z-scaling, each retained gene has approximately zero mean and unit variance across cells
- Genes expressed in too few or too many cells (and housekeeping genes) are absent from the preprocessed matrix
- The preprocessed AnnData loads correctly via sc.read_h5ad('preprocessed_adata.h5ad') and matches the expected gene dimension for MarkerMap (e.g., adata.shape[1] used as input_size)

## Limitations

- Dense conversion (adata.X.toarray()) can be memory-intensive for large datasets since MarkerMap models expect dense input
- Dataset-specific cleaning may be needed (e.g., 'removing outliers and normalizing the data in the case of Mouse Brain'), so a single preprocessing recipe may not transfer across the CITE-seq, Mouse Brain, Paul, and Zeisel datasets
- Google Colab users may need to restart the runtime after installing because scanpy installs a newer matplotlib than the default

## Evidence

- [methods] To convert, merely do `adata.X = adata.X.toarray()`: "To convert, merely do `adata.X = adata.X.toarray()`"
- [methods] Data is handled with numpy and with scanpy which is used for many computational biology datasets.: "Data is handled with numpy and with scanpy which is used for many computational biology datasets."
- [readme] Set file_path to wherever your data is located. We then read in the data using scanpy which returns an AnnData object.: "We then read in the data using scanpy which returns an AnnData object. The gene data is stored in adata.X, and the cell labels are stored in adata.obs['names']."
- [readme] We then split the data into training, validation, and test sets with a 70%, 10%, 20% split.: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [other] The preprocessing step removes some genes and then applies scaling, normalization, and a log(1+X) transform to the remaining genes before marker selection.: [evidence span withheld — anchor status: not_found]
