---
name: kmd-hierarchical-clustering
description: Use when clustering simulated, mass cytometry, or scRNA-seq data where
  general-purpose methods fail on non-globular or noisy clusters and you want automatic
  k selection rather than user-specified cryptic hyperparameters.
when_to_use_negative:
- 'Runtime is critical: a single KMD run maintains lists of k minimal distances for
  each pair of clusters and the python/scipy implementation is slower than optimized
  non-interpreted implementations of widely used clustering algorithms'
- The data contain very small clusters that a sampling-based subsample would not represent
  — the sampling extension may miss clusters that are not sampled well
- 'The standard silhouette is sufficient: KMD silhouette computation requires scanning
  a range of k values (k < 100 in all paper cases), consuming more computational resources
  than a single fixed-k run'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: KMDHierarchicalClustering
  role: 'Main implementation: provides the KMDClustering class with scikit-learn-style
    fit/predict, KMD silhouette-based k selection, outlier-aware partitioning and
    sub-sampling'
  repo: https://github.com/KaplanLab/KMDHierarchicalClustering
- name: hkariti/DBCV
  role: Density-based clustering validation used to estimate the number of clusters
    when it is not given a priori, computed after outlier removal
  repo: https://github.com/hkariti/DBCV
- name: scikit-learn
  role: Generates simulated benchmark datasets (make_moons, concentric circles, Gaussian
    blobs) with retained ground-truth labels and provides baseline AgglomerativeClustering
    single linkage
- name: SciPy
  role: Supplies the hierarchical-clustering framework and scipy.spatial.distance
    metrics used by KMD linkage; also runs complete and Ward linkage baselines via
    cluster.hierarchy.linkage
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:7add9fe63e085d207eb2658c3f826a60d6cf32ea9104a00cefe57dca5621e182
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KaplanLab/KMDHierarchicalClustering
  tools:
  - Bluster
  - FlowSOM
  - KMDHierarchicalClustering
  - PhenoGraph
  - Python
  - SCCAF
  - Scanpy
  - SciPy
  - Seurat
  - hdbscan
  - hkariti/DBCV
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# kmd-hierarchical-clustering

## Summary

Apply KMD (k-minimal distances) clustering — a generalization of single and average linkage hierarchical clustering — to datasets with known or unknown cluster structure, using the KMD silhouette to auto-select the hyperparameter k, an outlier-aware partitioning scheme, and optional DBCV-based cluster-number estimation.

## When to use

Use when clustering simulated, mass cytometry, or scRNA-seq data where general-purpose methods fail on non-globular or noisy clusters and you want automatic k selection rather than user-specified cryptic hyperparameters.

## When NOT to use

- Runtime is critical: a single KMD run maintains lists of k minimal distances for each pair of clusters and the python/scipy implementation is slower than optimized non-interpreted implementations of widely used clustering algorithms
- The data contain very small clusters that a sampling-based subsample would not represent — the sampling extension may miss clusters that are not sampled well
- The standard silhouette is sufficient: KMD silhouette computation requires scanning a range of k values (k < 100 in all paper cases), consuming more computational resources than a single fixed-k run

## Inputs

- Dataset matrix X (numpy array) of features per object, e.g. scikit-learn generated toy data, mass cytometry (CyTOF) marker intensities, or log1p-normalized scRNA-seq count matrices
- Precomputed adjacency/distance matrix (affinity='precomputed')
- Optional ground-truth labels y_true for evaluation scenarios

## Outputs

- Cluster label predictions per object (kmd_cluster.predict(X)), with outliers marked before certainty-based re-assignment
- Per-k KMD silhouette scores (and accuracy when y_true is given), optionally plotted when plot_scores=True
- Benchmark metric tables of accuracy/NMI/ARI vs. competing algorithms (e.g. toy_dataset_clustering_scores.csv, benchmark_performance_table.csv)

## How to apply

