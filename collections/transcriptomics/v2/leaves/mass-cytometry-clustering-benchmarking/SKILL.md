---
name: mass-cytometry-clustering-benchmarking
description: Use when evaluating a clustering algorithm on mass cytometry data with
  known cell-type ground truth, or when comparing a new method to specialized cytometry
  clustering tools on the Levine15_13/Levine15_32/Samusik16 benchmarks.
when_to_use_negative:
- Input is unlabeled data with no ground-truth population annotations — accuracy/NMI/ARI
  against y_true cannot be computed
- Data is scRNA-seq rather than mass cytometry — use the paper's scRNA-seq benchmarks
  (Li17, Lawlor17, Zeisel15) and their preprocessing/filtering instead
- Dataset is too large to run 10 resamples of 20,000 cells and full k scanning — the
  sampling-based extension with smaller cores (e.g. 5,000 cells) is the appropriate
  approach
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: KMDHierarchicalClustering
  role: Primary clustering method under test; KMD linkage with KMD silhouette k selection
    and outlier-aware partitioning, run via KMDClustering.fit with sub_sample=True
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: Python
  role: Implementation environment for the benchmark harness and score aggregation
    (KMD is implemented in python/scipy)
- name: SciPy
  role: Underlying scipy infrastructure for KMD clustering and linkage computations
- name: FlowSOM
  role: Specialized mass cytometry comparison algorithm; reference scores taken from
    Liu et al.
- name: PhenoGraph
  role: General-purpose shared-neighbor-graph/Louvain comparison algorithm; reference
    scores from Liu et al.
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:c18618893215120354a30985e6aa40f1d39dc5bf527cc9506b51dba085ba8b2a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FlowSOM
  - KMDHierarchicalClustering
  - PhenoGraph
  - Python
  - SCCAF
  - Scanpy
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# mass-cytometry-clustering-benchmarking

## Summary

Benchmarks a general-purpose clustering method (e.g. KMD clustering) against specialized mass cytometry algorithms (Xshift, DEPECHE, Accense, FlowSOM) and general-purpose ones (kmeans, PhenoGraph) on the Levine15_13, Levine15_32 and Samusik16 bone marrow datasets, scoring accuracy, NMI and ARI against ground-truth cell-population labels.

## When to use

Use when evaluating a clustering algorithm on mass cytometry data with known cell-type ground truth, or when comparing a new method to specialized cytometry clustering tools on the Levine15_13/Levine15_32/Samusik16 benchmarks.

## When NOT to use

- Input is unlabeled data with no ground-truth population annotations — accuracy/NMI/ARI against y_true cannot be computed
- Data is scRNA-seq rather than mass cytometry — use the paper's scRNA-seq benchmarks (Li17, Lawlor17, Zeisel15) and their preprocessing/filtering instead
- Dataset is too large to run 10 resamples of 20,000 cells and full k scanning — the sampling-based extension with smaller cores (e.g. 5,000 cells) is the appropriate approach

## Inputs

- Transformed/filtered mass cytometry benchmark matrices (Levine15_13, Levine15_32, Samusik16) from FlowRepository FR-FCM-ZZPH
- Ground-truth cell-population labels (y_true) per dataset
- Published reference scores for kmeans, Xshift, DEPECHE, Accense, FlowSOM and PhenoGraph (Liu et al.)

## Outputs

- Per-dataset mean and standard deviation of accuracy, NMI and ARI for KMD clustering over 10 resamples
- Combined mean/std benchmark score table written to kmd_cytometry_benchmark_scores.csv

## How to apply

Download the transformed/filtered mass cytometry benchmark datasets from FlowRepository accession FR-FCM-ZZPH. For each dataset, run KMDClustering.fit with sub_sample=True on 10 resamples of 20,000 cells (seed=1), min_cluster_size=50, affinity='correlation', and k_scan_range=(1,100,5), letting the highest KMD silhouette score pick the optimal k (k limited to < 100 as in the paper). Cells that were unassigned or had ambiguous annotations were discarded before scoring. For each resample compute accuracy, NMI and ARI against the known ground-truth labels, aggregate to mean and standard deviation per dataset, then merge the KMD scores with the published Liu et al. reference scores for kmeans, Xshift, DEPECHE, Accense, FlowSOM and PhenoGraph. The rationale is that specialized cytometry methods are often not transferable across data types and vary in performance, so KMD's KMD-silhouette-based k selection and outlier-aware partitioning should be tested head-to-head on the same labeled benchmarks.

## Related tools

- **KMDHierarchicalClustering** (Primary clustering method under test; KMD linkage with KMD silhouette k selection and outlier-aware partitioning, run via KMDClustering.fit with sub_sample=True) — github.com/KaplanLab/KMDHierarchicalClustering
- **Python** (Implementation environment for the benchmark harness and score aggregation (KMD is implemented in python/scipy))
- **SciPy** (Underlying scipy infrastructure for KMD clustering and linkage computations)
- **FlowSOM** (Specialized mass cytometry comparison algorithm; reference scores taken from Liu et al.)
- **PhenoGraph** (General-purpose shared-neighbor-graph/Louvain comparison algorithm; reference scores from Liu et al.)

## Examples

```
pip install git+https://github.com/KaplanLab/KMDHierarchicalClustering; kmd_cluster = KMDClustering(min_cluster_size=50, affinity='correlation', k_scan_range=(1,100,5)); kmd_cluster.fit(X, sub_sample=True, seed=1)
```

## Evaluation signals

- kmd_cytometry_benchmark_scores.csv exists and contains mean/std accuracy, NMI and ARI per dataset for KMD plus the six reference algorithms
- Levine15_32 KMD accuracy is approximately 0.948
- KMD achieves the best average performance across the three cytometry datasets (0.894 avg. accuracy, 0.879 avg. NMI, 0.892 avg. ARI)
- KMD outperforms PhenoGraph's reference scores (0.834/0.848/0.841) on the averaged metrics
- Each dataset's KMD score is aggregated over exactly 10 resamples of 20,000 cells with seed=1

## Limitations

- Automatic selection of the hyperparameter k requires running the clustering with a large range of k values, requiring more computational resources
- The python/scipy implementation results in longer running times than widely used clustering algorithms implemented with optimized, non-interpreted code
- A single KMD run is inherently slower than single/average linkage because it must maintain lists of k minimal distances for each pair of clusters
- No software version numbers are reported for the compared tools or for KMDHierarchicalClustering

## Evidence

- [results] Averaged over the three bone marrow mass cytometry benchmark datasets, KMD clustering achieves the best performance on all metrics: [evidence span withheld — anchor status: not_found]
- [discussion] KMD silhouette k selection: "running the clustering across a range of k values (in all cases we limited k < 100) and automatically selecting the solution with the highest KMD silhouette score."
- [discussion] benchmark datasets and their sizes: "The datasets Levine15_13 (167,044 Cells), Levine15_23 (265,627 cells) and Samusik16 (86,864 Cells) are known benchmarking mass cytometry datasets"
- [discussion] data download source: "downloaded from the “flowrepository” repository (http://flowrepository.org/id/FR-FCM-ZZPH)8"
- [intro] specialized algorithms not transferable: "they are often not transferrable to other types of data and their performance can vary significantly even on datasets of the same type"
- [discussion] discard unannotated cells: "Cells that were unassigned or had ambiguous annotations were discarded."
- [readme] KMDClustering usage interface: "kmd_cluster = KMDClustering(n_clusters = 2)
kmd_cluster.fit(X) # X is the dataset matrix"
