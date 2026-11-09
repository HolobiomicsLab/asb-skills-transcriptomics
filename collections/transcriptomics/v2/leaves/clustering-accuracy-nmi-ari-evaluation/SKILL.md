---
name: clustering-accuracy-nmi-ari-evaluation
description: Use when cluster assignments from one or more clustering algorithms must
  be scored against known ground-truth cell labels, to produce a comparable accuracy/NMI/ARI
  benchmark table across methods.
when_to_use_negative:
- No ground-truth labels exist for the dataset — NMI/ARI/accuracy cannot be computed;
  use intrinsic scores (KMD silhouette, DBCV) instead
- You need an intrinsic cluster-quality criterion rather than external validation
  — note the paper warns that the silhouette score is biased towards globular cluster
  shapes and is not adequate as an intrinsic evaluation function
- Input is mass cytometry or simulated data with a different ground-truth annotation
  pipeline (e.g. Levine_32dim/FITNESS-style preprocessing with unassigned or ambiguous
  cells already discarded) — the metric recipe here is calibrated to the scRNA-seq
  benchmark setup
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: KMDHierarchicalClustering
  role: Clustering method under evaluation; fitted with affinity='correlation', min_cluster_size=10,
    k='compute' to generate labels scored against ground truth
  repo: https://github.com/KaplanLab/KMDHierarchicalClustering
- name: SCCAF
  role: Baseline scRNA-seq clusterer run on identical preprocessed matrices for accuracy/NMI/ARI
    comparison
- name: Scanpy
  role: Provides the Scanpy Louvain and Scanpy Leiden baseline clusterers scored in
    the benchmark table
- name: Seurat
  role: Baseline clusterer (Seurat Louvain) run on the identical preprocessed matrices
- name: hkariti/DBCV
  role: Density-based clustering validation used to select the number of clusters
    (2–14) that maximizes DBCV before labels are scored
  repo: https://github.com/hkariti/DBCV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:348f697f92e8cb1f95bd2c2c5adf2906d03e46ae2825216cabef575d88351ab2
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# clustering-accuracy-nmi-ari-evaluation

## Summary

Benchmarks cluster assignments against ground-truth cell labels using accuracy, normalized mutual information (NMI) and adjusted Rand index (ARI), as done in the KMD clustering paper when comparing against SCCAF, Scanpy Louvain/Leiden and Seurat on Lawlor17, Zeisel15 and Li17 scRNA-seq data.

## When to use

Use when cluster assignments from one or more clustering algorithms must be scored against known ground-truth cell labels, to produce a comparable accuracy/NMI/ARI benchmark table across methods.

## When NOT to use

- No ground-truth labels exist for the dataset — NMI/ARI/accuracy cannot be computed; use intrinsic scores (KMD silhouette, DBCV) instead
- You need an intrinsic cluster-quality criterion rather than external validation — note the paper warns that the silhouette score is biased towards globular cluster shapes and is not adequate as an intrinsic evaluation function
- Input is mass cytometry or simulated data with a different ground-truth annotation pipeline (e.g. Levine_32dim/FITNESS-style preprocessing with unassigned or ambiguous cells already discarded) — the metric recipe here is calibrated to the scRNA-seq benchmark setup

## Inputs

- Preprocessed single-cell count matrices with ground-truth cell labels (Lawlor17 from GEO GSE86469, Zeisel15 from GEO GSE71585, Li17 from GEO GSE81861)
- Cluster label vectors produced by each compared algorithm (KMD, SCCAF, Scanpy Louvain, Scanpy Leiden, Seurat) on the same preprocessed matrix
- Cluster-number selections (optimal number of clusters between 2 and 14 chosen by DBCV; k chosen by KMD silhouette)

## Outputs

- scrna_kmd_benchmark_table.csv with accuracy, NMI and ARI per method per dataset
- Best-matching accuracy per method against ground-truth labels (e.g. KMD 0.893 vs SCCAF 0.808 on Lawlor17)

## How to apply

