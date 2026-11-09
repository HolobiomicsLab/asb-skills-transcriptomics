---
name: auc-evaluation-of-type-detection
description: Use when query data are expected to contain cell types absent from a
  Symphony reference (e.g., ablated or held-out type scenarios) and you need to quantify
  how well mapping metrics flag those cells as novel rather than misannotating them.
when_to_use_negative:
- When the novel query population is very similar to an existing reference population
  — the metrics may be less informative (e.g., Kupffer per-cell AUC drops to 0.633
  versus 0.997 for very different non-immune cells)
- When the goal is genuine novel cell type discovery rather than benchmarking known
  held-out types — the article cautions that Symphony does not supplant de novo analyses
  of the query alone
- When the reference comprehensively covers all query cell types and no held-out detection
  is needed
tools:
- name: Symphony
  role: Reference building, query mapping via mapQuery, and Mahalanobis-based mapping
    metrics via calcPerCellMappingMetric and calcPerClusterMappingMetric
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Provides the integrated reference embedding that Symphony compresses; must
    run with return_object = TRUE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:8a9f6556ec3a1fc2f0780f259c408300b52e2440519b8caaf1976c8196c869b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# auc-evaluation-of-type-detection

## Summary

Benchmarks Symphony's Mahalanobis-based per-cell and per-cluster mapping-confidence metrics for detecting query cell types that are missing from the reference, quantifying detection performance as AUCs over held-out (ablated) cell-type scenarios.

## When to use

Use when query data are expected to contain cell types absent from a Symphony reference (e.g., ablated or held-out type scenarios) and you need to quantify how well mapping metrics flag those cells as novel rather than misannotating them.

## When NOT to use

- When the novel query population is very similar to an existing reference population — the metrics may be less informative (e.g., Kupffer per-cell AUC drops to 0.633 versus 0.997 for very different non-immune cells)
- When the goal is genuine novel cell type discovery rather than benchmarking known held-out types — the article cautions that Symphony does not supplant de novo analyses of the query alone
- When the reference comprehensively covers all query cell types and no held-out detection is needed

## Inputs

- Symphony reference object (compressed Harmony integration with saved PCA loadings s$u)
- Query expression matrix with held-out/ablated cell types (fetal liver non-immune, myeloid, Kupffer ablations; PBMC held-out-type scenarios)
- Query cluster labels for per-cluster aggregation
- Metric choice (Mahalanobis distance) and aggregation parameters u and lambda

## Outputs

- Per-cell mapping-confidence scores (Mahalanobis-distance-based metric per query cell)
- Per-cluster mapping-confidence scores aggregated by query_cluster_labels
- AUCs for held-out-type detection per scenario, e.g., written to mapping_metric_aucs.csv

## How to apply

First build a Symphony reference (variable gene selection with vargenes_vst, scaling with scaleDataWithStats, SVD-based PCA, Harmony integration with return_object = TRUE, then compression via buildReferenceFromHarmonyObj or buildReference). Map each held-out or ablated query (fetal liver non-immune onto an immune-only reference; myeloid and Kupffer ablations; PBMC held-out-type scenarios) using mapQuery. Compute the per-cell Mahalanobis-distance-based metric with calcPerCellMappingMetric(reference, query, Z_orig, metric), then aggregate to the per-cluster metric with calcPerClusterMappingMetric(reference, query, query_cluster_labels, metric, u, lambda), where u is the saved PCA gene-loadings matrix. Rank reference cell types by the metric and compute AUCs for held-out-type detection with evaluate, writing the results (e.g., non-immune per-cell 0.997 / per-cluster 1.0; Kupffer per-cell 0.633 / per-cluster 0.963) to mapping_metric_aucs.csv. The rationale is that Mahalanobis distances in the reference embedding separate query cells that fall outside reference support; per-cluster aggregation sharpens detection when per-cell scores are noisy.

## Related tools

- **Symphony** (Reference building, query mapping via mapQuery, and Mahalanobis-based mapping metrics via calcPerCellMappingMetric and calcPerClusterMappingMetric) — https://github.com/immunogenomics/symphony
- **Harmony** (Provides the integrated reference embedding that Symphony compresses; must run with return_object = TRUE)

## Examples

```
per_cell = calcPerCellMappingMetric(reference, query, query$Z_orig, metric); per_clust = calcPerClusterMappingMetric(reference, query, labels, metric, u, lambda); write.csv(aucs, 'mapping_metric_aucs.csv')
```

## Evaluation signals

- For very different missing cell states (non-immune cells mapped onto an immune-only reference), expect per-cell AUC ≈ 0.997 and per-cluster AUC = 1.0 (Supplementary Fig. 9)
- For held-out types resembling reference populations (Kupffer ablation), expect weaker but still substantial detection: per-cell AUC ≈ 0.633, per-cluster AUC ≈ 0.963
- Reproduced AUCs for all ablation scenarios should be written to mapping_metric_aucs.csv
- Confirm the reference was compressed from a Harmony object built with return_object = TRUE and that PCA gene loadings (s$u) were saved for per-cluster aggregation
- Check that held-out cell types rank highest among reference cell types when reference types are ordered by the mapping metric

## Limitations

- These metrics may be less informative in cases where the novel population is very similar to an existing reference population
- Identification of novel cell types that have failed to map remains an important unsolved future direction for mapping algorithms
- Symphony does not entirely supplant the need for users interested in novel cell type discovery to conduct de novo analyses of the query alone

## Evidence

- [discussion] metrics less informative for similar populations: [evidence span withheld — anchor status: not_found]
- [discussion] novel-type detection as future direction: "Identification of novel cell-types that have failed to map is an important future direction for mapping algorithms."
- [methods] Harmony integration step for reference: "Run Harmony integration. It is important to set `return_object = TRUE`."
- [readme] Symphony package availability: "Symphony is available on CRAN:

 r
install.packages("symphony")
"
- [methods] query mapping call: "query = mapQuery(query_exp,"
