---
name: clustering-benchmark-evaluation
description: Use when a new clustering method must be rigorously compared to competitors
  (e.g., PhenoGraph, HDBSCAN, Louvain/Leiden implementations) on simulated or biological
  datasets where ground-truth labels are available.
when_to_use_negative:
- The dataset has no ground-truth labels and only intrinsic quality measures (e.g.,
  KMD silhouette, DBCV) can be computed — external benchmark metrics like accuracy/NMI/ARI
  require y_true.
- The goal is to tune or select hyperparameters (k) rather than compare methods —
  use the KMD silhouette score scan, not a multi-method benchmark.
- Evaluating clustering on globular-shaped clusters only, where the standard silhouette
  score suffices and is biased toward such shapes anyway.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: KMDHierarchicalClustering
  role: Method under test; KMDClustering provides fit/predict labels with automatic
    k selection via KMD silhouette
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: scikit-learn
  role: Generates simulated datasets with ground-truth labels and provides single-linkage
    AgglomerativeClustering competitor
- name: SciPy
  role: Runs complete and Ward linkage competitors via cluster.hierarchy.linkage
- name: Scanpy
  role: Louvain and Leiden competitor implementations for cell clustering
- name: Seurat
  role: Louvain competitor implementation
- name: Bluster
  role: Louvain competitor implementation
- name: PhenoGraph
  role: Shared-neighbor-graph Louvain community-detection competitor
- name: hdbscan
  role: HDBSCAN competitor, run with non-default parameters
- name: SCCAF
  role: Self-projection machine-learning competitor for scRNA-seq clustering
- name: FlowSOM
  role: Self-organizing-map-based competitor for mass cytometry clustering
provenance:
  source_task_ids:
  - task_002
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:e6e16c899dc0f0bb90ca59ffc39446ff4d79b8d9b38f9aa9c5439ae377c3fd92
schema_version: 0.2.0
metadata:
  license_tier: open
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
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# clustering-benchmark-evaluation

## Summary

Benchmark a clustering method against multiple competing general-purpose and domain-specific algorithms on datasets with known ground truth, reporting accuracy, normalized mutual information (NMI) and adjusted Rand index (ARI) per method and dataset.

## When to use

Use when a new clustering method must be rigorously compared to competitors (e.g., PhenoGraph, HDBSCAN, Louvain/Leiden implementations) on simulated or biological datasets where ground-truth labels are available.

## When NOT to use

- The dataset has no ground-truth labels and only intrinsic quality measures (e.g., KMD silhouette, DBCV) can be computed — external benchmark metrics like accuracy/NMI/ARI require y_true.
- The goal is to tune or select hyperparameters (k) rather than compare methods — use the KMD silhouette score scan, not a multi-method benchmark.
- Evaluating clustering on globular-shaped clusters only, where the standard silhouette score suffices and is biased toward such shapes anyway.

## Inputs

- Simulated datasets with ground-truth labels (scikit-learn random sample generators, 1000 objects, specified noise/std/seed)
- Experimental biological datasets with known ground truth (e.g., Levine_32dim, Levine15_13, Levine15_23, Samusik16 mass cytometry; Zeisel15, Lawlor17, Li17, PBMC68K_Pure scRNA-seq)
- Fitted cluster label assignments from the method under test and each competitor

## Outputs

- Per-method, per-dataset table of accuracy, NMI, and ARI (e.g., benchmark_performance_table.csv)
- Ranking of methods across benchmark problems, highlighting methods that perform well on all problems

## How to apply

