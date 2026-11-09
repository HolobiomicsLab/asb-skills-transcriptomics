---
name: post-hoc-cluster-label-inference
description: Use when a dataset is too large to cluster directly (e.g. Levine15_32dim,
  104,184 cells; Samusik16, 86,864 cells) and a KMD model has been fit on a subsample
  whose labels must be propagated to the unsampled objects.
when_to_use_negative:
- The dataset is small enough to cluster directly without subsampling — the post hoc
  inferring step is only recommended for large datasets (sub_sample=True is 'reccomended
  on large dataset')
- Clusters of interest are very small and may be poorly represented in the subsample,
  since the sampling-based extension may miss clusters that are not sampled well
- A precomputed adjacency matrix or non-default affinity is required and incompatible
  with the subsample-then-infer flow without care
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: KMDHierarchicalClustering
  role: Provides KMDClustering whose fit(sub_sample=True) + predict() implements the
    post hoc cluster label inference over the unsampled cells
  repo: https://github.com/KaplanLab/KMDHierarchicalClustering
- name: scikit-learn
  role: Supplies the numpy/sklearn-style interface and metrics (accuracy, NMI, ARI)
    for validating propagated labels against y_true
  repo: https://github.com/KaplanLab/KMDHierarchicalClustering
- name: SciPy
  role: Underlies the KMD linkage computations and distance metrics used during inferring
    (implementation is in python/scipy)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:8d3f41f274c520dcb72c6b62a19ef84e092967c74bb2fcc62e28248fcab079cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KaplanLab/KMDHierarchicalClustering
  tools:
  - KMDHierarchicalClustering
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# post-hoc-cluster-label-inference

## Summary

Extends KMD clustering from a subsampled core to the full dataset: after fitting KMDClustering on a subsample (sub_sample=True), a post hoc inferring (predict) step labels all remaining objects against the core clusters. This is what makes KMD clustering scale to million-object biological datasets such as mass cytometry panels.

## When to use

Use when a dataset is too large to cluster directly (e.g. Levine15_32dim, 104,184 cells; Samusik16, 86,864 cells) and a KMD model has been fit on a subsample whose labels must be propagated to the unsampled objects.

## When NOT to use

- The dataset is small enough to cluster directly without subsampling — the post hoc inferring step is only recommended for large datasets (sub_sample=True is 'reccomended on large dataset')
- Clusters of interest are very small and may be poorly represented in the subsample, since the sampling-based extension may miss clusters that are not sampled well
- A precomputed adjacency matrix or non-default affinity is required and incompatible with the subsample-then-infer flow without care

## Inputs

- Dataset matrix X as a numpy array (e.g. preprocessed Levine_32dim mass cytometry measurements)
- A fitted KMDClustering model from KMDHierarchicalClustering (fit with sub_sample=True, percent_size, seed)
- Optional ground-truth labels y_true for evaluation

## Outputs

- Cluster labels for all unsampled objects (numpy array from KMDClustering.predict)
- Evaluation metrics (accuracy, NMI, ARI) versus y_true, e.g. written to sampling_strategy_comparison.csv

## How to apply

Fit KMDClustering on a subsampled core by calling fit(X, sub_sample=True, percent_size=0.2, seed=1), where percent_size<1 is a fraction of the dataset and seed fixes the random subsample; select k via k_scan_range=(1,100,3) using the KMD silhouette score. Then call predict on the full matrix (or the held-out cells) so the post hoc inferring step assigns each remaining object to a core cluster using KMD linkage at the optimal k. Control label propagation with certainty (0.5–1): certainty=0.5 assigns all outliers to core clusters, certainty=1 leaves all outliers unassigned. Evaluate propagated labels against y_true with accuracy, NMI, and ARI; on Levine15_32 the uniform-sampling baseline at 5,000 cells reaches 0.939 accuracy / 0.936 NMI / 0.966 ARI.

## Related tools

- **KMDHierarchicalClustering** (Provides KMDClustering whose fit(sub_sample=True) + predict() implements the post hoc cluster label inference over the unsampled cells) — https://github.com/KaplanLab/KMDHierarchicalClustering
- **scikit-learn** (Supplies the numpy/sklearn-style interface and metrics (accuracy, NMI, ARI) for validating propagated labels against y_true) — https://github.com/KaplanLab/KMDHierarchicalClustering
- **SciPy** (Underlies the KMD linkage computations and distance metrics used during inferring (implementation is in python/scipy))

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd = KMDClustering(k='compute', k_scan_range=(1,100,3), certainty=0.5); kmd.fit(X, sub_sample=True, percent_size=0.2, seed=1); y = kmd.predict(X)
```

## Evaluation signals

- predict() returns labels for every object outside the subsample, with the same label space as the core clustering
- With certainty=0.5 (default range 0.5–1) all outliers are assigned to core clusters; with certainty=1 none are — check assignment counts change as expected with certainty
- Propagated-label metrics on Levine15_32dim should reach the uniform-sampling baseline near 0.939 accuracy / 0.936 NMI / 0.966 ARI at a 5,000-cell budget
- The optimal k used for inferring must match the k selected by the KMD silhouette during the k_scan_range scan
- Output CSV contains both strategy rows with numeric accuracy/NMI/ARI at the stated sample budget

## Limitations

- The sampling-based extension may miss very small clusters when they are not sampled well in the core subset
- The python/scipy implementation results in longer running times than widely used optimized clustering algorithms
- Automatic k selection requires running clustering over a large range of k values, requiring more computational resources before inference can proceed

## Evidence

- [readme] sub_sample: If True, dataset will be subsampled and a post hoc cluster inffering method will be conducted, reccomended on large dataset: "`sub_sample`: If True ,dataset will be subsampled and a post hoc cluster inffering method will be conducted, reccomended on large dataset"
- [readme] certainty parameter: 0.5 assigns all outliers to core clusters, 1 means no outliers will be assigned: "`certainty`:number between 0.5 and 1 indicating a the confidence threshold to use for outlier assignment to core clusters."
- [discussion] Sampling-based extension makes KMD scalable to millions of objects: "we proposed a sampling-based extension to our approach, which makes it easily scalable to modern datasets containing millions of objects."
- [discussion] Outlier re-assignment uses KMD linkage at the optimal k found during clustering: "outlier re-assignment is based on KMD linkage matching the optimal k value found during clustering."
- [discussion] Limitation: sampling-based extension may miss very small clusters: "One limitation of this sampling approach is that it may miss very small clusters, in cases where they are not sampled well."
