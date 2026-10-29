---
name: k-scan-silhouette-evaluation
description: 'Use when clustering with KMDHierarchicalClustering and k is unknown:
  fit across a k_scan_range, compute KMD silhouette per k, and verify the silhouette
  is predictive of accuracy/NMI/ARI against ground-truth labels.'
when_to_use_negative:
- Clusters are known to be globular and you only need a generic intrinsic score —
  the article notes the ordinary silhouette is biased toward globular shapes, so it
  is not adequate as a general intrinsic evaluation function
- You have no ground-truth labels and only need the final clustering — y_true and
  plot_scores are evaluation-scenario parameters; simply fit with k='compute' rather
  than scoring correlations
- The dataset is in the millions of objects without using the sub_sample option —
  scanning a large range of k values requires substantial computational resources
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: KMDHierarchicalClustering
  role: 'Primary implementation: KMDClustering scans k values via k_scan_range and
    reports KMD silhouette vs accuracy per k'
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: scikit-learn
  role: Random sample generators used to produce the simulated two-moon datasets with
    retained ground-truth labels
- name: SciPy
  role: Python/SciPy implementation base for the clustering code
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:251f4c818be728c01a8fa286593dbd520a17dcab75c9519f3391956715c173a7
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# k-scan-silhouette-evaluation

## Summary

Runs KMD clustering in parallel over a scan of k values and uses the KMD silhouette score (a generalized silhouette) to automatically select the k whose clustering best matches actual performance, eliminating the cryptic hyperparameter k.

## When to use

Use when clustering with KMDHierarchicalClustering and k is unknown: fit across a k_scan_range, compute KMD silhouette per k, and verify the silhouette is predictive of accuracy/NMI/ARI against ground-truth labels.

## When NOT to use

- Clusters are known to be globular and you only need a generic intrinsic score — the article notes the ordinary silhouette is biased toward globular shapes, so it is not adequate as a general intrinsic evaluation function
- You have no ground-truth labels and only need the final clustering — y_true and plot_scores are evaluation-scenario parameters; simply fit with k='compute' rather than scoring correlations
- The dataset is in the millions of objects without using the sub_sample option — scanning a large range of k values requires substantial computational resources

## Inputs

- Dataset matrix X (numpy array; e.g. scikit-learn make_moons output with noise=0.24, 1000 objects)
- Ground-truth cluster labels y_true (evaluation scenario only)
- k_scan_range tuple, e.g. (1, 100)

## Outputs

- Per-k clustering accuracy and KMD silhouette scores (e.g. kmd_halfmoons_scores.csv)
- Per-instance Pearson correlations between KMD silhouette and accuracy/NMI/ARI across k values
- Selected best k (highest KMD silhouette) and corresponding clustering solution

## How to apply

Instantiate KMDClustering with n_clusters set, k='compute', k_scan_range=(1,100), y_true bound to ground-truth labels, and plot_scores=True to mirror the scikit-learn-style fit/predict interface. For each instance, fit the model on the dataset matrix and record clustering accuracy plus the KMD silhouette score at every k in the scan range (the article limits k < 100). Compute the Pearson correlation between KMD silhouette and each performance metric across k values per instance. On 50 instances of the high-noise (noise=0.24, 1000-object) two-moon dataset, median Pearson correlations reached 0.89 (accuracy), 0.85 (NMI) and 0.87 (ARI). Write per-k accuracy/silhouette scores and per-instance correlations to a CSV such as kmd_halfmoons_scores.csv. The rationale is that the standard silhouette is biased toward globular shapes, so the KMD silhouette — which uses k minimal distances between clusters — is the appropriate intrinsic criterion for selecting k.

## Related tools

- **KMDHierarchicalClustering** (Primary implementation: KMDClustering scans k values via k_scan_range and reports KMD silhouette vs accuracy per k) — github.com/KaplanLab/KMDHierarchicalClustering
- **scikit-learn** (Random sample generators used to produce the simulated two-moon datasets with retained ground-truth labels)
- **SciPy** (Python/SciPy implementation base for the clustering code)

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd_cluster = KMDClustering(n_clusters=2, k='compute', k_scan_range=(1,100), y_true=y_true, plot_scores=True); kmd_cluster.fit(X)
```

## Evaluation signals

- Median Pearson correlation between KMD silhouette and clustering accuracy across k values over 50 high-noise half-moon instances should be ~0.89
- Median correlations for NMI (~0.85) and ARI (~0.87) must match the reported benchmark values for noise=0.24
- The k selected by maximum KMD silhouette should correspond to the k with highest accuracy in the per-k score records
- Output CSV contains per-k accuracy and silhouette scores plus per-instance Pearson correlations for all 50 instances

## Limitations

- Automatic k selection requires running clustering over a large range of k values, demanding more computational resources
- The python/scipy implementation results in longer running times than widely used optimized clustering algorithms
- A single KMD clustering run is inherently slower because it must maintain lists of k minimal distances for each pair of clusters
- In all reported k scans the range was limited to k < 100

## Evidence

- [intro] KMD silhouette predicts accuracy: "we eliminate hyperparameter k by running the clustering in parallel over several values of k and picking the k value that has the highest KMD silhouette score"
- [discussion] k scan limit: "running the clustering across a range of k values (in all cases we limited k < 100) and automatically selecting the solution with the highest KMD silhouette score."
- [discussion] silhouette bias: "The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate."
- [readme] k parameter semantics: "`k`: number of minimum distances to calculate distance between clusters. if flag is compute, best k will be predicted."
- [readme] k_scan_range parameter: "`k_scan_range`: tuple indicating the range of k values to be used to search for k. Given as (start k, stop k, jumps)."
- [discussion] computation cost of k scan: "automatic selection of the hyperparameter k requires running the clustering with a large range of k values and selecting the solution with the best KMD silhouette score."
