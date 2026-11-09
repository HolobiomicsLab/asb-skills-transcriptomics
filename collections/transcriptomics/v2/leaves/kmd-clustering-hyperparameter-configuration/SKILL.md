---
name: kmd-clustering-hyperparameter-configuration
description: Use when fitting KMD clustering to a biological dataset (mass cytometry,
  scRNA-seq, or simulated data) and you need to set KMDClustering hyperparameters
  — especially whether to let k be auto-computed via KMD silhouette or fixed, and
  how to configure sub-sampling for large datasets.
when_to_use_negative:
- When the goal is de novo estimation of the number of clusters — the article notes
  DBCV-based cluster number estimation is only preliminary and 'further research is
  needed'
- When compute time is critical — automatic k selection requires scanning many k values
  and the python/scipy implementation has longer running times than optimized implementations
- When the dataset contains very small clusters that a sub-sampled fit may miss because
  they are not sampled well
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: KMDHierarchicalClustering
  role: Provides the KMDClustering scikit-learn-style API whose hyperparameters (k,
    n_clusters, min_cluster_size, affinity, certainty, k_scan_range, sub_sample, percent_size,
    seed) this skill configures
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: SciPy
  role: Underlying implementation of KMD clustering (python/scipy); affinity metrics
    reuse scipy.spatial.distance.pdist
- name: scikit-learn
  role: Dependency of KMDHierarchicalClustering (sklearn>=0.23.1); KMDClustering usage
    follows the scikit-learn interface
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:4f0cb7b347070262a51dca4eadd1c9f1fc943c1d4a9266a057b2fd860fe5aaf7
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
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# kmd-clustering-hyperparameter-configuration

## Summary

Configuration guide for KMDHierarchicalClustering's KMDClustering class: choosing k, min_cluster_size, affinity, certainty, and k_scan_range so the KMD silhouette score selects the optimal k automatically and outlier-aware partitioning is applied correctly.

## When to use

Use when fitting KMD clustering to a biological dataset (mass cytometry, scRNA-seq, or simulated data) and you need to set KMDClustering hyperparameters — especially whether to let k be auto-computed via KMD silhouette or fixed, and how to configure sub-sampling for large datasets.

## When NOT to use

- When the goal is de novo estimation of the number of clusters — the article notes DBCV-based cluster number estimation is only preliminary and 'further research is needed'
- When compute time is critical — automatic k selection requires scanning many k values and the python/scipy implementation has longer running times than optimized implementations
- When the dataset contains very small clusters that a sub-sampled fit may miss because they are not sampled well

## Inputs

- Dataset matrix X (numpy array) of cells/features, e.g. transformed mass cytometry benchmark data from FlowRepository FR-FCM-ZZPH (Levine15_13, Levine15_32, Samusik16)
- Precomputed adjacency/distance matrix when affinity='precomputed'
- Optional ground-truth labels y_true for evaluation scenarios

## Outputs

- Cluster label predictions y = kmd_cluster.predict(X)
- Automatically selected best k from the k_scan_range search
- KMD Silhouette vs accuracy plot across k values when plot_scores=True

## How to apply

Install the package with `pip install git+https://github.com/KaplanLab/KMDHierarchicalClustering` and instantiate `KMDClustering(k='compute', n_clusters=<K>, affinity='euclidean'|'correlation'|'spearman'|'precomputed', k_scan_range=(1,100,3))`. The main parameter users should set is n_clusters; the paper's cytometry benchmark used affinity='correlation', min_cluster_size=50, and k_scan_range=(1,100,5), limiting k<100 and letting the highest KMD silhouette pick the optimal k. For datasets with hundreds of thousands of cells (e.g. Levine15_13 with 167,044 cells or Samusik16 with 86,864 cells), call `fit(X, sub_sample=True, percent_size=..., seed=1)` so the model subsamples and infers remaining clusters post hoc; set min_cluster_size so tiny clusters are treated as outliers, and use certainty (0.5 = all outliers left out, 1 = all assigned to core clusters) to control outlier re-assignment via KMD linkage at the optimal k. Rationale: cryptic hyperparameters like k change clustering results dramatically, so KMD eliminates the choice by running clustering over a range of k values and picking the one with the highest KMD silhouette score.

