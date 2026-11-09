---
name: nmi-ari-metric-computation
description: Use when clustering solutions from multiple algorithms must be quantitatively
  compared against known ground-truth labels using accuracy, NMI, and ARI — e.g.,
  benchmarking KMD clustering on simulated datasets where y_true is available.
when_to_use_negative:
- No ground-truth labels are available — use intrinsic measures instead, but note
  the article warns the silhouette score is biased towards globular cluster shapes
  and is not adequate; KMD silhouette or DBCV are the paper's alternatives
- Goal is de novo estimation of the number of clusters rather than comparing solutions
  against known y_true — the paper routes this through DBCV after removing outliers,
  not through accuracy/NMI/ARI
- Datasets are unlabelled experimental data (e.g., mass cytometry Levine/Samusik or
  scRNA-seq sets) where the paper relies on expert annotations or simulated ground
  truth rather than this label-matching protocol
edam_operation: http://edamontology.org/operation_3432
tools:
- name: KMDHierarchicalClustering
  role: Produces the KMD cluster labels (via kmd_cluster.predict(X) with k='compute',
    k_scan_range=(1,100,3), min_cluster_size='compute', affinity='euclidean') whose
    accuracy/NMI/ARI are computed against y_true; accepts y_true for evaluation scenarios
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: scikit-learn
  role: Generates the simulated datasets with ground-truth labels and provides the
    AgglomerativeClustering single-linkage competitor whose predictions are scored
- name: SciPy
  role: Supplies the complete and Ward linkage competitors (cluster.hierarchy.linkage
    with default parameters) that are scored with the same accuracy/NMI/ARI metrics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:6494fcd99b9bc00146a2b6951c744a39f2e7b7df6ab79fc72e492f52e4944fb8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bluster
  - FlowSOM
  - KMDHierarchicalClustering
  - PhenoGraph
  - SCCAF
  - Scanpy
  - SciPy
  - Seurat
  - hdbscan
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# nmi-ari-metric-computation

## Summary

Computes accuracy, normalized mutual information (NMI) and adjusted Rand index (ARI) by matching predicted cluster labels to ground-truth labels (y_true), enabling the KMD clustering paper's benchmark of KMD against eleven competing methods on four noisy simulated datasets.

## When to use

Use when clustering solutions from multiple algorithms must be quantitatively compared against known ground-truth labels using accuracy, NMI, and ARI — e.g., benchmarking KMD clustering on simulated datasets where y_true is available.

## When NOT to use

- No ground-truth labels are available — use intrinsic measures instead, but note the article warns the silhouette score is biased towards globular cluster shapes and is not adequate; KMD silhouette or DBCV are the paper's alternatives
- Goal is de novo estimation of the number of clusters rather than comparing solutions against known y_true — the paper routes this through DBCV after removing outliers, not through accuracy/NMI/ARI
- Datasets are unlabelled experimental data (e.g., mass cytometry Levine/Samusik or scRNA-seq sets) where the paper relies on expert annotations or simulated ground truth rather than this label-matching protocol

## Inputs

- Noisy scikit-learn simulated datasets (nested circles, half moons, globular clusters, anisotropic clusters; 1000 objects) with ground-truth labels y_true
- Predicted cluster label vectors from KMD clustering and the eleven competing methods (SciPy complete/Ward/minimax/single linkage, Scanpy Louvain/Leiden, PhenoGraph, HDBSCAN, SCCAF, FlowSOM, Seurat Louvain)

## Outputs

- Per-method accuracy, normalized mutual information (NMI), and adjusted Rand index (ARI) values for each of the four datasets
- Comparison table written to benchmark_performance_table.csv (consistent with Supplementary Tables S2–S4)

## How to apply

Generate or obtain each dataset together with its ground-truth labels (y_true) from the scikit-learn random sample generators (1000 objects, high-noise settings noise=0.14/0.24, std=[2.0,2.0,2.0], seed=185). Run every method on each dataset: KMD via KMDClustering(k='compute', k_scan_range=(1,100,3), min_cluster_size='compute', affinity='euclidean') followed by kmd_cluster.predict(X), plus the eleven competitors (complete and Ward linkage via SciPy's cluster.hierarchy.linkage, minimax linkage, single linkage via scikit-learn AgglomerativeClustering with non-default connectivity, Scanpy Louvain and Leiden, PhenoGraph, HDBSCAN with non-default parameters, SCCAF, FlowSOM, and Seurat Louvain). Match each method's predicted cluster labels to y_true (accounting for label permutation), then compute accuracy, NMI, and ARI for every method-dataset pair. Assemble the accuracy/NMI/ARI values into a comparison table and write it to benchmark_performance_table.csv. Correct application requires KMD accuracies to reproduce the reported values (0.989 nested circles, 0.933 half moons, 0.909 globular clusters, 0.992 anisotropic clusters) and KMD to be the only method performing well on all four noisy problems, consistent with Supplementary Tables S2–S4.

## Related tools

- **KMDHierarchicalClustering** (Produces the KMD cluster labels (via kmd_cluster.predict(X) with k='compute', k_scan_range=(1,100,3), min_cluster_size='compute', affinity='euclidean') whose accuracy/NMI/ARI are computed against y_true; accepts y_true for evaluation scenarios) — github.com/KaplanLab/KMDHierarchicalClustering
- **scikit-learn** (Generates the simulated datasets with ground-truth labels and provides the AgglomerativeClustering single-linkage competitor whose predictions are scored)
- **SciPy** (Supplies the complete and Ward linkage competitors (cluster.hierarchy.linkage with default parameters) that are scored with the same accuracy/NMI/ARI metrics)

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd_cluster = KMDClustering(k='compute', k_scan_range=(1,100,3), min_cluster_size='compute', affinity='euclidean'); kmd_cluster.fit(X); y_pred = kmd_cluster.predict(X)
```

## Evaluation signals

- KMD accuracies on the four noisy datasets must match the reported values: 0.989 (nested circles), 0.933 (half moons), 0.909 (globular clusters), 0.992 (anisotropic clusters)
- KMD must be the only tested method that performs well on all four noisy clustering problems
- Computed accuracy/NMI/ARI values must be consistent with Supplementary Tables S2–S4
- Metrics must be produced for all methods (KMD plus the 11 competitors) across all four datasets and written to benchmark_performance_table.csv
- Predicted labels must be matched to y_true before scoring (label correspondence is not assumed to be identity)

## Limitations

- The silhouette score is biased towards globular cluster shapes and is not adequate as an intrinsic evaluation function, so external accuracy/NMI/ARI against y_true is preferred when ground truth exists
- No software version numbers are reported for the compared tools, and no version is specified for KMDHierarchicalClustering (installed from the GitHub default branch), so exact metric reproduction may be sensitive to tool versions
- The Python/SciPy implementation results in longer running times than widely used clustering algorithms, which affects how many competitor re-runs are practical when recomputing the metric table

## Evidence

- [abstract] Rigorous comparisons to general and specialized clustering methods on simulated, mass cytometry and scRNA-seq datasets show consistent high performance of KMD clustering across all datasets.: "Rigorous comparisons to general and specialized clustering methods on simulated, mass cytometry and scRNA-seq datasets show consistent high performance of KMD clustering across all datasets."
- [discussion] All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1.: "All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1."
- [discussion] The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate.: "The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate."
- [readme] y_true: true cluster labels (only used in evaluation scenario).: "y_true: true cluster labels (only used in evaluation scenario)."
