---
name: per-cluster-metric-aggregation
description: Use when mapping a query single-cell dataset onto a Symphony reference
  and you need to detect query cell populations (clusters) that are absent from the
  reference, e.g. after holding out or ablating a cell type.
when_to_use_negative:
- When the novel query population is very similar to an existing reference population
  — the paper states these metrics 'may be less informative in cases where the novel
  population is very similar to an existing reference population'
- When the goal is de novo discovery of novel cell types rather than flagging non-mapping
  populations — Symphony 'does not entirely supplant the need for users interested
  in novel cell type discovery to conduct de novo analyses of the query alone'
- When the query already has full cell-type coverage in the reference (no held-out/ablated
  types), since detection of missing types is the target task
tools:
- name: Symphony
  role: Provides mapQuery, calcPerCellMappingMetric, and calcPerClusterMappingMetric
    for computing and aggregating Mahalanobis-based mapping-confidence metrics
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Underpins reference integration; the reference is a compressed Harmony object
    built with return_object = TRUE before query mapping
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:da79a3c8be9da56f844d54fa5498e03849a09014a6caf774ab2d645027d57617
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# per-cluster-metric-aggregation

## Summary

Aggregate Symphony's per-cell Mahalanobis-distance-based mapping metrics into per-cluster metrics (calcPerClusterMappingMetric) to flag query clusters that represent cell types missing from the reference, benchmarking detection with AUCs for held-out types.

## When to use

Use when mapping a query single-cell dataset onto a Symphony reference and you need to detect query cell populations (clusters) that are absent from the reference, e.g. after holding out or ablating a cell type.

## When NOT to use

- When the novel query population is very similar to an existing reference population — the paper states these metrics 'may be less informative in cases where the novel population is very similar to an existing reference population'
- When the goal is de novo discovery of novel cell types rather than flagging non-mapping populations — Symphony 'does not entirely supplant the need for users interested in novel cell type discovery to conduct de novo analyses of the query alone'
- When the query already has full cell-type coverage in the reference (no held-out/ablated types), since detection of missing types is the target task

## Inputs

- Symphony reference object (built via buildReference or buildReferenceFromHarmonyObj)
- Query expression matrix for the ablated/held-out cell type scenario (e.g. fetal liver non-immune, myeloid, or Kupffer ablations; PBMC held-out-type scenarios)
- Per-cell mapping metric values from calcPerCellMappingMetric (with Z_orig)
- Query cluster labels (query_cluster_labels)

## Outputs

- Per-cluster mapping-confidence scores per reference cell type
- AUC values quantifying held-out-type detection (e.g. non-immune: 0.997 per-cell / 1.0 per-cluster; Kupffer: 0.633 per-cell / 0.963 per-cluster)
- CSV of reproduced AUCs (mapping_metric_aucs.csv)

## How to apply

First map the held-out/ablated query expression matrix into the reference embedding with Symphony's mapQuery, then compute the per-cell Mahalanobis-distance-based metric for every query cell with calcPerCellMappingMetric(reference, query, Z_orig, metric). Aggregate to the per-cluster level with calcPerClusterMappingMetric(reference, query, query_cluster_labels, metric, u, lambda), where query_cluster_labels supplies cluster assignments and u/lambda control the aggregation. Rank reference cell types by the resulting metric and compute AUCs for held-out-type detection with evaluate (e.g. non-immune fetal liver cells mapped to an immune-only reference yield per-cell AUC = 0.997 and per-cluster AUC = 1.0, Supplementary Fig. 9). Decision point: trust the metric only for very dissimilar novel populations — the paper cautions the metrics are less informative when the novel population closely resembles an existing reference population. Write reproduced AUCs to a CSV (e.g. mapping_metric_aucs.csv) for comparison.

## Related tools

- **Symphony** (Provides mapQuery, calcPerCellMappingMetric, and calcPerClusterMappingMetric for computing and aggregating Mahalanobis-based mapping-confidence metrics) — https://github.com/immunogenomics/symphony
- **Harmony** (Underpins reference integration; the reference is a compressed Harmony object built with return_object = TRUE before query mapping)

## Examples

```
query = mapQuery(query_exp, query_meta, reference, vargenes_means_sds); metric = calcPerCellMappingMetric(reference, query, Z_orig, metric); cluster_metrics = calcPerClusterMappingMetric(reference, query, query_cluster_labels, metric, u, lambda)
```

## Evaluation signals

- Per-cell and per-cluster AUCs for held-out-type detection should approach published values: non-immune cells onto immune-only reference give AUC = 0.997 (per-cell) and 1.0 (per-cluster); Kupffer ablation gives 0.633 (per-cell) and 0.963 (per-cluster), matching Supplementary Fig. 9
- Per-cluster metric values should be a deterministic aggregate of per-cell Mahalanobis metrics within each cluster label, parameterized by u and lambda
- The missing (held-out) cluster should rank first against reference cell types when reference and query cell states are very different
- Output file mapping_metric_aucs.csv should contain one AUC per (scenario, granularity) pair and be reproducible across runs

## Limitations

- Identification of novel cell types that have failed to map is described as an unsolved future direction for mapping algorithms
- Metrics may be less informative when the novel population is very similar to an existing reference population
- Symphony does not replace de novo analyses of the query alone for users interested in novel cell type discovery

## Evidence

- [supplementary] When query cell types are very different from the reference (mapping non-immune cells onto an immune-only reference), the mapping metrics clearly distinguish the missing cell states as novel, with per-cell AUC = 0.997 and per-cluster AUC = 1.0: [evidence span withheld — anchor status: not_found]
- [discussion] Metrics are less reliable when a novel population is similar to a reference population: [evidence span withheld — anchor status: not_found]
- [discussion] Novel cell type identification is an open problem: "Identification of novel cell-types that have failed to map is an important future direction for mapping algorithms."
- [other] Workflow uses the named Symphony functions for per-cell and per-cluster metric computation: [evidence span withheld — anchor status: not_found]
