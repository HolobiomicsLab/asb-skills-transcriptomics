---
name: resampling-based-clustering-evaluation
description: Use when a clustering dataset is too large for full hierarchical clustering
  (e.g. mass cytometry datasets with 86,864–265,627 cells) and you need repeatable
  resample-based clustering with performance metrics (accuracy, NMI, ARI) aggregated
  over resamples.
when_to_use_negative:
- The dataset is small enough to cluster directly — use sub_sample=False for a single
  run on the full matrix
- Very small clusters are of primary interest, since the sampling approach may miss
  clusters that are not sampled well
- Ground truth is unknown and you need exact cluster assignments for every object
  with no post hoc inference step
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: KMDHierarchicalClustering
  role: Provides KMDClustering with sub_sample/percent_size/seed parameters for the
    resampling-based clustering workflow and KMD silhouette-based k selection
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: Python
  role: Implementation language for the KMD clustering package (python/scipy implementation)
- name: SciPy
  role: Underlying scipy dependency used by KMDHierarchicalClustering (scipy>=1.3.0
    required)
- name: scikit-learn
  role: Dependency (sklearn>=0.23.1); KMDClustering follows the scikit-learn fit/predict
    interface
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:28e35cf738216e019286f860b13241ac0937eb65aa49d6fc4eb96c2ca90f2fbd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KMDHierarchicalClustering
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# resampling-based-clustering-evaluation

## Summary

A sampling-based extension of KMD clustering in which large datasets are subsampled repeatedly, KMD clustering is run on each resample, and clusters are inferred post hoc for the remaining objects — enabling application to million-object datasets and enabling replicate-based accuracy/NMI/ARI evaluation against known ground truth.

## When to use

Use when a clustering dataset is too large for full hierarchical clustering (e.g. mass cytometry datasets with 86,864–265,627 cells) and you need repeatable resample-based clustering with performance metrics (accuracy, NMI, ARI) aggregated over resamples.

## When NOT to use

- The dataset is small enough to cluster directly — use sub_sample=False for a single run on the full matrix
- Very small clusters are of primary interest, since the sampling approach may miss clusters that are not sampled well
- Ground truth is unknown and you need exact cluster assignments for every object with no post hoc inference step

## Inputs

- Dataset matrix X as a numpy array (e.g. transformed/filtered mass cytometry benchmark data from FlowRepository FR-FCM-ZZPH: Levine15_13, Levine15_23/Levine_32dim, Samusik16)
- Ground-truth cluster labels for evaluation scenarios (y_true)
- Subsample size or fraction (percent_size) and random seed

## Outputs

- Cluster label predictions per resample (y = kmd_cluster.predict(X))
- Per-resample accuracy, NMI and ARI scores, aggregated to mean and standard deviation per dataset
- Optional plot of KMD silhouette vs accuracy across k values (plot_scores=True)

## How to apply

Run KMDClustering.fit with sub_sample=True, specifying percent_size (fraction or absolute size of the subsample) and seed to control the random subsample; the paper's cytometry benchmark used 10 resamples of 20,000 cells per dataset with seed=1. For each resample, KMD clustering with min_cluster_size=50 and affinity='correlation' scans k over k_scan_range=(1,100,5) (k limited to <100 as in the paper) and selects the k with the highest KMD silhouette score. Remaining cells are assigned post hoc via KMD linkage matching the optimal k (certainty parameter controls outlier re-assignment). Compute accuracy, NMI and ARI against known ground-truth labels per resample, then aggregate to mean and standard deviation per dataset. Note the sampling approach may miss very small clusters that are not sampled well.

## Related tools

- **KMDHierarchicalClustering** (Provides KMDClustering with sub_sample/percent_size/seed parameters for the resampling-based clustering workflow and KMD silhouette-based k selection) — github.com/KaplanLab/KMDHierarchicalClustering
- **Python** (Implementation language for the KMD clustering package (python/scipy implementation))
- **SciPy** (Underlying scipy dependency used by KMDHierarchicalClustering (scipy>=1.3.0 required))
- **scikit-learn** (Dependency (sklearn>=0.23.1); KMDClustering follows the scikit-learn fit/predict interface)

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd = KMDClustering(affinity='correlation', min_cluster_size=50, k_scan_range=(1,100,5)); kmd.fit(X, sub_sample=True, percent_size=20000, seed=1); y = kmd.predict(X)
```

## Evaluation signals

- Levine15_32 KMD accuracy should be approximately 0.948 when resampling evaluation is run as in the paper
- Averaged over the three bone marrow cytometry datasets, KMD should achieve ~0.894 accuracy, ~0.879 NMI and ~0.892 ARI, beating PhenoGraph (0.834/0.848/0.841)
- Each resample is reproducible given the same seed (seed=1 controls the random subsample)
- The selected k for each resample should be the one with the highest KMD silhouette score within the scanned range (k < 100)

## Limitations

- The sampling-based extension may miss very small clusters, in cases where they are not sampled well
- The python/scipy implementation results in longer running times than widely used clustering algorithms
- Automatic k selection requires running clustering over a large range of k values, requiring more computational resources

## Evidence

- [discussion] Sampling-based extension for scalability to millions of objects, with per-resample subsampling controlled by percent_size and seed: "we proposed a sampling-based extension to our approach, which makes it easily scalable to modern datasets containing millions of objects."
- [discussion] One limitation of this sampling approach is that it may miss very small clusters, in cases where they are not sampled well.: "One limitation of this sampling approach is that it may miss very small clusters, in cases where they are not sampled well."
- [readme] README documents sub_sample, percent_size and seed parameters of KMD.fit for the resampling workflow: [evidence span withheld — anchor status: not_found]
- [intro] KMD silhouette-based automatic k selection across a range of k values: "we eliminate hyperparameter k by running the clustering in parallel over several values of k and picking the k value that has the highest KMD silhouette score"
- [discussion] The mass cytometry benchmark datasets used in the resampling evaluation: "The datasets Levine15_13 (167,044 Cells), Levine15_23 (265,627 cells) and Samusik16 (86,864 Cells) are known benchmarking mass cytometry datasets"
