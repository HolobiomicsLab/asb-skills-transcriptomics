---
name: kmd-silhouette-k-selection
description: 'Use when fitting KMDClustering without an a priori k: ground-truth labels
  are unavailable, the number of neighbors driving the KMD linkage is unknown, and
  the user wants k selected intrinsically via the KMD silhouette instead of manual
  tuning.'
when_to_use_negative:
- When k is already known or scientifically motivated — the scan runs clustering repeatedly
  over many k values and requires substantially more computational resources
- When cluster shape is strongly non-globular and an intrinsic criterion is needed
  — the silhouette score is biased towards globular cluster shapes; prefer DBCV-based
  validation instead
- When using the sampled KMD extension on datasets with very small clusters — sampling
  may miss small clusters regardless of the selected k
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: KMDHierarchicalClustering
  role: Provides KMDClustering with k='compute' and k_scan_range=(1,100,3) so best
    k is predicted by maximizing the KMD silhouette score
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: scikit-learn
  role: Dependency of the KMD clustering implementation (sklearn>=0.23.1) used within
    the k scan
- name: SciPy
  role: Numerical/hierarchical-clustering backend of the python/scipy KMD implementation
    used during the k scan
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:65f8e7c8d3ad4ba212376612f96c53d269e86ffb43afc87da67544f2c6fbc992
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KMDHierarchicalClustering
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# kmd-silhouette-k-selection

## Summary

Automatic elimination of the cryptic hyperparameter k in KMD clustering by running the clustering in parallel over a range of k values (e.g. k_scan_range=(1,100,3)) and selecting the k whose solution has the highest generalized KMD silhouette score, rather than requiring the user to guess k a priori.

## When to use

Use when fitting KMDClustering without an a priori k: ground-truth labels are unavailable, the number of neighbors driving the KMD linkage is unknown, and the user wants k selected intrinsically via the KMD silhouette instead of manual tuning.

## When NOT to use

- When k is already known or scientifically motivated — the scan runs clustering repeatedly over many k values and requires substantially more computational resources
- When cluster shape is strongly non-globular and an intrinsic criterion is needed — the silhouette score is biased towards globular cluster shapes; prefer DBCV-based validation instead
- When using the sampled KMD extension on datasets with very small clusters — sampling may miss small clusters regardless of the selected k

## Inputs

- Dataset matrix X (numpy array) to cluster
- KMDClustering configuration: k='compute' and k_scan_range tuple, e.g. (1,100,3)
- Optional ground-truth labels y_true for evaluation-mode diagnostics

## Outputs

- Best-k selection (k with maximal KMD silhouette score) and clustering labels fit at that k
- Optional plot of KMD silhouette vs accuracy across the scanned k values (plot_scores=True)
- Predicted labels from kmd_cluster.predict(X) at the selected k

## How to apply

Set k='compute' in KMDClustering so the algorithm scans a range of k values given as k_scan_range=(start_k, stop_k, jumps); the paper's default scan is (1,100,3) with k kept below 100 in all cases. For each candidate k the clustering is run and a generalized silhouette-like function (KMD silhouette, an intrinsic measure predictive of clustering quality) is computed on the resulting partition. The k with the highest KMD silhouette score is then fixed for the final solution; in evaluation scenarios y_true and plot_scores=True can be supplied to inspect KMD silhouette vs accuracy across k values. This removes the need to choose k a priori, which the paper warns can otherwise lead to post hoc fitting and hidden overfitting. If a coarse scan is used (jumps of 3 in the default range), precision of the selected k is limited by the scan grid.

## Related tools

- **KMDHierarchicalClustering** (Provides KMDClustering with k='compute' and k_scan_range=(1,100,3) so best k is predicted by maximizing the KMD silhouette score) — github.com/KaplanLab/KMDHierarchicalClustering
- **scikit-learn** (Dependency of the KMD clustering implementation (sklearn>=0.23.1) used within the k scan)
- **SciPy** (Numerical/hierarchical-clustering backend of the python/scipy KMD implementation used during the k scan)

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd = KMDClustering(k='compute', n_clusters=2, k_scan_range=(1,100,3)); kmd.fit(X)
```

## Evaluation signals

- KMDClustering is instantiated with k='compute' so the returned cluster object reports a predicted best k rather than a user-fixed k
- The selected k lies within the supplied k_scan_range (e.g. 1–100 in steps of 3) and corresponds to the highest KMD silhouette score across that range
- With y_true supplied, the KMD silhouette maximum should align with the accuracy peak in the plot_scores=True diagnostic plot
- Repeated runs with the same seed=1 produce the same selected k and labels (deterministic scan)
- Downstream metrics (accuracy, NMI, ARI) against y_true remain consistent when the scan grid is refined, confirming the k selection is stable

## Limitations

- Automatic selection of k requires running the clustering with a large range of k values, requiring more computational resources
- The silhouette score is biased towards globular cluster shapes and is not adequate as a general intrinsic evaluation function
- A single KMD run is inherently slower because it maintains lists of k minimal distances for each pair of clusters, and the python/scipy implementation has longer running times

## Evidence

- [intro] we eliminate hyperparameter k by running the clustering in parallel over several values of k and picking the k value that has the highest KMD silhouette score: "we eliminate hyperparameter k by running the clustering in parallel over several values of k and picking the k value that has the highest KMD silhouette score"
- [discussion] running the clustering across a range of k values (in all cases we limited k < 100) and automatically selecting the solution with the highest KMD silhouette score: "running the clustering across a range of k values (in all cases we limited k < 100) and automatically selecting the solution with the highest KMD silhouette score."
- [readme] `k`: number of minimum distances to calculate distance between clusters. if flag is compute, best k will be predicted.: "`k`: number of minimum distances to calculate distance between clusters. if flag is compute, best k will be predicted."
- [readme] k_scan_range`: tuple indicating the range of k values to be used to search for k. Given as (start k, stop k, jumps).: "k_scan_range`: tuple indicating the range of k values to be used to search for k. Given as (start k, stop k, jumps)."
- [abstract] We introduce a generalized silhouette-like function to eliminate the cryptic hyperparameter k: "We introduce a generalized silhouette-like function to eliminate the cryptic hyperparameter k"
