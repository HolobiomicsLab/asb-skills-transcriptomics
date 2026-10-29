---
name: baseline-clustering-algorithm-comparison
description: Use when a new general-purpose clustering algorithm must be validated
  against standard baselines on simulated datasets with known ground-truth labels,
  producing a per-algorithm, per-dataset metric table for publication-style comparison.
when_to_use_negative:
- The input is experimental data without ground-truth labels — accuracy/ARI scoring
  against truth is not possible and a different validation strategy (e.g. DBCV) is
  needed
- Intrinsic cluster evaluation is the goal using the plain silhouette score — it is
  biased towards globular cluster shapes and is not adequate as an intrinsic evaluation
  function
- Datasets contain very small clusters that may not be sampled well — the sampling-based
  extension used for scalability may miss them
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: KMDHierarchicalClustering
  role: Candidate clustering method fitted on each toy dataset with k='compute', min_cluster_size='compute',
    k scanned and selected by KMD silhouette score
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: scikit-learn
  role: Generates the simulated datasets via random sample generators and provides
    AgglomerativeClustering (single linkage, non-default connectivity) and DBSCAN
    baselines
- name: SciPy
  role: Runs complete and Ward linkage baselines via scipy's cluster.hierarchy.linkage
    with default parameters
- name: hdbscan
  role: HDBSCAN baseline run with the non-default parameter setting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:2fbe472bd061ee0895b286b96b28892f8b93f87ee3c4532ec3e6fae87c6316c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KMDHierarchicalClustering
  - Python
  - SciPy
  - hdbscan
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# baseline-clustering-algorithm-comparison

## Summary

Benchmark a candidate clustering method (here, KMD clustering) against six general-purpose baselines — spectral via AgglomerativeClustering with non-default connectivity, single linkage, complete/Ward linkage via SciPy, DBSCAN, HDBSCAN, and Gaussian mixture — on the four standard scikit-learn simulated datasets, scoring accuracy, NMI, and ARI against ground truth.

## When to use

Use when a new general-purpose clustering algorithm must be validated against standard baselines on simulated datasets with known ground-truth labels, producing a per-algorithm, per-dataset metric table for publication-style comparison.

## When NOT to use

- The input is experimental data without ground-truth labels — accuracy/ARI scoring against truth is not possible and a different validation strategy (e.g. DBCV) is needed
- Intrinsic cluster evaluation is the goal using the plain silhouette score — it is biased towards globular cluster shapes and is not adequate as an intrinsic evaluation function
- Datasets contain very small clusters that may not be sampled well — the sampling-based extension used for scalability may miss them

## Inputs

- Four scikit-learn simulated datasets (1000 objects, seed = 1): concentric circles (factor=0.3, noise=0.05), two moons (noise=0.05), three Gaussian blobs (cluster_std=[1.0, 2.5, 0.5], random_state=170), anisotropic blobs
- Ground-truth cluster labels for each simulated dataset
- Fitted cluster label assignments from each of the 7 algorithms (candidate + 6 baselines)

## Outputs

- 7-algorithm x 4-dataset metric table of accuracy, NMI, and ARI (toy_dataset_clustering_scores.csv), including KMD core-clusters-only rows
- Per-algorithm cluster labelings on each simulated dataset

## How to apply

Generate the four scikit-learn random sample datasets (concentric circles with factor=0.3 and noise=0.05, two moons with noise=0.05, three Gaussian blobs with cluster_std=[1.0, 2.5, 0.5] and random_state=170, and an anisotropically transformed blob set) with 1000 objects and seed = 1, retaining ground-truth labels. Fit the candidate method — KMDClustering with k='compute' and min_cluster_size='compute', scanning k over k_scan_range=(1,100,3) and selecting the solution with the highest KMD silhouette score, with outlier re-assignment at the optimal k. Run the comparison algorithms on the same matrices: single linkage via AgglomerativeClustering with the non-default connectivity parameter, complete and Ward linkage via scipy.cluster.hierarchy.linkage with default parameters, DBSCAN, HDBSCAN (hdbscan python package, non-default parameter setting), and Gaussian mixture modeling. Score every predicted labeling against ground truth using accuracy, normalized mutual information (NMI), and adjusted Rand index (ARI), including rows for KMD core clusters only, and assemble the 7-algorithm x 4-dataset table (e.g. toy_dataset_clustering_scores.csv).

## Related tools

- **KMDHierarchicalClustering** (Candidate clustering method fitted on each toy dataset with k='compute', min_cluster_size='compute', k scanned and selected by KMD silhouette score) — github.com/KaplanLab/KMDHierarchicalClustering
- **scikit-learn** (Generates the simulated datasets via random sample generators and provides AgglomerativeClustering (single linkage, non-default connectivity) and DBSCAN baselines)
- **SciPy** (Runs complete and Ward linkage baselines via scipy's cluster.hierarchy.linkage with default parameters)
- **hdbscan** (HDBSCAN baseline run with the non-default parameter setting)

## Examples

```
pip install git+https://github.com/KaplanLab/KMDHierarchicalClustering; from KMDHierarchicalClustering import KMDClustering; kmd_cluster = KMDClustering(n_clusters = 2); kmd_cluster.fit(X)
```

## Evaluation signals

- KMD must attain accuracy/NMI/ARI = 1 on the nested-circles dataset and on the half-moons dataset
- The assembled 7-algorithm x 4-dataset metric table must match the values reported in Table S1
- Every algorithm-dataset pair yields all three metrics (accuracy, NMI, ARI) with no missing cells, including KMD core-clusters-only rows
- k scan is limited to k < 100 and the reported optimal k is the one maximizing the KMD silhouette score
- Toy_dataset_example.ipynb reproduces the full comparison in approximately 8 minutes of expected run time

## Limitations

- Automatic selection of the hyperparameter k requires running the clustering with a large range of k values, requiring more computational resources
- The current implementation is in python/scipy, resulting in longer running times than widely used clustering algorithms with optimized, non-interpreted implementations
- A single run of KMD clustering may be inherently slower than single or average linkage because it must maintain, for each pair of clusters, a list of k minimal distances
- No software version numbers are reported for the compared tools, including KMDHierarchicalClustering itself

## Evidence

- [discussion] All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1.: "All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1."
- [discussion] Single linkage via AgglomerativeClustering with non-default connectivity: "Single linkage clustering was run with the AgglomerativeClustering scikit-learn algorithm, using the non-default parameter connectivity"
- [discussion] Complete and Ward linkage via scipy.cluster.hierarchy.linkage with default parameters: "Complete and Ward linkages were run using Scipy’s cluster.hierarchy.linkage method using default parameters"
- [discussion] HDBSCAN baseline configuration: "HDBSCAN was run with the hdbscan python package39, using the non-default parameter setting"
- [discussion] KMD fit across k range with automatic silhouette-based selection: "running the clustering across a range of k values (in all cases we limited k < 100) and automatically selecting the solution with the highest KMD silhouette score."
- [readme] Toy dataset demo notebook scope and runtime: "Toy_dataset_example.ipynb - Evaluates performance on a standard set of simulated toy datasets - expected run time: ~8 minutes"
- [discussion] Silhouette score bias caveat motivating KMD silhouette / DBCV alternatives: "The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate."
