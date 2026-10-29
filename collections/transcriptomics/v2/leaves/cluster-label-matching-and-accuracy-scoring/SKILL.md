---
name: cluster-label-matching-and-accuracy-scoring
description: Use when evaluating a clustering algorithm's predictions against known
  ground-truth labels from data generators or annotated datasets, and you need accuracy,
  NMI, and ARI per method to build a comparison table (e.g. Supplementary Tables S2–S4).
when_to_use_negative:
- Datasets with no ground-truth labels available — accuracy against y_true cannot
  be computed and the silhouette score itself is inadequate because it is biased towards
  globular cluster shapes
- Intrinsic model selection during clustering (e.g. choosing k) — the article distinguishes
  KMD silhouette-based selection from this post hoc label-matching/evaluation step,
  which is only an evaluation scenario
- Comparing clustering solutions on arbitrary-shaped clusters where DBCV (density-based
  clustering validation) is the appropriate quality measure rather than label-matched
  accuracy
edam_operation: http://edamontology.org/operation_2990
tools:
- name: KMDHierarchicalClustering
  role: Produces the predicted cluster labels (kmd_cluster.predict(X)) that are matched
    against y_true and scored; y_true can also be passed to KMDClustering in evaluation
    scenario
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: scikit-learn
  role: Generates simulated datasets with ground-truth labels via random sample generators
    and provides the clustering baselines (AgglomerativeClustering single linkage)
    being scored
- name: SciPy
  role: Provides complete and Ward linkage baselines via cluster.hierarchy.linkage
    whose predictions are matched to ground truth and scored
- name: hdbscan
  role: HDBSCAN baseline run with non-default parameters; its labels are matched to
    y_true and scored in the benchmark table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:6e5a0b7c3c8b35b78b1e0c8b756c1b51eb2ff937d3e18b5ab434d9a4be8bde5d
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cluster-label-matching-and-accuracy-scoring

## Summary

Matches predicted cluster labels to ground-truth labels (y_true) and computes clustering accuracy, normalized mutual information, and adjusted Rand index for every method and dataset, then assembles a benchmark comparison table. This is the scoring layer that lets KMD clustering (0.989/0.933/0.909/0.992 accuracy on noisy scikit-learn datasets) be compared against 11 competitors on equal footing.

## When to use

Use when evaluating a clustering algorithm's predictions against known ground-truth labels from data generators or annotated datasets, and you need accuracy, NMI, and ARI per method to build a comparison table (e.g. Supplementary Tables S2–S4).

## When NOT to use

- Datasets with no ground-truth labels available — accuracy against y_true cannot be computed and the silhouette score itself is inadequate because it is biased towards globular cluster shapes
- Intrinsic model selection during clustering (e.g. choosing k) — the article distinguishes KMD silhouette-based selection from this post hoc label-matching/evaluation step, which is only an evaluation scenario
- Comparing clustering solutions on arbitrary-shaped clusters where DBCV (density-based clustering validation) is the appropriate quality measure rather than label-matched accuracy

## Inputs

- Dataset matrix X (e.g. noisy scikit-learn simulated datasets with noise=0.14/0.24, std=[2.0,2.0,2.0], seed=185)
- Ground-truth cluster labels y_true from data generators or expert annotation
- Predicted cluster label vectors from each clustering method (KMD and 11 competitors)

## Outputs

- Per-method accuracy, normalized mutual information, and adjusted Rand index values
- Benchmark comparison table written to benchmark_performance_table.csv (comparable to Supplementary Tables S2-S4)

## How to apply

Run each clustering method (e.g. KMDClustering with k='compute', k_scan_range=(1,100,3), min_cluster_size='compute', affinity='euclidean', labels obtained via kmd_cluster.predict(X)) on the same dataset matrix X, keeping the ground-truth labels from the scikit-learn generators (1000 objects, seed=1 for simulations). Because predicted cluster IDs are arbitrary, match predicted labels to y_true by optimal permutation before computing accuracy, and additionally compute normalized mutual information and adjusted Rand index, which are permutation-invariant. Repeat for every method (complete/Ward linkage via Scipy cluster.hierarchy.linkage, minimax linkage, single linkage via AgglomerativeClustering, Scanpy Louvain/Leiden, PhenoGraph, HDBSCAN with non-default parameters, SCCAF, FlowSOM, Seurat Louvain) across all four noisy datasets (nested circles, half moons, globular, anisotropic). Assemble accuracy/NMI/ARI for all methods into a single benchmark table (benchmark_performance_table.csv). Validation: KMD accuracies must match the reported 0.989, 0.933, 0.909, 0.992 and KMD must be the only method performing well on all four problems.

## Related tools

- **KMDHierarchicalClustering** (Produces the predicted cluster labels (kmd_cluster.predict(X)) that are matched against y_true and scored; y_true can also be passed to KMDClustering in evaluation scenario) — github.com/KaplanLab/KMDHierarchicalClustering
- **scikit-learn** (Generates simulated datasets with ground-truth labels via random sample generators and provides the clustering baselines (AgglomerativeClustering single linkage) being scored)
- **SciPy** (Provides complete and Ward linkage baselines via cluster.hierarchy.linkage whose predictions are matched to ground truth and scored)
- **hdbscan** (HDBSCAN baseline run with non-default parameters; its labels are matched to y_true and scored in the benchmark table)

## Examples

```
kmd_cluster = KMDClustering(k='compute', k_scan_range=(1,100,3), min_cluster_size='compute', affinity='euclidean'); kmd_cluster.fit(X); y_pred = kmd_cluster.predict(X); match y_pred to y_true and compute accuracy/NMI/ARI
```

## Evaluation signals

- KMD accuracies on the four noisy scikit-learn problems must equal the reported values: 0.989 (nested circles), 0.933 (half moons), 0.909 (globular clusters), 0.992 (anisotropic clusters)
- Accuracy, NMI, and ARI must be computed for every method on all four datasets — a complete 12-method × 4-dataset × 3-metric grid
- KMD must be the only tested method that performs well on all four noisy problems (consistent with Supplementary Tables S2-S4)
- Predicted label IDs must be matched to y_true via an optimal permutation before accuracy is computed; NMI and ARI should agree with accuracy in ranking methods
- benchmark_performance_table.csv must be produced containing the assembled accuracy/NMI/ARI comparison

## Limitations

- The silhouette score is biased towards globular cluster shapes and is thus not adequate as an intrinsic evaluation function, so plain accuracy-based comparison is limited to problems with ground truth
- No software version numbers are reported for the compared tools, so exact reproduction of competitor scores is not guaranteed
- KMD's automatic k selection requires running clustering over a large range of k values, consuming more computational resources before labels can be scored

## Evidence

- [methods] Match predicted cluster labels to ground truth (y_true) and compute accuracy, normalized mutual information, and adjusted Rand index for every method on all four datasets.: [evidence span withheld — anchor status: not_found]
- [results] KMD accuracies must match reported values (0.989, 0.933, 0.909, 0.992): [evidence span withheld — anchor status: not_found]
- [discussion] All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1.: "All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1."
- [readme] y_true: true cluster labels (only used in evaluation scenario): "`y_true`: true cluster labels (only used in evaluation scenario)."
- [discussion] Silhouette biased towards globular cluster shapes: "The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate."
