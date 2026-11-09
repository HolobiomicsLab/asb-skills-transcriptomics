---
name: knn-affinity-graph-construction
description: Use when you need a cell-to-cell affinity graph for label-propagation
  clustering of single-cell data (e.g. PCA-reduced PBMC expression or ATAC peak matrices)
  before running FFC, or when propagating labels to out-of-sample points via a joint
  affinity matrix.
when_to_use_negative:
- The input is raw, unnormalized count data — the article specifies preprocessing
  (zheng17 recipe, PCA) before affinity construction.
- The data are extremely high-dimensional without nonlinear dimensionality reduction;
  the method is sensitive to high dimensionality and limited by the curse of dimensionality.
- You need exact all-pairs distances for >1 million points; the article notes representing
  such distance matrices is computationally expensive and requires the KNN approximation.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Provides sc.pp.neighbors for the KNN search combined with an adaptive kernel
    used to obtain the affinity matrix A from the data matrix W.
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scipy
  role: Used for sparse matrix acceleration of the affinity/distance computations.
- name: numba
  role: Used for numerical acceleration of preprocessing and propagation routines.
- name: numpy
  role: Used for linear algebra operations when constructing and manipulating the
    distance/affinity matrices.
- name: forest-fire-clustering (FFC)
  role: Wraps affinity construction in its preprocess(X) step; the resulting graph
    feeds fit(fire_temp=100) label propagation.
  repo: https://github.com/gersteinlab/forest-fire-clustering
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:afecda74c17725178840565a61f096fca40c7242bdb01a703214e57c4096ca1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - forest-fire-clustering (FFC)
  - numba
  - numpy
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# knn-affinity-graph-construction

## Summary

Builds the [sample x sample] affinity matrix A that Forest Fire Clustering propagates labels over, by approximating the full L2 distance matrix with KNN distances (via scanpy's sc.pp.neighbors with an adaptive kernel) and converting distances to affinities. This is the preprocessing step that determines the graph structure on which all subsequent label propagation, Monte Carlo validation, and online inference operate.

## When to use

Use when you need a cell-to-cell affinity graph for label-propagation clustering of single-cell data (e.g. PCA-reduced PBMC expression or ATAC peak matrices) before running FFC, or when propagating labels to out-of-sample points via a joint affinity matrix.

## When NOT to use

- The input is raw, unnormalized count data — the article specifies preprocessing (zheng17 recipe, PCA) before affinity construction.
- The data are extremely high-dimensional without nonlinear dimensionality reduction; the method is sensitive to high dimensionality and limited by the curse of dimensionality.
- You need exact all-pairs distances for >1 million points; the article notes representing such distance matrices is computationally expensive and requires the KNN approximation.

## Inputs

- [sample x feature] matrix X (e.g. scanpy zheng17-preprocessed PBMC RNA counts reduced to 30 PCs)
- ATAC peak matrix with top 10,000 highly variable peaks for PCA (multiome)
- Projected out-of-sample testing data (after mutual nearest neighbor batch correction, for joint affinity construction)

## Outputs

- Affinity matrix A ([sample x sample]) used for label propagation
- Fitted FFC object with per-vertex degrees Di defining acceptance thresholds Ti = 1/Di
- Joint affinity matrix of training and testing data (online setting)

## How to apply

Start from a preprocessed [sample x feature] matrix X — e.g. PBMC gene expression preprocessed with scanpy's 'zheng17' recipe and reduced to the first 30 principal components, or the multiome data with the top 10,000 highly variable ATAC peaks used for PCA. Compute the KNN graph with scanpy's sc.pp.neighbors and an adaptive kernel, which approximates the full L2 distance matrix by using distances between the KNNs of each data point, then converts the distances into affinities to yield the affinity matrix A. In the FFC API this is done via cluster_obj.preprocess(X), which sets up the affinity structure and per-vertex degrees Di used to define acceptance thresholds Ti = 1/Di during fit (e.g. with fire_temp=100). For online/out-of-sample inference, project the testing data onto the training low-dimensional representation, apply mutual nearest neighbor batch correction, and compute a joint affinity matrix of training and testing points. Note the method is sensitive to high dimensionality — apply nonlinear dimensionality reduction (e.g. UMAP, t-SNE, PHATE) when needed, and for very large datasets (>1 million points) rely on the KNN approximation rather than an explicit distance matrix.

## Related tools

- **scanpy** (Provides sc.pp.neighbors for the KNN search combined with an adaptive kernel used to obtain the affinity matrix A from the data matrix W.) — https://github.com/gersteinlab/forest-fire-clustering
- **scipy** (Used for sparse matrix acceleration of the affinity/distance computations.)
- **numba** (Used for numerical acceleration of preprocessing and propagation routines.)
- **numpy** (Used for linear algebra operations when constructing and manipulating the distance/affinity matrices.)
- **forest-fire-clustering (FFC)** (Wraps affinity construction in its preprocess(X) step; the resulting graph feeds fit(fire_temp=100) label propagation.) — https://github.com/gersteinlab/forest-fire-clustering

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100)
```

## Evaluation signals

- Affinity matrix A is square with shape [n_samples x n_samples] derived from the KNN structure of X.
- After preprocess(X) followed by fit(fire_temp=100), every data point receives exactly one cluster label in cluster_obj.cluster_labels.
- On the PBMC benchmark (30 PCs, zheng17 preprocessing), clusters evaluated by ARI/purity against 10x Genomics marker-gene reference labels should outperform Louvain/Leiden/PARC baselines, with ARI improving >20% after PEP < 0.1 filtering via pval_list.
- For online extension, propagated labels to out-of-sample points should yield average ARI and purity both > 0.90 as unseen-cluster count and Gaussian noise increase.
- Computation on large datasets stays tractable because the L2 distance matrix is approximated via KNN distances rather than computed exhaustively.

## Limitations

- Sensitive to high dimensionality because the approach mimics a low-dimensional fire propagation process; should be performed with nonlinear dimensionality reduction (e.g. UMAP, t-SNE, PHATE).
- Limited by the curse of dimensionality — clustering degrades as distances become more uniform.
- Representing the full distance matrix for datasets over 1 million points is computationally expensive and must be approximated via KNN distances.
- The fire temperature c strongly influences the size and number of discovered clusters (resolution-like sensitivity), so affinity-based clustering results depend on this parameter choice.

## Evidence

- [discussion] KNN distance approximation and kernel conversion: [evidence span withheld — anchor status: not_found]
- [discussion] scanpy neighbors used to obtain A: "we directly use this function in our preprocessing step to obtain the affinity matrix A from the data matrix W."
- [discussion] scanpy sc.pp.neighbors recommendation: "the scanpy package in Python has a highly efficient and stable function called “sc.pp.neighbors”"
- [methods] FFC preprocess API call: "from forest_fire_clustering.forest_fire_clustering import FFC
cluster_obj = FFC()
cluster_obj.preprocess(X)"
- [discussion] Nonlinear dimensionality reduction requirement: "Hence, Forest Fire Clustering should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)"