Run every clusterer (KMDHierarchicalClustering with affinity='correlation', min_cluster_size=10, k='compute', plus baselines SCCAF, Scanpy Louvain, Scanpy Leiden and Seurat) on the identical preprocessed matrices — e.g. Lawlor17 (GSE86469) filtered for cells with under 200 expressed genes and genes expressed in fewer than 3 cells, Zeisel15 restricted to the 2000 highest-variance genes. For KMD, first scan k (k < 100) and pick the k with the highest KMD silhouette score, then select the number of clusters between 2 and 14 that maximizes the DBCV score (hkariti/DBCV implementation) after outlier removal, re-assigning outliers to core clusters at the optimal k. Compute accuracy, NMI and ARI of each method's labels against the ground-truth cell labels, since NMI and ARI are label-permutation-invariant while accuracy requires the best label matching. Write all scores to scrna_kmd_benchmark_table.csv and sanity-check against the published values (KMD: 0.893 accuracy, 0.790 NMI, 0.831 ARI on Lawlor17; 0.838 accuracy on Li17 vs 0.729 for Louvain).

## Related tools

- **KMDHierarchicalClustering** (Clustering method under evaluation; fitted with affinity='correlation', min_cluster_size=10, k='compute' to generate labels scored against ground truth) — https://github.com/KaplanLab/KMDHierarchicalClustering
- **SCCAF** (Baseline scRNA-seq clusterer run on identical preprocessed matrices for accuracy/NMI/ARI comparison)
- **Scanpy** (Provides the Scanpy Louvain and Scanpy Leiden baseline clusterers scored in the benchmark table)
- **Seurat** (Baseline clusterer (Seurat Louvain) run on the identical preprocessed matrices)
- **hkariti/DBCV** (Density-based clustering validation used to select the number of clusters (2–14) that maximizes DBCV before labels are scored) — https://github.com/hkariti/DBCV

## Examples

```
from KMDHierarchicalClustering import KMDClustering; kmd = KMDClustering(k='compute', n_clusters=2, min_cluster_size=10, affinity='correlation'); kmd.fit(X)
```

## Evaluation signals

- KMD accuracy on Lawlor17 is approximately 0.893 (0.790 NMI, 0.831 ARI), with SCCAF second best at 0.808 accuracy
- KMD accuracy on Li17 is approximately 0.838, exceeding Scanpy Louvain's 0.729
- All methods are scored on identical preprocessed matrices (Lawlor17 gene/cell filtering, Zeisel15 2000 highest-variance genes), so metric differences reflect clusterer choices only
- Benchmark rows cover KMD plus all four baselines (SCCAF, Scanpy Louvain, Scanpy Leiden, Seurat) with accuracy, NMI and ARI columns populated per dataset
- The DBCV-selected number of clusters falls within the searched range of 2 to 14, and k was chosen from the k < 100 scan by highest KMD silhouette

## Limitations

- The silhouette score is biased towards globular cluster shapes and thus not adequate as an intrinsic evaluation function; DBCV-based de novo cluster number estimation is only preliminary
- No software version numbers are reported for the compared tools, and no version is specified for KMDHierarchicalClustering (installed via pip from GitHub), so exact reproducibility of metric values may vary
- The article notes that further research is needed to improve cluster number estimation before relying on DBCV selection alone

## Evidence

- [abstract] Rigorous comparisons to general and specialized clustering methods on simulated, mass cytometry and scRNA-seq datasets show consistent high performance of KMD clustering across all datasets.: "Rigorous comparisons to general and specialized clustering methods on simulated, mass cytometry and scRNA-seq datasets show consistent high performance of KMD clustering across all datasets."
- [discussion] Cluster number selection by DBCV is the decision point that determines the labels later scored with accuracy/NMI/ARI.: "we calculate the DBCV (after removing outliers) of the solution with the k value selected by the KMD silhouette, and choose the number of clusters maximizes the DBCV score."
- [readme] The silhouette value measures how well an object is classified in its own cluster instead of neighboring clusters.: [evidence span withheld — anchor status: external_doc]
- [intro] The benchmark baseline toolset includes SCCAF, Scanpy Louvain/Leiden and Seurat, which the article names as specialized scRNA-seq clusterers compared against KMD.: "other implementations include Scanpy14 Louvain and Seurat15 Louvain and Bluster16 Louvain"
