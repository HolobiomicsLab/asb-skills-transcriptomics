---
name: large-dataset-subsampled-clustering
description: Use when a clustering task involves very large datasets (e.g. 100k-1M
  cells/objects, such as mass cytometry or scRNA-seq) where full-dataset KMD clustering
  is computationally infeasible, and the number of populations is known.
when_to_use_negative:
- Dataset is small enough to cluster directly without subsampling (the repo recommends
  sub_sample only on large datasets)
- The number of clusters is unknown — the sampling scheme was demonstrated with n_clusters
  set to the known number of populations, and de novo cluster number estimation via
  DBCV is only preliminary
- Very small clusters are expected that may not be captured by a small random sample
  (a stated limitation of the sampling approach)
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: KMDHierarchicalClustering
  role: KMDClustering estimator fit with sub_sample=True and percent_size to cluster
    the core subsample and re-assign remaining objects via KMD linkage
  repo: https://github.com/KaplanLab/KMDHierarchicalClustering
- name: scikit-learn
  role: Scoring sampled clustering solutions against y_true (ARI/accuracy)
- name: Scanpy
  role: Baseline comparison methods (Louvain, Leiden) run on the same datasets and
    sample sizes
- name: SCCAF
  role: Baseline comparison clustering method evaluated on the same sampled datasets
- name: Splatter
  role: Generates the simulated million-cell dataset used to test scaling (parameters
    estimated from Zeisel15, seed=1)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:2e1a25c3914572f03de7b4f7164ecb23378fd0c542a46f1af97ebf211bd17571
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KaplanLab/KMDHierarchicalClustering
  tools:
  - KMDHierarchicalClustering
  - Python
  - SCCAF
  - Scanpy
  - Splatter
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# large-dataset-subsampled-clustering

## Summary

Scales KMD hierarchical clustering (KMD linkage + KMD silhouette + outlier-aware partitioning) to million-object datasets by clustering a small random subsample and post-hoc assigning remaining objects via the outlier classification scheme.

## When to use

Use when a clustering task involves very large datasets (e.g. 100k-1M cells/objects, such as mass cytometry or scRNA-seq) where full-dataset KMD clustering is computationally infeasible, and the number of populations is known.

## When NOT to use

- Dataset is small enough to cluster directly without subsampling (the repo recommends sub_sample only on large datasets)
- The number of clusters is unknown — the sampling scheme was demonstrated with n_clusters set to the known number of populations, and de novo cluster number estimation via DBCV is only preliminary
- Very small clusters are expected that may not be captured by a small random sample (a stated limitation of the sampling approach)

## Inputs

- Feature matrix X (numpy array) of a large biological dataset, e.g. Levine_32dim mass cytometry from FlowRepository FR-FCM-ZZPH (104,184 cells), PBMC68K_Pure (Brown et al.), or a Splatter-simulated 1,000,000-cell dataset (parameters estimated from Zeisel15, seed=1)
- Known number of populations (ground-truth cluster count) for n_clusters
- Ground-truth labels y_true for evaluation

## Outputs

- Cluster label predictions for the full dataset (core clusters plus post-hoc outlier re-assignment)
- Performance-vs-sample-size table (e.g. kmd_sample_size_performance.csv with sample size, repeats, ARI/accuracy)
- Algorithm-comparison table (e.g. kmd_algorithm_comparison.csv) against Scanpy Louvain, Scanpy Leiden and SCCAF

## How to apply

Fit KMDClustering with n_clusters set to the known number of populations, sub_sample=True, and percent_size set to produce a small core sample (e.g. a 5,000-cell core on Levine15_32 mass cytometry data with 104,184 cells), fixing seed=1 for reproducibility; let k be selected automatically via the KMD silhouette over k_scan_range (1,100,3). On the sampled core, KMD clustering runs over multiple k values, picks the k with the highest KMD silhouette score, and partitions with the outlier-aware scheme; remaining objects are then re-assigned to core clusters by the outlier classification scheme using KMD linkage at the optimal k. Benchmark against full-sample fits by scoring predictions against y_true with ARI, NMI and accuracy; on Levine_32dim, sweep subsample sizes 1,000-20,000 with 5 repeats per size to check performance stability. Rationale: sampling 5,000 cells was sufficient for optimal sampled KMD performance on 104,184-cell mass cytometry data, and slightly larger samples can even decrease performance.

## Related tools

- **KMDHierarchicalClustering** (KMDClustering estimator fit with sub_sample=True and percent_size to cluster the core subsample and re-assign remaining objects via KMD linkage) — https://github.com/KaplanLab/KMDHierarchicalClustering
- **scikit-learn** (Scoring sampled clustering solutions against y_true (ARI/accuracy))
- **Scanpy** (Baseline comparison methods (Louvain, Leiden) run on the same datasets and sample sizes)
- **SCCAF** (Baseline comparison clustering method evaluated on the same sampled datasets)
- **Splatter** (Generates the simulated million-cell dataset used to test scaling (parameters estimated from Zeisel15, seed=1))

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd = KMDClustering(n_clusters=32, k='compute'); kmd.fit(X, sub_sample=True, percent_size=5000, seed=1)
```

## Evaluation signals

- On Levine_32dim with a 5,000-cell subsample, sampled KMD clustering reaches ARI ~0.966, NMI ~0.936 and accuracy ~0.939 against y_true
- On the simulated dataset at sample size 5,000, accuracy falls in the reported range (~0.956)
- Performance-vs-sample-size table spans subsample sizes 1,000-20,000 with 5 repeats per size, and 5,000 cells is the point at which sampled performance saturates on Levine15_32 (104,184 cells)
- Both CSV outputs exist with required columns (sample size, repeats, ARI/accuracy per method)
- k is auto-selected (k='compute') via KMD silhouette over the default k_scan_range (1,100,3) rather than set manually

## Limitations

- The sampling-based extension may miss very small clusters in cases where they are not sampled well
- Automatic k selection requires running the clustering over a large range of k values, requiring more computational resources
- The python/scipy implementation results in longer running times than widely used clustering algorithms

## Evidence

- [results] Sampling sufficiency finding: [evidence span withheld — anchor status: not_found]
- [abstract] Sampling motivation: "and use sampling to enable application to million-object datasets"
- [intro] k selection method: "we eliminate hyperparameter k by running the clustering in parallel over several values of k and picking the k value that has the highest KMD silhouette score"
- [readme] sub_sample parameter: "sub_sample`: If True ,dataset will be subsampled and a post hoc cluster inffering method will be conducted, reccomended on large dataset"
- [discussion] small cluster limitation: "One limitation of this sampling approach is that it may miss very small clusters, in cases where they are not sampled well."
