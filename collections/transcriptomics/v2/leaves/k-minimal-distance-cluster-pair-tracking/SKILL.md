---
name: k-minimal-distance-cluster-pair-tracking
description: Use when implementing hierarchical clustering with an arbitrary linkage
  function (e.g., KMD linkage), where naive agglomerative recomputation is impractical
  and you need linear-time linkage updates and k-independent memory.
when_to_use_negative:
- When a standard linkage (single, average, complete, Ward) suffices — those run directly
  via SciPy's cluster.hierarchy.linkage and KMD's k-list bookkeeping only adds overhead
- 'When n is in the millions: the O(n^2) memory implementation is infeasible; the
  paper''s sampling-based extension is the appropriate route instead'
- When you only need cluster labels rather than the full dendrogram and cannot afford
  KMD's inherently slower per-run cost of maintaining k-distance lists for each cluster
  pair
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: KMDHierarchicalClustering
  role: Python/scipy implementation of KMD linkage clustering with the k-minimal-distance
    pair tracking; installable via pip from GitHub
  repo: https://github.com/KaplanLab/KMDHierarchicalClustering
- name: SciPy
  role: cluster.hierarchy.linkage provides the single-linkage (k=1 equivalent) and
    average-linkage reference dendrograms used for cophenetic-correlation validation
- name: scikit-learn
  role: random sample generators produce the 1000-object simulated toy datasets (seed=1)
    used to benchmark and validate the implementation
- name: psutil
  role: checks available/peak memory during the O(n^2)-memory runs, recommended for
    Windows and macOS users
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:2dbdbb117a23637f4945807012b89c64c6edbd3d6086cde1b0a1b09bafb6748a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KaplanLab/KMDHierarchicalClustering
  tools:
  - KMDHierarchicalClustering
  - Python
  - SciPy
  - psutil
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# k-minimal-distance-cluster-pair-tracking

## Summary

Efficiently implements KMD (k-minimal distances) linkage for agglomerative hierarchical clustering by maintaining, for each pair of clusters, a heap of the k smallest inter-cluster distances, merging sorted distance lists in O(k) per pair to achieve O(nk) linkage updates with O(n^2) memory independent of k.

## When to use

Use when implementing hierarchical clustering with an arbitrary linkage function (e.g., KMD linkage), where naive agglomerative recomputation is impractical and you need linear-time linkage updates and k-independent memory.

## When NOT to use

- When a standard linkage (single, average, complete, Ward) suffices — those run directly via SciPy's cluster.hierarchy.linkage and KMD's k-list bookkeeping only adds overhead
- When n is in the millions: the O(n^2) memory implementation is infeasible; the paper's sampling-based extension is the appropriate route instead
- When you only need cluster labels rather than the full dendrogram and cannot afford KMD's inherently slower per-run cost of maintaining k-distance lists for each cluster pair

## Inputs

- Distance matrix or feature matrix X (numpy array) for n objects, with pairwise distances computed via a chosen affinity (e.g., euclidean, correlation, spearman, or precomputed adjacency matrix)
- Hyperparameter k: number of minimal distances retained per cluster pair (or k='compute' to scan a k_scan_range such as (1,100,3))

## Outputs

- Hierarchical clustering linkage matrix (dendrogram) using KMD linkage
- Timing and peak-memory log per dataset and per k setting (e.g., kmd_linkage_verification_report.md)
- Cophenetic correlation values against SciPy single-linkage (k=1) and average-linkage (large k) references

## How to apply

For every pair of active clusters, keep a sorted list/heap of the k minimal inter-cluster distances rather than a single summary statistic. When two clusters merge, form the candidate distance lists for the new cluster against all others by merging the two sorted child-pair lists in O(k) per pair, giving O(nk) work per agglomeration iteration and O(n^2) total memory that does not grow with k. This makes runs at large k (e.g., k up to the <100 scan range used in the paper) feasible. Validate the implementation by running at k=1 and comparing the dendrogram to SciPy's cluster.hierarchy.linkage single linkage via cophenetic correlation, and at a large k against average linkage, accepting when both cophenetic correlations are >= 0.99 on the simulated toy datasets (scikit-learn generators, 1000 objects, seed=1). Record per-dataset timing and peak memory (psutil is recommended for memory checks on Windows/macOS) for each k setting.

## Related tools

- **KMDHierarchicalClustering** (Python/scipy implementation of KMD linkage clustering with the k-minimal-distance pair tracking; installable via pip from GitHub) — https://github.com/KaplanLab/KMDHierarchicalClustering
- **SciPy** (cluster.hierarchy.linkage provides the single-linkage (k=1 equivalent) and average-linkage reference dendrograms used for cophenetic-correlation validation)
- **scikit-learn** (random sample generators produce the 1000-object simulated toy datasets (seed=1) used to benchmark and validate the implementation)
- **psutil** (checks available/peak memory during the O(n^2)-memory runs, recommended for Windows and macOS users)

## Examples

```
pip install git+https://github.com/KaplanLab/KMDHierarchicalClustering; from KMDHierarchicalClustering import KMDClustering; kmd_cluster = KMDClustering(k='compute', n_clusters=2, k_scan_range=(1,100,3)); kmd_cluster.fit(X)
```

## Evaluation signals

- Cophenetic correlation between the k=1 KMD dendrogram and SciPy single linkage is >= 0.99 on each simulated toy dataset
- Cophenetic correlation between the large-k KMD dendrogram and SciPy average linkage is >= 0.99 on each simulated toy dataset
- Per-iteration linkage update cost scales as O(nk) and peak memory stays O(n^2) regardless of the k value used
- Timing and peak-memory measurements are logged for every dataset and k setting and written to the verification report (kmd_linkage_verification_report.md)

## Limitations

- A single KMD run is inherently slower than single or average linkage because it must maintain a list of the k minimal distances for each pair of clusters
- The python/scipy implementation results in longer running times than widely used clustering algorithms implemented in non-interpreted languages
- Automatic selection of k requires running the clustering over a large range of k values (k<100 in the paper), which demands more computational resources

## Evidence

- [results] For KMD linkage, the authors show a method for efficient linkage update in linear time and a quadratic memory implementation that is independent of k: [evidence span withheld — anchor status: not_found]
- [discussion] a single run of KMD clustering may be inherently slower than some other approaches (such as single and average linkage clustering) as it requires maintaining for each pairs of clusters a list of the: "a single run of KMD clustering may be inherently slower than some other approaches (such as single and average linkage clustering) as it requires maintaining for each pairs of clusters a list of the"
- [methods] All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1.: "All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1."
- [discussion] Complete and Ward linkages were run using Scipy’s cluster.hierarchy.linkage method using default parameters: "Complete and Ward linkages were run using Scipy’s cluster.hierarchy.linkage method using default parameters"
- [readme] psutil>=5.8.0  # recommended for Windows and macOS platform users, in order to check available memory: "psutil>=5.8.0  # recommended for Windows and macOS platform users, in order to check available memory"