Install the package from GitHub (pip install git+https://github.com/KaplanLab/KMDHierarchicalClustering) and instantiate KMDClustering with k='compute', min_cluster_size='compute', k_scan_range=(1,100,3), affinity='euclidean' (or 'correlation' for scRNA-seq count matrices). Fit the model on the dataset matrix X, letting the algorithm run clustering in parallel over the k scan range and pick the k with the highest KMD silhouette score — a generalized silhouette-like function introduced because the standard silhouette is biased towards globular cluster shapes. Clusters smaller than min_cluster_size are treated as outliers; at the optimal k, outliers are re-assigned to core clusters based on KMD linkage (controlled by the certainty parameter, where 0.5 leaves all outliers unassigned). If the number of clusters is unknown, compute DBCV (hkariti/DBCV implementation) after outlier removal for each candidate n_clusters and choose the cluster number that maximizes the DBCV score. For datasets with millions of objects, call fit with sub_sample=True and percent_size≈0.2, then use the post hoc cluster-inferring method to label the remaining objects.

## Related tools

- **KMDHierarchicalClustering** (Main implementation: provides the KMDClustering class with scikit-learn-style fit/predict, KMD silhouette-based k selection, outlier-aware partitioning and sub-sampling) — https://github.com/KaplanLab/KMDHierarchicalClustering
- **hkariti/DBCV** (Density-based clustering validation used to estimate the number of clusters when it is not given a priori, computed after outlier removal) — https://github.com/hkariti/DBCV
- **scikit-learn** (Generates simulated benchmark datasets (make_moons, concentric circles, Gaussian blobs) with retained ground-truth labels and provides baseline AgglomerativeClustering single linkage)
- **SciPy** (Supplies the hierarchical-clustering framework and scipy.spatial.distance metrics used by KMD linkage; also runs complete and Ward linkage baselines via cluster.hierarchy.linkage)

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd_cluster = KMDClustering(n_clusters=2, k='compute', k_scan_range=(1,100,3)); kmd_cluster.fit(X); y = kmd_cluster.predict(X)
```

## Evaluation signals

- On the nested-circles toy dataset (factor=0.3, noise=0.05, 1000 objects, seed=1), KMD must attain accuracy/NMI/ARI = 1, matching Supplementary Table S1
- On high-noise datasets (noise=0.14/0.24, seed=185), accuracies must match the reported 0.989 (nested circles), 0.933 (half moons), 0.909 (globular), 0.992 (anisotropic), and KMD should be the only method performing well on all four problems (Tables S2–S4)
- KMD silhouette must be predictive of clustering quality: median Pearson correlation across 50 high-noise half-moon instances of ~0.89 (accuracy), ~0.85 (NMI), ~0.87 (ARI)
- On gold-standard scRNA-seq data (Lawlor17 GSE86469, affinity='correlation', min_cluster_size=10), KMD accuracy should reach ~0.893 vs. 0.808 for SCCAF; on Li17 (GSE81861) ~0.838 vs. 0.729 for Scanpy Louvain
- Predicted labels are matched to ground truth before scoring, and rows for KMD core clusters only (pre-outlier-assignment) should be reported alongside full predictions

## Limitations

- Automatic k selection requires running the clustering over a large range of k values, requiring more computational resources
- Implementation in python/scipy results in longer running times than optimized, non-interpreted implementations
- A single run may be inherently slower than single/average linkage because it maintains, for each pair of clusters, a list of the k minimal distances
- The sampling-based extension may miss very small clusters that are not sampled well
- De novo cluster number estimation via DBCV is only preliminary; further research is needed to improve cluster number estimation
- No software version numbers are reported for KMDHierarchicalClustering or compared tools; README states it was tested on Python 3.6.5/3.7 and is not expected to work with Python 2.x

## Evidence

- [abstract] We present k minimal distance (KMD) clustering, a general-purpose method based on a generalization of single and average linkage hierarchical clustering.: "We present k minimal distance (KMD) clustering, a general-purpose method based on a generalization of single and average linkage hierarchical clustering."
- [discussion] running the clustering across a range of k values (in all cases we limited k < 100) and automatically selecting the solution with the highest KMD silhouette score: "running the clustering across a range of k values (in all cases we limited k < 100) and automatically selecting the solution with the highest KMD silhouette score"
- [discussion] we calculate the DBCV (after removing outliers) of the solution with the k value selected by the KMD silhouette, and choose the number of clusters maximizes the DBCV score: "we calculate the DBCV (after removing outliers) of the solution with the k value selected by the KMD silhouette, and choose the number of clusters maximizes the DBCV score"
- [readme] KMD = KMDClustering(k='compute', n_clusters = 2, min_cluster_size = 'compute', affinity = 'euclidean', certainty = 0.5 ,k_scan_range = (1,100,3), y_true = [], plot_scores=False, path=False): "KMD = KMDClustering(k='compute', n_clusters = 2, min_cluster_size = 'compute', affinity = 'euclidean', certainty = 0.5 ,k_scan_range = (1,100,3), y_true = [], plot_scores=False, path=False)"
- [discussion] a single run of KMD clustering may be inherently slower than some other approaches (such as single and average linkage clustering) as it requires maintaining for each pairs of clusters a list of the: "a single run of KMD clustering may be inherently slower than some other approaches (such as single and average linkage clustering) as it requires maintaining for each pairs of clusters a list of the"
