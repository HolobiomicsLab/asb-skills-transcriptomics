---
name: single-cell-clustering-algorithm-comparison
description: Use when comparing clustering method performance on single-cell (mass
  cytometry or scRNA-seq) datasets with known cell-type labels, or when testing whether
  a clustering method stays accurate at large scale via subsampling.
when_to_use_negative:
- The dataset has no ground-truth labels or known number of populations — the benchmark
  requires n_clusters set a priori and scoring against y_true
- The dataset is small (a few thousand cells or fewer) — sub_sample=True is recommended
  only for large datasets, and the sampling extension may miss very small clusters
  that are not sampled well
- The goal is de novo estimation of the number of clusters — DBCV-based cluster number
  selection is described as preliminary and needing further research
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: KMDHierarchicalClustering
  role: 'Primary method under evaluation: KMD linkage hierarchical clustering with
    KMD silhouette-based k selection, outlier-aware partitioning and subsampled fitting
    via the scikit-learn-style KMDClustering API'
  repo: https://github.com/KaplanLab/KMDHierarchicalClustering
- name: Scanpy
  role: 'Comparison method: Louvain and Leiden community-detection clustering run
    on the same datasets with the same sample sizes'
- name: SCCAF
  role: 'Comparison method: self-projection machine learning clustering applied to
    the same benchmark datasets'
- name: scikit-learn
  role: Provides random sample generators for simulated datasets and scoring metrics
    used to compare solutions against ground truth
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:9d4c7b5e73f6fc1145409a0a8fa6b88750b2971bce47eeddef2f599cd114f794
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KaplanLab/KMDHierarchicalClustering
  tools:
  - KMDHierarchicalClustering
  - Python
  - SCCAF
  - Scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-clustering-algorithm-comparison

## Summary

Benchmarks clustering algorithms (sampled KMD clustering, Scanpy Louvain/Leiden, SCCAF) on single-cell datasets with known ground truth, scoring agreement via ARI, NMI and accuracy across sample sizes. Grounded in the KMD clustering paper's sampling-based scaling study on Levine_32dim, PBMC68K_Pure and a Splatter-simulated million-cell dataset.

## When to use

Use when comparing clustering method performance on single-cell (mass cytometry or scRNA-seq) datasets with known cell-type labels, or when testing whether a clustering method stays accurate at large scale via subsampling.

## When NOT to use

- The dataset has no ground-truth labels or known number of populations — the benchmark requires n_clusters set a priori and scoring against y_true
- The dataset is small (a few thousand cells or fewer) — sub_sample=True is recommended only for large datasets, and the sampling extension may miss very small clusters that are not sampled well
- The goal is de novo estimation of the number of clusters — DBCV-based cluster number selection is described as preliminary and needing further research

## Inputs

- Levine_32dim mass cytometry dataset from FlowRepository FR-FCM-ZZPH (104,184 cells, preprocessed; unassigned/ambiguously annotated cells discarded)
- PBMC68K_Pure scRNA-seq dataset from Brown et al.
- Splatter-simulated dataset (1,000,000 cells, parameters estimated from Zeisel15, seed=1)
- Ground-truth cell labels (y_true) with known number of populations for each dataset

## Outputs

- kmd_sample_size_performance.csv with columns: sample size, repeats, ARI/accuracy per method
- kmd_algorithm_comparison.csv with per-method agreement (ARI/NMI/accuracy) with ground truth at each sample size
- Optional KMD silhouette vs accuracy plots per k value (plot_scores=True)

## How to apply

Install KMDHierarchicalClustering (pip install git+https://github.com/KaplanLab/KMDHierarchicalClustering) and fit KMDClustering with n_clusters set to the known number of populations, sub_sample=True, percent_size corresponding to a 5,000-cell core, and seed=1, letting k be chosen automatically by the KMD silhouette over k_scan_range (1,100,3). Repeat the sampled fit across subsample sizes 1,000–20,000 with 5 repeats per size, scoring each solution against y_true with ARI/NMI/accuracy. Run Scanpy Louvain, Scanpy Leiden and SCCAF on the same datasets with the same sample sizes as comparison methods. Write results to two CSVs: kmd_sample_size_performance.csv (performance vs sample size) and kmd_algorithm_comparison.csv (per-method agreement with ground truth). Reference points: Levine_32dim ARI ~0.966 and simulated-dataset accuracy ~0.956 at sample size 5,000; on full Levine15_32 (104,184 cells), 5,000 sampled cells sufficed (0.939 accuracy, 0.936 NMI, 0.966 ARI).

## Related tools

- **KMDHierarchicalClustering** (Primary method under evaluation: KMD linkage hierarchical clustering with KMD silhouette-based k selection, outlier-aware partitioning and subsampled fitting via the scikit-learn-style KMDClustering API) — https://github.com/KaplanLab/KMDHierarchicalClustering
- **Scanpy** (Comparison method: Louvain and Leiden community-detection clustering run on the same datasets with the same sample sizes)
- **SCCAF** (Comparison method: self-projection machine learning clustering applied to the same benchmark datasets)
- **scikit-learn** (Provides random sample generators for simulated datasets and scoring metrics used to compare solutions against ground truth)

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd = KMDClustering(k='compute', n_clusters=n_populations, k_scan_range=(1,100,3)); kmd.fit(X, sub_sample=True, percent_size=5000, seed=1)
```

## Evaluation signals

- Both CSV files (kmd_sample_size_performance.csv, kmd_algorithm_comparison.csv) exist with columns for sample size, repeats, and ARI/accuracy per method
- KMD entries at sample size 5,000 fall in the reported range: ARI ~0.966 on Levine_32dim and accuracy ~0.956 on the Splatter-simulated data
- Full-dataset Levine15_32 baseline reproduced: ~0.939 accuracy, 0.936 NMI, 0.966 ARI with a 5,000-cell core
- Each sample size in the 1,000–20,000 sweep has 5 independent repeats (seeded subsampling, seed=1) so performance-vs-size trends are stable
- Scanpy Louvain/Leiden and SCCAF entries use identical datasets and sample sizes as the KMD entries, so the algorithm comparison is like-for-like

## Limitations

- The sampling-based extension may miss very small clusters, in cases where they are not sampled well
- Automatic k selection requires running the clustering over a large range of k values, requiring more computational resources
- The python/scipy implementation results in longer running times than widely used clustering algorithms, and a single KMD run is inherently slower because it maintains lists of k minimal distances for each pair of clusters
- No software version numbers are reported for the compared tools, so exact reproducibility of the comparison may vary

## Evidence

- [abstract] Sampling is used to enable application to million-object datasets.: [evidence span withheld — anchor status: not_found]
- [readme] Subsampling recommendation for large datasets: "sub_sample: If True ,dataset will be subsampled and a post hoc cluster inffering method will be conducted, reccomended on large dataset"
- [discussion] k selected by KMD silhouette across a scan range: "running the clustering across a range of k values (in all cases we limited k < 100) and automatically selecting the solution with the highest KMD silhouette score."
- [discussion] Sampling limitation for small clusters: "One limitation of this sampling approach is that it may miss very small clusters, in cases where they are not sampled well."
- [discussion] Silhouette biased toward globular shapes: "The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate."
- [intro] Outlier-aware partitioning rationale: "combine it with a simple outlier-aware partitioning scheme in order to improve performance in noisy scenarios"
