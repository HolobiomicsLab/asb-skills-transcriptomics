---
name: external-reference-score-integration
description: Use when you have computed performance metrics for your method on a shared
  benchmark dataset and need to integrate them with previously published scores of
  competing algorithms so all methods can be compared on identical data and metrics.
when_to_use_negative:
- The reference study used different dataset versions or preprocessing than yours
  — merged scores would not be comparable
- You need to re-evaluate the reference algorithms themselves (e.g. with new parameters)
  rather than integrate their published scores
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: KMDHierarchicalClustering
  role: Produces the KMD clustering accuracy/NMI/ARI scores that are merged with the
    external reference scores
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: Python
  role: Implementation environment for computing and merging the score tables
- name: SciPy
  role: Underlying library of the KMD implementation whose scores are integrated with
    the published reference values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:7f0defccc81c117ae93a8e74f63c4f96a78758377698470e5fc3c488ad743d83
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KMDHierarchicalClustering
  - Python
  - SciPy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# external-reference-score-integration

## Summary

Merge newly computed benchmark scores (accuracy, NMI, ARI) for a method such as KMD clustering with published reference scores from prior benchmark studies (Liu et al. scores for Xshift, DEPECHE, Accense, FlowSOM, kmeans, PhenoGraph) into a single comparison table.

## When to use

Use when you have computed performance metrics for your method on a shared benchmark dataset and need to integrate them with previously published scores of competing algorithms so all methods can be compared on identical data and metrics.

## When NOT to use

- The reference study used different dataset versions or preprocessing than yours — merged scores would not be comparable
- You need to re-evaluate the reference algorithms themselves (e.g. with new parameters) rather than integrate their published scores

## Inputs

- Per-resample clustering evaluation metrics (accuracy, NMI, ARI) computed on benchmark mass cytometry datasets (Levine15_13, Levine15_23, Samusik16) from FlowRepository FR-FCM-ZZPH
- Published reference benchmark scores from Liu et al. for kmeans, Xshift, DEPECHE, Accense, FlowSOM and PhenoGraph on the same datasets

## Outputs

- Combined mean/std score table (CSV, e.g. kmd_cytometry_benchmark_scores.csv) with accuracy, NMI and ARI for KMD and all reference algorithms per dataset
- Aggregated cross-dataset averages (e.g. KMD 0.894 avg. accuracy, 0.879 avg. NMI, 0.892 avg. ARI vs PhenoGraph 0.834/0.848/0.841)

## How to apply

Run your method (here, KMDClustering.fit with sub_sample=True, seed=1, min_cluster_size=50, affinity='correlation', k_scan_range=(1,100,5)) on the same preprocessed benchmark data used by the reference study — the transformed/filtered mass cytometry datasets from FlowRepository FR-FCM-ZZPH. Compute per-resample accuracy, NMI and ARI against ground-truth labels, then aggregate to mean and standard deviation per dataset (e.g. Levine15_13, Levine15_23, Samusik16; ~20,000-cell subsamples, 10 resamples). Retrieve the published per-algorithm scores from the reference benchmark (Liu et al.) rather than re-running those tools, ensuring the same metric definitions and dataset versions are used. Merge the KMD scores with the reference scores for kmeans, Xshift, DEPECHE, Accense, FlowSOM and PhenoGraph into one table with mean/std per metric per dataset, and write it to a CSV (e.g. kmd_cytometry_benchmark_scores.csv). Validate the integration by checking that reproduced scores match known anchor values (Levine15_32 KMD accuracy ≈ 0.948) and that aggregated averages are internally consistent.

## Related tools

- **KMDHierarchicalClustering** (Produces the KMD clustering accuracy/NMI/ARI scores that are merged with the external reference scores) — github.com/KaplanLab/KMDHierarchicalClustering
- **Python** (Implementation environment for computing and merging the score tables)
- **SciPy** (Underlying library of the KMD implementation whose scores are integrated with the published reference values)

## Examples

```
KMD = KMDClustering(k='compute', min_cluster_size=50, affinity='correlation', k_scan_range=(1,100,5)); KMD.fit(X, sub_sample=True, percent_size=20000, seed=1)
```

## Evaluation signals

- Levine15_32 (Levine_32dim) KMD accuracy is approximately 0.948 after reproduction
- Averaged over the three bone marrow cytometry datasets, KMD achieves the best performance on all metrics (0.894 avg. accuracy, 0.879 avg. NMI, 0.892 avg. ARI), above PhenoGraph (0.834/0.848/0.841)
- The output CSV contains mean and standard deviation per metric per dataset for all seven algorithms (KMD plus kmeans, Xshift, DEPECHE, Accense, FlowSOM, PhenoGraph)
- Reference algorithm scores match the published Liu et al. values exactly (not recomputed), confirming they were integrated rather than re-run

## Limitations

- Reference scores are taken as published; if the reference benchmark used different preprocessing or metric implementations, the comparison may be biased
- KMD scores depend on the resampling scheme (10 resamples of 20,000 cells, seed=1); different subsample sizes or seeds will shift the reproduced scores
- No software version numbers are reported for the compared tools, limiting reproducibility of the merged comparison

## Evidence

- [discussion] Workflow step: merge KMD scores with published reference scores: [evidence span withheld — anchor status: not_found]
- [discussion] Benchmark data source and preprocessing provenance: "downloaded from the “flowrepository” repository (http://flowrepository.org/id/FR-FCM-ZZPH)8"
- [abstract] Averaged KMD performance over the three benchmark datasets: "Rigorous comparisons to general and specialized clustering methods on simulated, mass cytometry and scRNA-seq datasets show consistent high performance"
- [methods] KMD fit invocation used to generate the scores: "kmd_cluster.fit(X) # X is the dataset matrix"
- [discussion] K-scan range constraint applied during score reproduction: "running the clustering across a range of k values (in all cases we limited k < 100) and automatically selecting the solution with the highest KMD silhouette score."