Generate or load datasets with retained ground-truth labels (e.g., noisy scikit-learn simulations: nested circles noise=0.14, half moons noise=0.24, globular std=[2.0,2.0,2.0], anisotropic, seed=185; or mass cytometry/scRNA-seq sets like Levine_32dim, Levine15_13, Samusik16). Run the method under test — here KMDClustering with k='compute', k_scan_range=(1,100,3), min_cluster_size='compute', affinity='euclidean', obtaining labels via kmd_cluster.predict(X) — plus each competitor with the settings specified in Methods (e.g., complete and Ward linkage via Scipy's cluster.hierarchy.linkage, single linkage via scikit-learn AgglomerativeClustering, Scanpy Louvain and Leiden, PhenoGraph, HDBSCAN with non-default parameters, SCCAF, FlowSOM, Seurat Louvain). Match predicted cluster labels to y_true (accounting for label permutation) and compute accuracy, NMI, and ARI for every method on every dataset. Assemble a comparison table (e.g., benchmark_performance_table.csv) and check it against published reference values, such as KMD accuracies of 0.989 (nested circles), 0.933 (half moons), 0.909 (globular), 0.992 (anisotropic). The key claim to verify is relative: whether the method under test is the only one performing well across all problems.

## Related tools

- **KMDHierarchicalClustering** (Method under test; KMDClustering provides fit/predict labels with automatic k selection via KMD silhouette) — github.com/KaplanLab/KMDHierarchicalClustering
- **scikit-learn** (Generates simulated datasets with ground-truth labels and provides single-linkage AgglomerativeClustering competitor)
- **SciPy** (Runs complete and Ward linkage competitors via cluster.hierarchy.linkage)
- **Scanpy** (Louvain and Leiden competitor implementations for cell clustering)
- **Seurat** (Louvain competitor implementation)
- **Bluster** (Louvain competitor implementation)
- **PhenoGraph** (Shared-neighbor-graph Louvain community-detection competitor)
- **hdbscan** (HDBSCAN competitor, run with non-default parameters)
- **SCCAF** (Self-projection machine-learning competitor for scRNA-seq clustering)
- **FlowSOM** (Self-organizing-map-based competitor for mass cytometry clustering)

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd_cluster = KMDClustering(k='compute', n_clusters=2, k_scan_range=(1,100,3), min_cluster_size='compute', affinity='euclidean'); kmd_cluster.fit(X); y_pred = kmd_cluster.predict(X)
```

## Evaluation signals

- The benchmark CSV exists and contains accuracy/NMI/ARI values for every method-dataset pair.
- KMD accuracies on the four noisy simulated datasets match 0.989 (nested circles), 0.933 (half moons), 0.909 (globular clusters), 0.992 (anisotropic clusters), consistent with Supplementary Tables S2-S4.
- Label matching between predicted clusters and y_true is done before computing accuracy (raw label IDs are not comparable across methods).
- The benchmarked method is the only one among all tested competitors performing well on all four noisy clustering problems.
- Competitor settings mirror the article's Methods section (e.g., HDBSCAN non-default parameters, single-linkage AgglomerativeClustering with non-default connectivity).

## Limitations

- No software version numbers are reported for the compared tools, and no version is specified for KMDHierarchicalClustering (installed from GitHub master), so exact reproducibility of competitor results may vary.
- Benchmark metrics like accuracy, NMI, and ARI require ground-truth labels; they cannot be computed on unlabeled data.
- Competitor runtimes and behaviors depend on each implementation (KMD itself is in python/scipy and is slower than optimized non-interpreted implementations).

## Evidence

- [abstract] Rigorous comparisons to general and specialized clustering methods on simulated, mass cytometry and scRNA-seq datasets show consistent high performance of KMD clustering across all datasets.: "Rigorous comparisons to general and specialized clustering methods on simulated, mass cytometry and scRNA-seq datasets show consistent high performance of KMD clustering across all datasets."
- [methods] Complete and Ward linkages were run using Scipy's cluster.hierarchy.linkage method using default parameters: "Complete and Ward linkages were run using Scipy’s cluster.hierarchy.linkage method using default parameters"
- [methods] HDBSCAN was run with the hdbscan python package, using the non-default parameter setting: "HDBSCAN was run with the hdbscan python package39, using the non-default parameter setting"
- [methods] All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1.: "All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1."
- [discussion] The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate.: "The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate."
