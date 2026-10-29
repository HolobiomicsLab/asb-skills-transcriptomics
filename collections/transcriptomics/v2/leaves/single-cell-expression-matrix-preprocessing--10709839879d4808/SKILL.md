---
name: single-cell-expression-matrix-preprocessing
description: Use when a sample-by-feature single-cell expression matrix (e.g., PBMC
  or MCA gene expression counts in a .h5ad-like matrix X) must be normalized, reduced
  to ~30 PCs, and converted to a KNN affinity graph before clustering.
when_to_use_negative:
- Input is already a precomputed affinity/adjacency graph rather than a sample-by-feature
  expression matrix
- Data dimensionality is very high (>1 million points) without KNN distance approximation
  — the full distance matrix becomes computationally expensive
- Only linear structure is expected — the article states clustering should be performed
  with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: scanpy
  role: Provides the 'sc.pp.neighbors' KNN search and adaptive kernel used in the
    preprocessing step to obtain the affinity matrix A; 'zheng17' recipe preprocesses
    PBMC counts
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: numpy
  role: Linear algebra operations for normalization, standardization, and PCA computation
- name: scipy
  role: Sparse matrix acceleration for the affinity graph
- name: numba
  role: Numerical acceleration of preprocessing computations
- name: forest-fire-clustering
  role: Receives the preprocessed matrix X via cluster_obj.preprocess(X) to initialize
    the clustering fit
  repo: https://github.com/gersteinlab/forest-fire-clustering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:e99c4ba94aa88e7803afe95ad3c3f1a5f7be5f30a019416a9fd8dc31d5a994e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - forest-fire-clustering
  - numba
  - numpy
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# single-cell-expression-matrix-preprocessing

## Summary

Preprocess a sample-by-feature single-cell expression matrix (e.g., PBMC gene expression preprocessed with scanpy's 'zheng17' recipe, reduced to the first 30 principal components) and compute the affinity matrix A via scanpy's sc.pp.neighbors KNN search with an adaptive kernel, as the prerequisite stage of the Forest Fire Clustering pipeline.

## When to use

Use when a sample-by-feature single-cell expression matrix (e.g., PBMC or MCA gene expression counts in a .h5ad-like matrix X) must be normalized, reduced to ~30 PCs, and converted to a KNN affinity graph before clustering.

## When NOT to use

- Input is already a precomputed affinity/adjacency graph rather than a sample-by-feature expression matrix
- Data dimensionality is very high (>1 million points) without KNN distance approximation — the full distance matrix becomes computationally expensive
- Only linear structure is expected — the article states clustering should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)

## Inputs

- Sample-by-feature gene expression count matrix X (e.g., PBMC or MCA scRNA-seq counts)
- scanpy AnnData object preprocessed with the 'zheng17' recipe
- Optional: top 5000 highly variable genes for MCA; top 10,000 highly variable ATAC peaks for scATAC-seq PCA

## Outputs

- Affinity matrix A derived from the KNN-approximated L2 distances and adaptive kernel
- First 30 principal components used as the clustering embedding
- Fitted FFC object ready for cluster_obj.fit(fire_temp=100)

## How to apply

Load the preprocessed sample-by-feature matrix X (e.g., PBMC gene expression count data preprocessed using the 'zheng17' recipe in the scanpy package, or MCA counts with the top 5000 highly variable genes selected). After normalization and standardization, retain the first 30 principal components for clustering (or the top 10,000 highly variable ATAC peaks for PCA in the scATAC-seq variant). Compute the L2 distance matrix approximated by using the L2 distance between the KNNs of each data point, then convert the distances into affinities with an adaptive kernel. Use scanpy's 'sc.pp.neighbors' function in the preprocessing step to obtain the affinity matrix A from the data matrix W, which becomes the propagation graph for FFC's preprocess(X) call.

## Related tools

- **scanpy** (Provides the 'sc.pp.neighbors' KNN search and adaptive kernel used in the preprocessing step to obtain the affinity matrix A; 'zheng17' recipe preprocesses PBMC counts) — https://github.com/gersteinlab/forest-fire-clustering
- **numpy** (Linear algebra operations for normalization, standardization, and PCA computation)
- **scipy** (Sparse matrix acceleration for the affinity graph)
- **numba** (Numerical acceleration of preprocessing computations)
- **forest-fire-clustering** (Receives the preprocessed matrix X via cluster_obj.preprocess(X) to initialize the clustering fit) — https://github.com/gersteinlab/forest-fire-clustering

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X)
```

## Evaluation signals

- Every data point in the downstream clustering receives exactly one cluster label (all vertices of the affinity graph are labeled)
- The affinity matrix A is sparse and derived from KNN-approximated L2 distances, not a full dense distance matrix
- Exactly 30 principal components are retained for clustering after normalization and standardization
- The cluster_labels.csv output contains one row per input sample
- The pipeline progresses cleanly from preprocess(X) to fit(fire_temp=100) without affinity-matrix errors

## Limitations

- Representing the distance matrix for large datasets (>1 million data points) is computationally expensive and must be approximated via KNN
- Forest Fire Clustering is limited by the curse of dimensionality, as distances become more uniform at high dimension
- The method is sensitive to high dimensionality because it mimics a low-dimensional fire propagation process, hence it should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)

## Evidence

- [discussion] The RNA gene expression count data for both PBMC datasets were preprocessed using the 'zheng17' recipe in the scanpy package: [evidence span withheld — anchor status: not_found]
- [discussion] First 30 principal components used for clustering: "After normalization and standardization, the first 30 principal components were used for clustering."
- [discussion] KNN-based distance approximation and adaptive kernel affinity: [evidence span withheld — anchor status: not_found]
- [discussion] scanpy neighbors function for affinity matrix: "we directly use this function in our preprocessing step to obtain the affinity matrix A from the data matrix W."
- [readme] README execution snippet: "cluster_obj = FFC()
cluster_obj.preprocess(X)
cluster_obj.fit(fire_temp=100)"
