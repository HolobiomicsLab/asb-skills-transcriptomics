---
name: dbcv-cluster-number-selection
description: Use when clustering biological data (e.g. scRNA-seq count matrices) with
  KMDHierarchicalClustering and the true number of clusters is unknown, so it must
  be estimated de novo from the data.
when_to_use_negative:
- When the number of clusters is already known a priori — just set n_clusters directly
  instead of scanning
- When clusters are expected to be very small and rare — the sampling-based KMD extension
  may miss clusters that are not sampled well, corrupting DBCV estimates
- When silhouette-type scores are adequate (globular clusters) and DBCV's density-based
  machinery is unnecessary overhead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: KMDHierarchicalClustering
  role: Produces KMD clustering solutions over a scan of k values with outlier-aware
    partitioning; k='compute' triggers selection by KMD silhouette
  repo: https://github.com/KaplanLab/KMDHierarchicalClustering
- name: hkariti/DBCV
  role: Optimized Python implementation of DBCV used to score cluster solutions after
    outlier removal and select the cluster number
  repo: https://github.com/hkariti/DBCV
- name: christopherjenness/DBCV
  role: Original DBCV implementation that hkariti/DBCV optimizes; validates non-globular
    cluster assignments
  repo: https://github.com/christopherjenness/DBCV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:b9b3e74c626a93fb33388095fe817aac4cf90a04e1617dd4dbfc99cbda258b23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KaplanLab/KMDHierarchicalClustering
  tools:
  - KMDHierarchicalClustering
  - SCCAF
  - Scanpy
  - Seurat
  - christopherjenness/DBCV
  - hkariti/DBCV
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# dbcv-cluster-number-selection

## Summary

Selects the number of clusters in KMD clustering by maximizing the DBCV (Density-Based Clustering Validation) score, computed after outlier removal, on solutions whose k hyperparameter was chosen by the KMD silhouette score.

## When to use

Use when clustering biological data (e.g. scRNA-seq count matrices) with KMDHierarchicalClustering and the true number of clusters is unknown, so it must be estimated de novo from the data.

## When NOT to use

- When the number of clusters is already known a priori — just set n_clusters directly instead of scanning
- When clusters are expected to be very small and rare — the sampling-based KMD extension may miss clusters that are not sampled well, corrupting DBCV estimates
- When silhouette-type scores are adequate (globular clusters) and DBCV's density-based machinery is unnecessary overhead

## Inputs

- Preprocessed scRNA-seq count matrix (log1p-normalized, gene/cell filtered), e.g. Lawlor17 (GSE86469), Zeisel15 (GSE71585), Li17 (GSE81861)
- KMD clustering solutions generated across a range of k values (k_scan_range, k < 100)
- min_cluster_size threshold (e.g. 10) defining which tiny clusters are treated as outliers

## Outputs

- Final cluster label assignments per cell at the DBCV-optimal cluster number (2–14 scanned)
- Outlier re-assignments to core clusters at the optimal k
- Benchmark metrics (accuracy, NMI, ARI) against ground-truth labels, e.g. scrna_kmd_benchmark_table.csv

## How to apply

Run KMD clustering over a scan range of k values (k < 100 in the paper) and pick the k whose KMD silhouette score is highest. For that k, remove outliers (tiny clusters smaller than min_cluster_size, e.g. 10) and compute the DBCV score using the hkariti/DBCV Python implementation (an optimized fork of christopherjenness/DBCV, which validates non-globular cluster shapes where the standard silhouette fails). Scan candidate numbers of clusters between 2 and 14 and choose the partition that maximizes the DBCV score; re-assign removed outliers to core clusters at the optimal k. Rationale: DBCV scores density within versus between clusters and tolerates non-globular shapes, unlike silhouette which is biased towards globular clusters.

## Related tools

- **KMDHierarchicalClustering** (Produces KMD clustering solutions over a scan of k values with outlier-aware partitioning; k='compute' triggers selection by KMD silhouette) — https://github.com/KaplanLab/KMDHierarchicalClustering
- **hkariti/DBCV** (Optimized Python implementation of DBCV used to score cluster solutions after outlier removal and select the cluster number) — https://github.com/hkariti/DBCV
- **christopherjenness/DBCV** (Original DBCV implementation that hkariti/DBCV optimizes; validates non-globular cluster assignments) — https://github.com/christopherjenness/DBCV

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd = KMDClustering(k='compute', min_cluster_size=10, affinity='correlation'); kmd.fit(X)
```

## Evaluation signals

- On Lawlor17, the pipeline should reproduce KMD accuracy ≈ 0.893 (NMI 0.790, ARI 0.831); on Li17, accuracy ≈ 0.838 (vs 0.729 for Louvain)
- The chosen cluster number must lie in the scanned range 2–14 and correspond to the DBCV maximum computed after outlier removal
- The k fed into DBCV evaluation must equal the k with the highest KMD silhouette score over the scan range (k < 100)
- Outliers (clusters smaller than min_cluster_size) must be removed before scoring DBCV and re-assigned afterwards using KMD linkage at the optimal k
- DBCV README sanity check: high density within clusters and low density between clusters should yield a high DBCV score on a known toy dataset (e.g. noisy moons)

## Limitations

- De novo cluster number estimation via DBCV is only preliminary; the authors state further research is needed to improve cluster number estimation
- Automatic k selection requires running clustering over a large range of k values, consuming more computational resources; the python/scipy implementation runs slower than optimized alternatives
- The sampling-based extension may miss very small clusters that are not sampled well
- Standard silhouette is biased towards globular cluster shapes, which is why DBCV — not silhouette — is used for the final cluster-number decision

## Evidence

- [discussion] DBCV selection rationale: "we calculate the DBCV (after removing outliers) of the solution with the k value selected by the KMD silhouette, and choose the number of clusters maximizes the DBCV score."
- [discussion] Why DBCV over silhouette: "The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate."
- [discussion] Outlier definition: "tiny clusters which are smaller than a size threshold can be considered to be outliers"
- [discussion] DBCV implementation provenance: [evidence span withheld — anchor status: not_found]
- [readme] DBCV scoring principle: [evidence span withheld — anchor status: external_doc]
- [discussion] Preliminary nature of DBCV estimation: "The initial results seem potentially promising based on this strategy, but further research is needed to improve cluster number estimation."
- [discussion] Outlier re-assignment step: "outlier re-assignment is based on KMD linkage matching the optimal k value found during clustering."
