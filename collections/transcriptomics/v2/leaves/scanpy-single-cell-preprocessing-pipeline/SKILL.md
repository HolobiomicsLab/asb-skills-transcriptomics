---
name: scanpy-single-cell-preprocessing-pipeline
description: Use when raw single-cell gene expression count matrices (e.g., MCA or
  PBMC h5ad/count matrices) must be normalized, reduced to highly variable genes,
  and converted into affinity matrices or low-dimensional embeddings before clustering.
when_to_use_negative:
- Input is already a normalized, feature-selected, dimensionality-reduced embedding
  (e.g., a precomputed UMAP/PCA matrix) rather than a raw count matrix
- The downstream method requires nonlinear dimensionality reduction that you have
  not yet computed — applying PCA-only affinities to Forest Fire Clustering violates
  its low-dimensional fire-propagation assumption
- Data are not single-cell expression data (e.g., bulk RNA-seq or non-genomic feature
  tables) where the zheng17 recipe's assumptions do not apply
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scanpy
  role: Preprocesses count matrices with the 'zheng17' recipe, selects highly variable
    genes, and provides 'sc.pp.neighbors' used in the FFC preprocessing step to obtain
    the affinity matrix from the data matrix
- name: numpy
  role: Used for linear algebra operations during preprocessing and clustering
- name: scipy
  role: Used for sparse matrix acceleration of the affinity computations
- name: UMAP
  role: Recommended nonlinear dimensionality reduction to apply before clustering
    methods such as Forest Fire Clustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:88cc763a2801dd2a4b83290637dad052772be1b6931924249d038887809f6e9c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - UMAP
  - numba
  - numpy
  - pegasus
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# scanpy-single-cell-preprocessing-pipeline

## Summary

Preprocess single-cell RNA-seq count matrices with scanpy — using the 'zheng17' recipe, selecting highly variable genes (e.g., top 5000 for MCA), and computing neighbors/affinities via 'sc.pp.neighbors' — to produce PCA-reduced, KNN-affinity inputs suitable for downstream clustering such as Forest Fire Clustering.

## When to use

Use when raw single-cell gene expression count matrices (e.g., MCA or PBMC h5ad/count matrices) must be normalized, reduced to highly variable genes, and converted into affinity matrices or low-dimensional embeddings before clustering.

## When NOT to use

- Input is already a normalized, feature-selected, dimensionality-reduced embedding (e.g., a precomputed UMAP/PCA matrix) rather than a raw count matrix
- The downstream method requires nonlinear dimensionality reduction that you have not yet computed — applying PCA-only affinities to Forest Fire Clustering violates its low-dimensional fire-propagation assumption
- Data are not single-cell expression data (e.g., bulk RNA-seq or non-genomic feature tables) where the zheng17 recipe's assumptions do not apply

## Inputs

- Single-cell gene expression count matrix (e.g., MCA or PBMC scRNA-seq count data, scanpy AnnData)
- Choice of preprocessing recipe (scanpy 'zheng17')
- Number of highly variable genes/peaks to retain (e.g., 5000 HVGs for MCA, 10,000 ATAC peaks)
- Number of principal components for clustering (first 30)

## Outputs

- Normalized and standardized expression matrix restricted to top highly variable genes
- First 30 principal components for clustering
- KNN neighbor graph / affinity matrix A produced via sc.pp.neighbors
- Optionally, differential expression tables from scanpy rank_genes_groups (Wilcoxon + Bonferroni)

## How to apply

Load the gene expression count matrix and preprocess it with the scanpy 'zheng17' recipe, which normalizes and standardizes the data. Select highly variable genes — the MCA benchmark uses the top 5000, while scATAC uses the top 10,000 highly variable peaks — then take the first 30 principal components for clustering. Compute the neighborhood graph with scanpy's 'sc.pp.neighbors', which Forest Fire Clustering's preprocess step calls directly to obtain the affinity matrix A from the data matrix W; this approximates the L2 distance matrix via KNN distances and converts distances into affinities with an adaptive kernel. Note that Forest Fire Clustering should subsequently be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE) because fire propagation mimics a low-dimensional process. Differential expression on the resulting clusters can be extracted with scanpy's 'rank_genes_groups' using the Wilcoxon test with Bonferroni corrections.

## Related tools

- **scanpy** (Preprocesses count matrices with the 'zheng17' recipe, selects highly variable genes, and provides 'sc.pp.neighbors' used in the FFC preprocessing step to obtain the affinity matrix from the data matrix)
- **numpy** (Used for linear algebra operations during preprocessing and clustering)
- **scipy** (Used for sparse matrix acceleration of the affinity computations)
- **UMAP** (Recommended nonlinear dimensionality reduction to apply before clustering methods such as Forest Fire Clustering)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('mca.h5ad'); sc.pp.recipe_zheng17(adata, n_top_genes=5000); sc.pp.pca(adata, n_comps=30); sc.pp.neighbors(adata, n_pcs=30)
```

## Evaluation signals

- The count matrix was preprocessed with the scanpy 'zheng17' recipe and normalized/standardized before further analysis
- Exactly the top 5000 highly variable genes (MCA) or 10,000 ATAC peaks are retained, and only the first 30 principal components feed clustering
- The affinity matrix A passed to downstream clustering is derived from sc.pp.neighbors, matching the article's KNN-approximated L2 distance converted to affinities with an adaptive kernel
- Downstream clustering quality (e.g., ARI against tissue-of-origin labels) is computed on the processed embedding and shows sensible cluster structure (e.g., FFC ARI ≈ 0.38 unfiltered on MCA)
- Nonlinear dimensionality reduction (UMAP/t-SNE/PHATE) has been applied before clustering, per the method's stated requirement

## Limitations

- Curse of dimensionality: clustering degrades as distances become more uniform in high dimension, hence the need for PCA and nonlinear embeddings
- Forest Fire Clustering is sensitive to high dimensionality because it mimics a low-dimensional fire propagation process; it must be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)
- Representing the full distance matrix for datasets over 1 million data points is computationally expensive and must be approximated via KNN

## Evidence

- [discussion] The gene expression count matrix for MCA was also preprocessed using the “zheng17” recipe in the scanpy package: "The gene expression count matrix for MCA was also preprocessed using the “zheng17” recipe in the scanpy package"
- [discussion] After normalization and standardization, the first 30 principal components were used for clustering.: "After normalization and standardization, the first 30 principal components were used for clustering."
- [discussion] we directly use this function in our preprocessing step to obtain the affinity matrix A from the data matrix W.: "we directly use this function in our preprocessing step to obtain the affinity matrix A from the data matrix W."
- [discussion] the scanpy package in Python has a highly efficient and stable function called “sc.pp.neighbors”: "the scanpy package in Python has a highly efficient and stable function called “sc.pp.neighbors”"
- [discussion] Hence, Forest Fire Clustering should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE): "Hence, Forest Fire Clustering should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)"
- [methods] from forest_fire_clustering.forest_fire_clustering import FFC
cluster_obj = FFC()
cluster_obj.preprocess(X): "from forest_fire_clustering.forest_fire_clustering import FFC"
