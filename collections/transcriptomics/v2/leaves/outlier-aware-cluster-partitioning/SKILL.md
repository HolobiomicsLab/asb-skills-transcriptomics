---
name: outlier-aware-cluster-partitioning
description: Use when clustering noisy datasets where small spurious clusters are
  expected and you need automatic outlier detection plus re-assignment of outliers
  to core clusters rather than discarding them.
when_to_use_negative:
- When the dataset contains genuine small clusters of biological interest — the min_cluster_size
  rule labels tiny clusters as outliers, and the sampling-based extension may miss
  very small clusters that are not sampled well
- When a simple silhouette-score-based partitioning would suffice — the article notes
  the silhouette score is biased towards globular cluster shapes and is not adequate
  as an intrinsic evaluation function, so plain silhouette partitioning should not
  substitute the KMD silhouette + outlier scheme
- When the input is a domain-specific feature representation better served by specialized
  pipelines (e.g. mass cytometry data already processed by FlowSOM/SCCAF workflows)
edam_operation: http://edamontology.org/operation_3432
tools:
- name: KMDHierarchicalClustering
  role: Python/scipy implementation of KMD clustering providing the KMDClustering
    class with min_cluster_size and certainty parameters for outlier-aware partitioning
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: hkariti/DBCV
  role: Optimized Python DBCV implementation used in the authors' workflow (e.g. for
    cluster-number estimation after outlier removal)
  repo: https://github.com/hkariti/DBCV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:4b2bf586bc84898245b808cecdf2041b8d3c7db674732e12a82baed4bcf8fe1d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KMDHierarchicalClustering
  - Python
  - SciPy
  - hdbscan
  - hkariti/DBCV
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# outlier-aware-cluster-partitioning

## Summary

Combines KMD hierarchical clustering with an outlier-aware partitioning scheme: clusters smaller than a min_cluster_size threshold are flagged as outliers and then re-assigned to core clusters via KMD linkage at the optimal k, improving performance on noisy datasets.

## When to use

Use when clustering noisy datasets where small spurious clusters are expected and you need automatic outlier detection plus re-assignment of outliers to core clusters rather than discarding them.

## When NOT to use

- When the dataset contains genuine small clusters of biological interest — the min_cluster_size rule labels tiny clusters as outliers, and the sampling-based extension may miss very small clusters that are not sampled well
- When a simple silhouette-score-based partitioning would suffice — the article notes the silhouette score is biased towards globular cluster shapes and is not adequate as an intrinsic evaluation function, so plain silhouette partitioning should not substitute the KMD silhouette + outlier scheme
- When the input is a domain-specific feature representation better served by specialized pipelines (e.g. mass cytometry data already processed by FlowSOM/SCCAF workflows)

## Inputs

- Dataset feature matrix X (numpy array, n objects x features; or precomputed adjacency matrix when affinity='precomputed')
- KMD clustering solution with k_scan_range, e.g. (1,100,3)
- min_cluster_size threshold ('compute' or integer)
- certainty threshold in [0.5, 1] for outlier-to-core assignment

## Outputs

- Cluster labels for core clusters plus re-assigned outlier labels
- KMD silhouette score per scanned k value used to pick the optimal k
- Metric table of accuracy, NMI, and ARI vs ground truth (e.g. toy_dataset_clustering_scores.csv)

## How to apply

After running KMD clustering across a k_scan_range and selecting the solution with the highest KMD silhouette score, treat any cluster smaller than min_cluster_size (set explicitly or with min_cluster_size='compute') as an outlier. Then re-assign outliers to core clusters using KMD linkage matching the optimal k value, controlled by the certainty parameter (0.5 leaves all outliers unassigned, 1 assigns all outliers to core clusters; default behavior in the toy-dataset benchmark re-assigns outliers at the optimal k). Rationale: tiny clusters below a size threshold are likely noise, but KMD linkage between an outlier and core clusters can still recover its correct membership, improving accuracy/NMI/ARI on noisy data. On the four scikit-learn simulated datasets (1000 objects, seed=1), this yields perfect scores (accuracy/NMI/ARI = 1) on nested circles and half moons.

## Related tools

- **KMDHierarchicalClustering** (Python/scipy implementation of KMD clustering providing the KMDClustering class with min_cluster_size and certainty parameters for outlier-aware partitioning) — github.com/KaplanLab/KMDHierarchicalClustering
- **hkariti/DBCV** (Optimized Python DBCV implementation used in the authors' workflow (e.g. for cluster-number estimation after outlier removal)) — https://github.com/hkariti/DBCV

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd = KMDClustering(k='compute', n_clusters=2, min_cluster_size='compute', certainty=0.5, k_scan_range=(1,100,3)); kmd.fit(X); y = kmd.predict(X)
```

## Evaluation signals

- On the nested-circles toy dataset (factor=0.3, noise=0.05, 1000 objects), accuracy/NMI/ARI must equal 1 after outlier re-assignment
- Predicted labels must match Table S1 values when assembling toy_dataset_clustering_scores.csv
- No cluster smaller than min_cluster_size remains in the core labeling; outliers are either re-assigned or left out per the certainty threshold
- The optimal k selected by KMD silhouette falls within the k_scan_range (article limited k < 100)
- Fitting reproduces the scikit-learn-style API: fit(X) followed by predict(X) returns labels for all 1000 objects

## Limitations

- Automatic k selection requires running clustering over a large range of k values, requiring more computational resources
- A single run of KMD clustering may be inherently slower than single/average linkage since it maintains a list of k minimal distances for each pair of clusters; the python/scipy implementation results in longer running times
- The sampling-based extension may miss very small clusters that are not sampled well

## Evidence

- [discussion] Outlier re-assignment step uses KMD linkage at optimal k: "outlier re-assignment is based on KMD linkage matching the optimal k value found during clustering."
- [discussion] Tiny clusters below a size threshold are treated as outliers: "tiny clusters which are smaller than a size threshold can be considered to be outliers"
- [results] Toy benchmark achieves perfect scores on nested circles: [evidence span withheld — anchor status: not_found]
- [readme] KMDClustering parameter block from README: "KMD = KMDClustering(k='compute', n_clusters = 2, min_cluster_size = 'compute', affinity = 'euclidean', certainty = 0.5 ,k_scan_range = (1,100,3)"
- [intro] Method combines KMD linkage with outlier-aware partitioning: "combine it with a simple outlier-aware partitioning scheme in order to improve performance in noisy scenarios"
