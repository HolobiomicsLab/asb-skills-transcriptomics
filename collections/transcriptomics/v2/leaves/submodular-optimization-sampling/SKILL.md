---
name: submodular-optimization-sampling
description: Use when applying the sampling-based extension of KMD clustering to a
  large dataset (e.g. >100k cells) and you want to test whether a structurally diverse
  core subset outperforms uniform random subsampling at a fixed sample budget.
when_to_use_negative:
- Datasets small enough to cluster directly without subsampling — KMDClustering's
  sub_sample=False path is preferred and sampling adds unnecessary variance
- Datasets dominated by very small clusters — the article notes the sampling approach
  'may miss very small clusters, in cases where they are not sampled well', a failure
  mode diversity sampling does not guarantee to fix
- When the goal is de novo cluster-number estimation rather than sampling-strategy
  comparison — DBCV-based estimation is flagged as preliminary
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_1317
- http://edamontology.org/topic_2229
tools:
- name: KMDHierarchicalClustering
  role: KMDClustering is fit on the sampled core (sub_sample=True, percent_size, seed=1,
    k_scan_range=(1,100,3)) and used post hoc (predict with certainty=0.5) to infer
    labels for unsampled cells
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: scikit-learn
  role: supplies metrics (accuracy, NMI, ARI) for comparing uniform vs. submodular
    sampled clusterings against y_true
- name: SciPy
  role: underlying implementation backend for the python/scipy KMD clustering code
    and pdist-based affinities
- name: Python
  role: implementation environment for the submodular (facility-location / diversity
    maximization) sampling routine
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:4939d79594c53f73d6349862e96ec8f0dc9ab32e6aff60d9f70045933760192c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KMDHierarchicalClustering
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_1317
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# submodular-optimization-sampling

## Summary

Replaces KMD clustering's default uniform random subsampling with a structure-aware (e.g. facility-location / diversity-maximizing) submodular-optimization core-set selection at an identical sample budget, so that clustering of very large biological datasets (e.g. Levine_32dim mass cytometry, 104,184 cells) samples a structurally diverse core subset instead of a random one.

## When to use

Use when applying the sampling-based extension of KMD clustering to a large dataset (e.g. >100k cells) and you want to test whether a structurally diverse core subset outperforms uniform random subsampling at a fixed sample budget.

## When NOT to use

- Datasets small enough to cluster directly without subsampling — KMDClustering's sub_sample=False path is preferred and sampling adds unnecessary variance
- Datasets dominated by very small clusters — the article notes the sampling approach 'may miss very small clusters, in cases where they are not sampled well', a failure mode diversity sampling does not guarantee to fix
- When the goal is de novo cluster-number estimation rather than sampling-strategy comparison — DBCV-based estimation is flagged as preliminary

## Inputs

- Levine_32dim mass cytometry dataset (Levine15_32, 104,184 cells) as a numpy feature matrix with ground-truth cell labels
- Other large biological datasets with ground-truth labels (mass cytometry or scRNA-seq) where sampled KMD clustering is used
- Fixed sample budget (e.g. percent_size=0.2 or 5000 cells) and KMDClustering parameters (k_scan_range=(1,100,3), seed=1, certainty=0.5)

## Outputs

- sampling_strategy_comparison.csv with rows for strategy (uniform vs. submodular) and columns accuracy, NMI, ARI, and sample budget
- KMD clustering labels for all cells (core fit + post hoc inference) under each sampling strategy
- Selected k value from the KMD silhouette score scan

## How to apply

First establish the uniform-sampling baseline: fit KMDClustering with sub_sample=True, percent_size (e.g. 0.2, corresponding to 5000 cells on Levine15_32's 104,184 cells) and seed=1, with k_scan_range=(1,100,3) so k is selected by the KMD silhouette score — the paper found 5000 uniform samples suffice for optimal performance (0.939 accuracy, 0.936 NMI, 0.966 ARI). Then implement a submodular-optimization sampling routine (facility-location / diversity maximization) to select an equal-size, structurally diverse core subset from the same dataset at the identical budget. Fit KMDClustering on the submodular core with the same settings, then run the post hoc inferring (predict) step with certainty=0.5 to label the remaining cells. Evaluate both strategies against y_true with accuracy, NMI, and ARI, and write a comparison CSV keyed by strategy. The rationale is that KMD's sampling extension enables scaling to million-object datasets, and a diversity-maximizing core set should preserve rare structure better than random draws.

## Related tools

- **KMDHierarchicalClustering** (KMDClustering is fit on the sampled core (sub_sample=True, percent_size, seed=1, k_scan_range=(1,100,3)) and used post hoc (predict with certainty=0.5) to infer labels for unsampled cells) — github.com/KaplanLab/KMDHierarchicalClustering
- **scikit-learn** (supplies metrics (accuracy, NMI, ARI) for comparing uniform vs. submodular sampled clusterings against y_true)
- **SciPy** (underlying implementation backend for the python/scipy KMD clustering code and pdist-based affinities)
- **Python** (implementation environment for the submodular (facility-location / diversity maximization) sampling routine)

## Examples

```
kmd_cluster = KMDClustering(n_clusters='compute', k='compute', k_scan_range=(1,100,3)); kmd_cluster.fit(X_core, sub_sample=False); y = kmd_cluster.predict(X_full)
```

## Evaluation signals

- sampling_strategy_comparison.csv exists and contains both strategy rows (uniform and submodular) with numeric accuracy/NMI/ARI values
- The submodular row matches or improves the uniform-sampling baseline metrics (baseline: 0.939 accuracy, 0.936 NMI, 0.966 ARI on Levine15_32)
- Both strategies use the identical sample budget (e.g. 5000 cells / percent_size=0.2), so the comparison is budget-controlled
- k was selected automatically within k_scan_range=(1,100,3) by the KMD silhouette score rather than set a priori
- Post hoc predict with certainty=0.5 labels all cells outside the core subset, so full-dataset metrics (accuracy, NMI, ARI) are computed against the complete y_true

## Limitations

- The sampling-based extension may miss very small clusters that are not sampled well
- Automatic k selection requires running clustering over a large range of k values, requiring more computational resources
- The python/scipy implementation results in longer running times than widely used optimized clustering algorithms

## Evidence

- [abstract] Sampling is used to enable application to million-object datasets.: "and use sampling to enable application to million-object datasets"
- [discussion] k selected by KMD silhouette across a scan range: "running the clustering across a range of k values (in all cases we limited k < 100) and automatically selecting the solution with the highest KMD silhouette score."
- [discussion] sampling extension scalability rationale: "we proposed a sampling-based extension to our approach, which makes it easily scalable to modern datasets containing millions of objects."
- [discussion] small-cluster failure mode of sampling: "One limitation of this sampling approach is that it may miss very small clusters, in cases where they are not sampled well."
- [readme] KMDClustering fit signature with subsampling parameters: "KMD.fit(X,sub_sample=False,percent_size=0.2,seed = 1)"
- [readme] certainty parameter controls outlier assignment: "`certainty`:number between 0.5 and 1 indicating a the confidence threshold to use for outlier assignment to core clusters."
