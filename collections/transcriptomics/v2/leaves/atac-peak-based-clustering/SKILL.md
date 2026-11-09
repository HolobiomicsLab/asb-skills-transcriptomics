---
name: atac-peak-based-clustering
description: Use when clustering single-cell multiome PBMC data where the ATAC modality
  must be reduced to its most variable peaks before PCA, and high-confidence cluster
  labels (PEP < 0.1) are needed for downstream ARI evaluation.
when_to_use_negative:
- Data contains only one cluster — Monte Carlo internal validation cannot accurately
  estimate uncertainty when every data point receives one label
- The input is raw high-dimensional ATAC counts without prior peak selection — Forest
  Fire Clustering is sensitive to high dimensionality and should be performed with
  nonlinear or reduced dimensionality representations
edam_operation: http://edamontology.org/operation_3432
tools:
- name: Forest Fire Clustering (FFC)
  role: 'Core clustering method: preprocess(X) builds the affinity matrix, fit(fire_temp=100)
    assigns labels, pval_list enables PEP-based label filtering'
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scanpy
  role: Preprocessing via the zheng17 recipe and sc.pp.neighbors for KNN affinity
    matrix construction from the reduced embeddings
- name: scikit-learn (sklearn)
  role: Baseline clustering comparison and adjusted Rand index / purity computation
    against reference labels
- name: numpy
  role: Linear algebra operations for PCA and matrix manipulation
- name: numba
  role: Numerical acceleration for parallelized Monte Carlo validation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:f4a8a93256a1e6b355966683c896dda8ded225f910e2bcdc60e10f4f63142ea7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Forest Fire Clustering (FFC)
  - Python
  - numba
  - numpy
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# atac-peak-based-clustering

## Summary

Cluster PBMC single-cell multiome data by selecting the top 10,000 highly variable ATAC peaks for PCA, then applying Forest Fire Clustering with fire_temp=100 and validating label confidence via posterior significance (PEP) values.

## When to use

Use when clustering single-cell multiome PBMC data where the ATAC modality must be reduced to its most variable peaks before PCA, and high-confidence cluster labels (PEP < 0.1) are needed for downstream ARI evaluation.

## When NOT to use

- Data contains only one cluster — Monte Carlo internal validation cannot accurately estimate uncertainty when every data point receives one label
- The input is raw high-dimensional ATAC counts without prior peak selection — Forest Fire Clustering is sensitive to high dimensionality and should be performed with nonlinear or reduced dimensionality representations

## Inputs

- [sample x feature] RNA/ATAC count matrix from a PBMC multiome dataset
- Top 10,000 highly variable ATAC peaks selected for PCA
- First 30 principal components used for clustering
- Marker-gene-derived multiome reference cell-type labels

## Outputs

- cluster_obj.cluster_labels per cell
- cluster_obj.entropy_list (per-cell label entropies)
- cluster_obj.pval_list (per-cell posterior significance / PEP values)
- ARI and purity scores, pre- and post-PEP<0.1 filtering, assembled into ari_purity_benchmark.csv

## How to apply

Start from the [sample x feature] PBMC multiome matrix and select the top 10,000 highly variable ATAC peaks as features for PCA. Feed the reduced [sample x feature] matrix into Forest Fire Clustering: instantiate FFC(), call preprocess(X), then fit(fire_temp=100) to obtain cluster_labels. Run internal validation with validate(), entropy(), and pval() to compute label entropies and posterior significance values (pval_list). Filter cells to high-confidence labels using PEP < 0.1 from cluster_obj.pval_list and recompute adjusted Rand index and purity against marker-gene-derived multiome reference labels to quantify the ARI gain (>20%) from label filtering.

## Related tools

- **Forest Fire Clustering (FFC)** (Core clustering method: preprocess(X) builds the affinity matrix, fit(fire_temp=100) assigns labels, pval_list enables PEP-based label filtering) — https://github.com/gersteinlab/forest-fire-clustering
- **scanpy** (Preprocessing via the zheng17 recipe and sc.pp.neighbors for KNN affinity matrix construction from the reduced embeddings)
- **scikit-learn (sklearn)** (Baseline clustering comparison and adjusted Rand index / purity computation against reference labels)
- **numpy** (Linear algebra operations for PCA and matrix manipulation)
- **numba** (Numerical acceleration for parallelized Monte Carlo validation)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.validate(); cluster_obj.pval()
```

## Evaluation signals

- Filtering cells to PEP < 0.1 via cluster_obj.pval_list should improve cluster ARI by more than 20% compared to Louvain, Leiden, and PARC baselines
- Adjusted Rand index and purity against marker-gene reference labels should be computed for every method x dataset combination
- cluster_obj.pval_list and cluster_obj.entropy_list must be populated after calling validate(), entropy(), and pval()
- ATAC feature space must contain exactly the top 10,000 highly variable peaks, and PCA should retain the first 30 principal components
- Results should reproduce the benchmark outputs assembled in ari_purity_benchmark.csv

## Limitations

- Forest Fire Clustering is limited by the curse of dimensionality; distances become more uniform as dimensionality grows
- The fire temperature c strongly influences the size and number of discovered clusters (resolution-like sensitivity)
- The number of Monte Carlo trials varies with the number of clusters and is not fixed, affecting reproducibility
- Large numbers of Monte Carlo trials are computationally intensive, though parallelizable

## Evidence

- [discussion] Top 10,000 highly variable ATAC peaks selected for PCA: "the top 10,000 highly variable ATAC peaks were selected for PCA"
- [discussion] First 30 principal components used for clustering: "After normalization and standardization, the first 30 principal components were used for clustering."
- [methods] FFC preprocess/fit invocation pattern: "from forest_fire_clustering.forest_fire_clustering import FFC
cluster_obj = FFC()
cluster_obj.preprocess(X)"
- [methods] fit with fire_temp=100: "cluster_obj.fit(fire_temp=100)"
- [readme] posterior significance values used for label-confidence filtering: "cluster_obj.pval_list # list of posterior significance values"
- [discussion] benchmark evaluation metrics: "There are three main ways to evaluate the performance of unsupervised methods. First, purity scores evaluate the cluster labels based on a set of reference labels."