## Related tools

- **KMDHierarchicalClustering** (Provides the KMDClustering scikit-learn-style API whose hyperparameters (k, n_clusters, min_cluster_size, affinity, certainty, k_scan_range, sub_sample, percent_size, seed) this skill configures) — github.com/KaplanLab/KMDHierarchicalClustering
- **SciPy** (Underlying implementation of KMD clustering (python/scipy); affinity metrics reuse scipy.spatial.distance.pdist)
- **scikit-learn** (Dependency of KMDHierarchicalClustering (sklearn>=0.23.1); KMDClustering usage follows the scikit-learn interface)

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd_cluster = KMDClustering(k='compute', n_clusters=2, affinity='correlation', k_scan_range=(1,100,5)); kmd_cluster.fit(X, sub_sample=True, percent_size=0.2, seed=1)
```

## Evaluation signals

- With k='compute', the returned best k should fall within k_scan_range and correspond to the highest KMD silhouette score (k limited to < 100 as in the paper)
- On the cytometry benchmark (sub_sample=True, 10 resamples of 20,000 cells, seed=1, min_cluster_size=50, affinity='correlation', k_scan_range=(1,100,5)), Levine15_32 KMD accuracy should be approximately 0.948
- Averaged over Levine15_13, Levine15_32 and Samusik16, accuracy/NMI/ARI should be ~0.894/0.879/0.892, outperforming PhenoGraph (0.834/0.848/0.841)
- min_cluster_size behavior check: clusters smaller than the threshold must be labeled outliers, and certainty=0.5 leaves all outliers unassigned while certainty=1 assigns all to core clusters
- Package must run on Python 3.x (tested on 3.6.5 and 3.7) with numpy>=1.18.4, scipy>=1.3.0, sklearn>=0.23.1 installed

## Limitations

- Automatic selection of the hyperparameter k requires running the clustering with a large range of k values, requiring more computational resources
- The python/scipy implementation results in longer running times than widely used clustering algorithms implemented with optimized, non-interpreted code
- The sampling-based extension may miss very small clusters when they are not sampled well
- Default k_scan_range jumps of 3 vs the paper's 5 may change which k is selected; the README recommends default values unless there is a specific reason to change them
- The package is not expected to work with Python 2.x

## Evidence

- [readme] KMD = KMDClustering(k='compute', n_clusters = 2, min_cluster_size = 'compute', affinity = 'euclidean', certainty = 0.5 ,k_scan_range = (1,100,3), y_true = [], plot_scores=False, path=False): "KMD = KMDClustering(k='compute', n_clusters = 2, min_cluster_size = 'compute', affinity = 'euclidean', certainty = 0.5 ,k_scan_range = (1,100,3), y_true = [], plot_scores=False, path=False)"
- [readme] The main parameter that users should set is n_clusters. We recommend using default values for the rest of the parameters unless there is a specific reason to change these.: "The main parameter that users should set is `n_clusters`. We recommend using default values for the rest of the parameters unless there is a specific reason to change these."
- [intro] we eliminate hyperparameter k by running the clustering in parallel over several values of k and picking the k value that has the highest KMD silhouette score: "we eliminate hyperparameter k by running the clustering in parallel over several values of k and picking the k value that has the highest KMD silhouette score"
- [readme] sub_sample: If True ,dataset will be subsampled and a post hoc cluster inffering method will be conducted, reccomended on large dataset: "`sub_sample`: If True ,dataset will be subsampled and a post hoc cluster inffering method will be conducted, reccomended on large dataset"
- [discussion] tiny clusters which are smaller than a size threshold can be considered to be outliers: "tiny clusters which are smaller than a size threshold can be considered to be outliers"
