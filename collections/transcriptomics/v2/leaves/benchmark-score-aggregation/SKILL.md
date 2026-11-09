---
name: benchmark-score-aggregation
description: Use when you have per-run score tables (e.g., KMD silhouette vs. accuracy
  for every scanned k value) over repeated dataset instances and need to quantify
  score–performance agreement, e.g., median Pearson correlation across instances.
when_to_use_negative:
- When the goal is a single clustering solution rather than validating an intrinsic
  quality measure — then fit KMD clustering and select k directly by highest KMD silhouette
  instead of aggregating scores
- 'When evaluating globular-only clusters with the standard silhouette: the article
  notes the silhouette score is biased towards globular cluster shapes and is not
  adequate as an intrinsic evaluation function, which is why the KMD silhouette is
  used instead'
- 'When ground truth is unavailable: Pearson correlation against accuracy/NMI/ARI
  requires y_true; the KMD silhouette is computed regardless, but the predictive-validity
  aggregation is not possible'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: KMDHierarchicalClustering
  role: Provides KMDClustering, which yields per-k clustering accuracy and KMD silhouette
    scores (with plot_scores=True) used as the benchmark signals to aggregate
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: scikit-learn
  role: Random sample generators produce the repeated simulated instances (e.g., make_moons
    with noise=0.24, 1000 objects) with retained ground-truth labels
- name: SciPy
  role: Underlying dependency for linkage computation and the python/scipy implementation
    on which KMD clustering runs
- name: Python
  role: Implementation language for running the score-scan loop and computing/aggregating
    Pearson correlations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:cc5485d77fde0b89d2f9117545c7ef81a5b9b922d6b4c0e204667160aeb21056
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KMDHierarchicalClustering
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# benchmark-score-aggregation

## Summary

Aggregates per-run benchmark scores (e.g., per-k clustering accuracy, NMI, ARI and intrinsic quality scores such as the KMD silhouette) across many dataset instances into summary statistics (Pearson correlations and medians) to test whether an intrinsic measure predicts actual clustering performance.

## When to use

Use when you have per-run score tables (e.g., KMD silhouette vs. accuracy for every scanned k value) over repeated dataset instances and need to quantify score–performance agreement, e.g., median Pearson correlation across instances.

## When NOT to use

- When the goal is a single clustering solution rather than validating an intrinsic quality measure — then fit KMD clustering and select k directly by highest KMD silhouette instead of aggregating scores
- When evaluating globular-only clusters with the standard silhouette: the article notes the silhouette score is biased towards globular cluster shapes and is not adequate as an intrinsic evaluation function, which is why the KMD silhouette is used instead
- When ground truth is unavailable: Pearson correlation against accuracy/NMI/ARI requires y_true; the KMD silhouette is computed regardless, but the predictive-validity aggregation is not possible

## Inputs

- Per-instance score table: clustering accuracy, NMI, ARI, and intrinsic score (KMD silhouette) for each k in the scan range (k < 100)
- Ground-truth labels retained from simulated data generation (e.g., sklearn make_moons with noise=0.24, 1000 objects, 50 instances)
- Dataset matrix X in numpy array format fed to KMDClustering.fit

## Outputs

- Per-instance Pearson correlations between KMD silhouette and each performance metric (accuracy, NMI, ARI)
- Median correlations across instances as the aggregated benchmark statistic
- Aggregated score file (e.g., kmd_halfmoons_scores.csv) with all per-k accuracy and silhouette scores

## How to apply

Run the clustering (here KMDClustering with n_clusters=2, k='compute', k_scan_range=(1,100), y_true set to ground-truth labels) over repeated instances of the benchmark data — e.g., 50 instances of the high-noise (noise=0.24) two-moons dataset with 1000 objects, seed=3 for the Fig. 2 instance, generated with scikit-learn random sample generators. For each instance record the clustering accuracy, NMI, and ARI alongside the intrinsic score (KMD silhouette) for every k in the scan range. Compute the Pearson correlation between intrinsic score and each external metric across k values within each instance, then aggregate across instances by taking the median. Correct application is judged by whether the median correlations reproduce reported benchmarks (accuracy ~0.89, NMI ~0.85, ARI ~0.87 at this noise level), and all per-k and per-instance values should be persisted (e.g., kmd_halfmoons_scores.csv) for auditability.

## Related tools

- **KMDHierarchicalClustering** (Provides KMDClustering, which yields per-k clustering accuracy and KMD silhouette scores (with plot_scores=True) used as the benchmark signals to aggregate) — github.com/KaplanLab/KMDHierarchicalClustering
- **scikit-learn** (Random sample generators produce the repeated simulated instances (e.g., make_moons with noise=0.24, 1000 objects) with retained ground-truth labels)
- **SciPy** (Underlying dependency for linkage computation and the python/scipy implementation on which KMD clustering runs)
- **Python** (Implementation language for running the score-scan loop and computing/aggregating Pearson correlations)

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd_cluster = KMDClustering(n_clusters=2, k='compute', k_scan_range=(1,100), y_true=y_true, plot_scores=True); kmd_cluster.fit(X)
```

## Evaluation signals

- Median Pearson correlation between KMD silhouette and accuracy across the 50 instances is ~0.89, matching the reported benchmark
- Median correlations for NMI (~0.85) and ARI (~0.87) also match the reported values at noise=0.24
- Score file (e.g., kmd_halfmoons_scores.csv) contains per-k accuracy and silhouette scores for all 50 instances plus per-instance correlations
- k scan covers the intended range (in all cases k < 100) and each k value yields paired silhouette/accuracy observations
- KMD silhouette vs accuracy plot (plot_scores=True) shows co-varying curves across k for representative instances (e.g., seed=3 for the Fig. 2 instance)

## Limitations

- Automatic k selection requires running clustering over a large range of k values, requiring more computational resources
- The python/scipy implementation results in longer running times than widely used clustering algorithms
- A single run of KMD clustering may be inherently slower because it maintains, for each pair of clusters, a list of k minimal distances
- The standard silhouette score is biased towards globular cluster shapes and is thus not adequate as an intrinsic evaluation function — the aggregation must use the KMD silhouette variant
- Pearson correlation across k within a single instance can be misleading; the median across 50 instances is the aggregate statistic relied upon

## Evidence

- [results] On 50 instances of the high-noise half moons dataset, the KMD silhouette score is predictive of all three clustering performance metrics, with median Pearson correlations of 0.89 (accuracy), 0.85 (NMI) and 0.87 (ARI).: [evidence span withheld — anchor status: not_found]
- [intro] k-selection-by-scan workflow: "we eliminate hyperparameter k by running the clustering in parallel over several values of k and picking the k value that has the highest KMD silhouette score"
- [discussion] silhouette inadequacy rationale: "The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate."
- [discussion] scan-range convention: "running the clustering across a range of k values (in all cases we limited k < 100) and automatically selecting the solution with the highest KMD silhouette score."
- [discussion] simulated data generation: "All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1."
- [readme] KMDClustering parameterization and fit interface: "KMD = KMDClustering(k='compute', n_clusters = 2, min_cluster_size = 'compute', affinity = 'euclidean', certainty = 0.5 ,k_scan_range = (1,100,3), y_true = [], plot_scores=False, path=False)"
