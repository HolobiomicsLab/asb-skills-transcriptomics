---
name: clustering-accuracy-evaluation-with-known-labels
description: Use when clustering results on datasets with known cell-type or class
  labels (e.g. mass cytometry benchmarks or simulated data with ground truth) must
  be quantitatively scored by ARI, NMI and accuracy for method comparison.
when_to_use_negative:
- No ground-truth labels exist for the dataset — de novo evaluation requires the KMD
  silhouette / DBCV route instead, and DBCV-based cluster-number estimation is only
  preliminary
- Selecting clusters by the standard silhouette score — it is biased towards globular
  cluster shapes and inadequate as an intrinsic evaluation function for non-globular
  clusters
- Evaluating a single published number without repeats — the benchmarking protocol
  uses 5 repeats per sample size to capture sampling variance
edam_operation: http://edamontology.org/operation_2995
tools:
- name: KMDHierarchicalClustering
  role: Provides the KMDClustering scikit-learn-style interface whose y_true and plot_scores
    parameters drive accuracy-based evaluation of fitted clusterings
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: scikit-learn
  role: Source of adjusted Rand index / accuracy scoring against y_true and of the
    AgglomerativeClustering baselines being compared
- name: Scanpy
  role: Louvain/Leiden comparison clusterings scored on the same labeled benchmarks
- name: SCCAF
  role: Self-projection-based scRNA-seq clustering baseline included in the same-labels
    comparison table
- name: DBCV
  role: Density-based clustering validation used to pick the number of clusters when
    ground truth is unavailable
  repo: https://github.com/hkariti/DBCV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:e4084310d8f2a9572b3dc7de1decfa1275bb8036bc0081a2f93f6e30437173f4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DBCV
  - KMDHierarchicalClustering
  - Python
  - SCCAF
  - Scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2995
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# clustering-accuracy-evaluation-with-known-labels

## Summary

Scores clustering solutions against known ground-truth labels (y_true) using accuracy, NMI and ARI, as done when benchmarking KMD clustering on datasets with known populations such as Levine_32dim, PBMC68K_Pure and Splatter-simulated data.

## When to use

Use when clustering results on datasets with known cell-type or class labels (e.g. mass cytometry benchmarks or simulated data with ground truth) must be quantitatively scored by ARI, NMI and accuracy for method comparison.

## When NOT to use

- No ground-truth labels exist for the dataset — de novo evaluation requires the KMD silhouette / DBCV route instead, and DBCV-based cluster-number estimation is only preliminary
- Selecting clusters by the standard silhouette score — it is biased towards globular cluster shapes and inadequate as an intrinsic evaluation function for non-globular clusters
- Evaluating a single published number without repeats — the benchmarking protocol uses 5 repeats per sample size to capture sampling variance

## Inputs

- Feature/data matrix X (numpy array) of clustered objects
- Ground-truth class labels y_true for the same objects
- Cluster label assignments from KMDClustering or a comparison method (Scanpy Louvain/Leiden, SCCAF)

## Outputs

- Per-method accuracy, NMI and ARI scores against y_true
- Performance-vs-sample-size and algorithm-comparison tables (e.g. kmd_sample_size_performance.csv, kmd_algorithm_comparison.csv)
- Plot of KMD Silhouette vs accuracy across k values (when plot_scores=True)

## How to apply

Attach the true labels to the fitted model via the y_true parameter of KMDClustering (y_true is used only in the evaluation scenario), optionally setting plot_scores=True to generate a plot of KMD Silhouette vs accuracy across k values from the k_scan_range (default (1,100,3)). For sampled fits, score each solution against y_true with metrics such as ARI and accuracy and repeat across sample sizes (e.g. subsample sizes 1,000-20,000 with 5 repeats per size) to build a performance-vs-sample-size table. Record reference points for judging success: on the full Levine15_32 mass cytometry dataset (104,184 cells), a 5,000-cell core sample achieved 0.939 accuracy, 0.936 NMI and 0.966 ARI, and the simulated million-cell dataset reached ~0.956 accuracy at sample size 5,000. Do not rely on the standard silhouette score alone as the intrinsic evaluation function, since it is biased towards globular cluster shapes; use the KMD silhouette for k selection and DBCV (after removing outliers) for de novo cluster-number estimation.

## Related tools

- **KMDHierarchicalClustering** (Provides the KMDClustering scikit-learn-style interface whose y_true and plot_scores parameters drive accuracy-based evaluation of fitted clusterings) — github.com/KaplanLab/KMDHierarchicalClustering
- **scikit-learn** (Source of adjusted Rand index / accuracy scoring against y_true and of the AgglomerativeClustering baselines being compared)
- **Scanpy** (Louvain/Leiden comparison clusterings scored on the same labeled benchmarks)
- **SCCAF** (Self-projection-based scRNA-seq clustering baseline included in the same-labels comparison table)
- **DBCV** (Density-based clustering validation used to pick the number of clusters when ground truth is unavailable) — https://github.com/hkariti/DBCV

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd_cluster = KMDClustering(n_clusters=32, y_true=y_true, plot_scores=True); kmd_cluster.fit(X, sub_sample=True, percent_size=5000, seed=1)
```

## Evaluation signals

- KMD entries at sample size 5,000 fall in the reported range: ARI ~0.966 on Levine_32dim and accuracy ~0.956 on the Splatter-simulated dataset
- Both output CSVs exist with required columns (sample size, repeats, ARI/accuracy per method)
- ARI/NMI scores are bounded in [-1, 1] and accuracy in [0, 1]; values far outside these ranges indicate a scoring bug
- With plot_scores=True, a KMD Silhouette vs accuracy plot across the k_scan_range is produced for the evaluation scenario
- Comparison baselines (Scanpy Louvain/Leiden, SCCAF) are scored on the same sample sizes with the same metrics for a fair table

## Limitations

- The standard silhouette score is biased towards globular cluster shapes and is not adequate as an intrinsic evaluation function
- Automatic k selection requires running clustering over a large range of k values, requiring more computational resources
- The python/scipy implementation results in longer running times than widely used clustering algorithms
- The sampling-based extension may miss very small clusters that are not sampled well, biasing evaluation of rare populations

## Evidence

- [readme] y_true used only in evaluation: "y_true: true cluster labels (only used in evaluation scenario)."
- [readme] plot of KMD Silhouette vs accuracy: "plot_scores: if True, a plot of KMD Silhouette vs accuracy across different k values will be generated"
- [abstract] rigorous benchmarking claim: "Rigorous comparisons to general and specialized clustering methods on simulated, mass cytometry and scRNA-seq datasets show consistent high performance"
- [discussion] silhouette bias caveat: "The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate."
- [discussion] DBCV-based cluster number selection: "we calculate the DBCV (after removing outliers) of the solution with the k value selected by the KMD silhouette"
