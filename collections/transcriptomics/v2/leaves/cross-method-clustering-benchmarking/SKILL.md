---
name: cross-method-clustering-benchmarking
description: Use when comparing a general-purpose clustering algorithm against domain-specific
  scRNA-seq clusterers on datasets with known cell-type labels, to quantify whether
  the general method matches or beats specialized tools.
when_to_use_negative:
- The dataset has no ground-truth cell-type annotations, since accuracy/NMI/ARI against
  known labels are the benchmark's scoring basis
- The goal is clustering itself rather than method comparison — use KMDHierarchicalClustering
  directly without the multi-method evaluation harness
- The data are very large (million-object scale) single-cell datasets, where the python/scipy
  KMD implementation's long runtimes and the need to scan many k values make this
  benchmark setup impractical without the sampling extension
edam_operation: http://edamontology.org/operation_0532
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: KMDHierarchicalClustering
  role: 'Method under test: KMD clustering fitted with affinity=''correlation'', min_cluster_size=10,
    k=''compute'', scanned over k < 100 with KMD silhouette selection and DBCV-based
    cluster-number estimation'
  repo: https://github.com/KaplanLab/KMDHierarchicalClustering
- name: SCCAF
  role: Baseline scRNA-seq clusterer run on the identical preprocessed matrices for
    comparison
- name: Scanpy
  role: Baseline scRNA-seq clusterer providing Louvain and Leiden community-detection
    clustering
- name: Seurat
  role: Baseline scRNA-seq clusterer run on the identical preprocessed matrices
- name: hkariti/DBCV
  role: Optimized DBCV implementation used to score cluster solutions and select the
    number of clusters maximizing DBCV after outlier removal
  repo: https://github.com/hkariti/DBCV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:2b437572a72bcabb39b523c7b3c2114ecc0ebef54edb28edfdee0cc4e06e69dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KaplanLab/KMDHierarchicalClustering
  tools:
  - KMDHierarchicalClustering
  - SCCAF
  - Scanpy
  - Seurat
  - hkariti/DBCV
  edam_operation: http://edamontology.org/operation_0532
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-method-clustering-benchmarking

## Summary

Benchmarks a general-purpose clustering method (KMD clustering with DBCV-based cluster-number estimation) against scRNA-seq-specific clusterers (SCCAF, Scanpy Louvain/Leiden, Seurat) on gold-standard single-cell datasets with ground-truth labels, scoring accuracy, NMI and ARI.

## When to use

Use when comparing a general-purpose clustering algorithm against domain-specific scRNA-seq clusterers on datasets with known cell-type labels, to quantify whether the general method matches or beats specialized tools.

## When NOT to use

- The dataset has no ground-truth cell-type annotations, since accuracy/NMI/ARI against known labels are the benchmark's scoring basis
- The goal is clustering itself rather than method comparison — use KMDHierarchicalClustering directly without the multi-method evaluation harness
- The data are very large (million-object scale) single-cell datasets, where the python/scipy KMD implementation's long runtimes and the need to scan many k values make this benchmark setup impractical without the sampling extension

## Inputs

- Raw scRNA-seq count matrices from GEO: GSE86469 (Lawlor17, 638 human islet cells), GSE71585 (Zeisel15, 3005 mice cerebral cortex cells), GSE81861 (Li17, 630 single cells from 7 cell lines)
- Ground-truth cell-type labels for each dataset
- Log1p-normalized, gene/cell-filtered expression matrices (identical across all methods)

## Outputs

- scrna_kmd_benchmark_table.csv with accuracy, NMI and ARI for KMD and each baseline clusterer per dataset
- Cluster label assignments from KMD and each baseline method on each preprocessed matrix

## How to apply

Download raw count matrices for the gold-standard scRNA-seq datasets from GEO (Lawlor17 GSE86469, Zeisel15 GSE71585, Li17 GSE81861) and preprocess each with log1p normalization plus the dataset-specific filters: for Lawlor17 drop cells with under 200 expressed genes and genes expressed in fewer than 3 cells; for Zeisel15 keep the 2000 highest-variance genes. Fit KMDHierarchicalClustering's KMDClustering with affinity='correlation', min_cluster_size=10 and k='compute' so k is chosen by the highest KMD silhouette score over the scan range (k < 100); select the number of clusters between 2 and 14 maximizing the hkariti/DBCV score after outlier removal, then assign outliers to core clusters at the optimal k. Run SCCAF, Scanpy Louvain, Scanpy Leiden and Seurat on the identical preprocessed matrices so all methods see the same input. Compute accuracy, NMI and ARI against ground-truth cell labels for every method and write the comparison table.

## Related tools

- **KMDHierarchicalClustering** (Method under test: KMD clustering fitted with affinity='correlation', min_cluster_size=10, k='compute', scanned over k < 100 with KMD silhouette selection and DBCV-based cluster-number estimation) — https://github.com/KaplanLab/KMDHierarchicalClustering
- **SCCAF** (Baseline scRNA-seq clusterer run on the identical preprocessed matrices for comparison)
- **Scanpy** (Baseline scRNA-seq clusterer providing Louvain and Leiden community-detection clustering)
- **Seurat** (Baseline scRNA-seq clusterer run on the identical preprocessed matrices)
- **hkariti/DBCV** (Optimized DBCV implementation used to score cluster solutions and select the number of clusters maximizing DBCV after outlier removal) — https://github.com/hkariti/DBCV

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd = KMDClustering(k='compute', min_cluster_size=10, affinity='correlation', k_scan_range=(1,100,3)); kmd.fit(X)
```

## Evaluation signals

- KMD accuracy on Lawlor17 is approximately 0.893 (NMI 0.790, ARI 0.831), with SCCAF second best at 0.808 accuracy
- KMD accuracy on Li17 is approximately 0.838, exceeding Scanpy Louvain at 0.729
- The number of clusters chosen by DBCV lies within the 2–14 range scanned after outlier removal
- All methods (KMD, SCCAF, Louvain, Leiden, Seurat) are evaluated on byte-identical preprocessed matrices, so metric differences are attributable to the clusterers
- The k selected by KMD silhouette falls inside the scanned range (k < 100)

## Limitations

- Automatic selection of hyperparameter k requires running clustering over a large range of k values, consuming more computational resources
- The python/scipy implementation results in longer running times than widely used, non-interpreted clustering algorithms, and a single KMD run is inherently slower because it maintains lists of k minimal distances for each pair of clusters
- The sampling-based extension may miss very small clusters that are not sampled well
- De novo cluster-number estimation via DBCV is only preliminary; the authors state further research is needed to improve cluster number estimation

## Evidence

- [discussion] KMD accuracy and DBCV cluster-number selection: "we calculate the DBCV (after removing outliers) of the solution with the k value selected by the KMD silhouette, and choose the number of clusters maximizes the DBCV score."
- [discussion] k scan range and KMD silhouette selection: "running the clustering across a range of k values (in all cases we limited k < 100) and automatically selecting the solution with the highest KMD silhouette score."
- [methods] Package installation method: "pip install git+https://github.com/KaplanLab/KMDHierarchicalClustering"
- [readme] KMDClustering parameter surface: "KMD = KMDClustering(k='compute', n_clusters = 2, min_cluster_size = 'compute', affinity = 'euclidean', certainty = 0.5 ,k_scan_range = (1,100,3)"
- [readme] Prepackaged benchmark notebooks per dataset: "Lawlor17_dataset.ipynb - Evaluates performance on a single cell dataset - - expected run time: ~20 seconds"
