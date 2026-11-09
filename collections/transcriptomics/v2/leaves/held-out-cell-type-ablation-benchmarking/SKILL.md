---
name: held-out-cell-type-ablation-benchmarking
description: Use when evaluating whether a Symphony reference mapping flags query
  cell populations absent from the reference as novel, or when reproducing the ablation
  benchmark of per-cell/per-cluster mapping-confidence metrics (AUCs, Supplementary
  Fig. 9).
when_to_use_negative:
- When the query's novel population is very similar to an existing reference population
  — the article states the metrics may be less informative in that case
- When the goal is de novo cell type discovery from the query alone — Symphony does
  not supplant de novo query-only analyses
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Symphony
  role: Maps ablated query cells into the reference embedding (mapQuery) and provides
    the per-cell/per-cluster Mahalanobis mapping metrics being benchmarked
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Integrates reference datasets during reference construction prior to compression
    into the Symphony reference used in the ablation benchmark
  repo: https://github.com/immunogenomics/symphony
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:09441ac70f0ad725addd0d8cd0764e89bbdb191459463a7e31e1d5610f29fd74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# held-out-cell-type-ablation-benchmarking

## Summary

Benchmark Symphony's Mahalanobis-distance-based per-cell and per-cluster mapping metrics by ablating (holding out) query cell types and testing whether the metrics flag the missing populations as novel, quantified by AUC against the reference cell types.

## When to use

Use when evaluating whether a Symphony reference mapping flags query cell populations absent from the reference as novel, or when reproducing the ablation benchmark of per-cell/per-cluster mapping-confidence metrics (AUCs, Supplementary Fig. 9).

## When NOT to use

- When the query's novel population is very similar to an existing reference population — the article states the metrics may be less informative in that case
- When the goal is de novo cell type discovery from the query alone — Symphony does not supplant de novo query-only analyses

## Inputs

- Symphony reference object(s) (compressed Harmony-derived reference with Z_orig, loadings s$u, gene means/sds)
- Ablated/holding-out query expression matrices (e.g. fetal liver non-immune, myeloid, Kupffer ablations; PBMC held-out-type scenarios)
- Query cluster labels (query_cluster_labels) for per-cluster aggregation

## Outputs

- Per-cell Mahalanobis mapping metric values for every query cell
- Per-cluster mapping metric values per query cluster
- AUC values for held-out cell-type detection per scenario (e.g. mapping_metric_aucs.csv)

## How to apply

Load the Symphony reference objects together with ablated (held-out) query expression matrices — e.g. fetal liver non-immune, myeloid, and Kupffer cell ablations, or PBMC held-out-type scenarios. Map each held-out query into the reference embedding using mapQuery (the reference was built via vargenes_vst variable-gene selection, scaleDataWithStats scaling, SVD PCA, and Harmony integration with return_object = TRUE, then compressed with buildReferenceFromHarmonyObj). Compute the per-cell Mahalanobis-distance mapping metric for every query cell with calcPerCellMappingMetric(reference, query, Z_orig, metric), then aggregate to clusters with calcPerClusterMappingMetric(reference, query, query_cluster_labels, metric, u, lambda). Rank the reference cell types by the metric and compute AUCs for held-out-type detection with evaluate; write results such as non-immune (0.997 per-cell / 1.0 per-cluster) and Kupffer (0.633 / 0.963) to mapping_metric_aucs.csv.

## Related tools

- **Symphony** (Maps ablated query cells into the reference embedding (mapQuery) and provides the per-cell/per-cluster Mahalanobis mapping metrics being benchmarked) — https://github.com/immunogenomics/symphony
- **Harmony** (Integrates reference datasets during reference construction prior to compression into the Symphony reference used in the ablation benchmark) — https://github.com/immunogenomics/symphony

## Examples

```
query = mapQuery(query_exp, reference, query_metadata, vars = c('donor')); per_cell = calcPerCellMappingMetric(reference, query, query$Z_orig, metric)
```

## Evaluation signals

- Reproduced non-immune ablation AUCs approach the published values: per-cell AUC = 0.997 and per-cluster AUC = 1.0 (Supplementary Fig. 9)
- Reproduced Kupffer ablation AUCs match per-cell 0.633 / per-cluster 0.963 within tolerance
- Held-out cell types rank first when reference cell types are ordered by the mapping metric (high detection AUC)
- Per-cell metric computed via calcPerCellMappingMetric uses the reference's Z_orig coordinates; per-cluster aggregation uses specified u and lambda parameters
- Outputs are written as a mapping_metric_aucs.csv with per-scenario per-cell and per-cluster AUC columns

## Limitations

- Mapping metrics may be less informative when a novel population is very similar to an existing reference population
- Identification of novel cell types that have failed to map remains an unsolved future direction for mapping algorithms
- Symphony does not entirely supplant the need for users interested in novel cell type discovery to conduct de novo analyses of the query alone

## Evidence

- [discussion] Identification of novel cell-types that have failed to map is an important future direction for mapping algorithms.: "Identification of novel cell-types that have failed to map is an important future direction for mapping algorithms."
- [discussion] Metrics less informative for similar novel populations: [evidence span withheld — anchor status: not_found]
- [methods] Symphony reference construction: "In this tutorial, we will construct a Symphony reference from two PBMC datasets"
- [abstract] Stable low-dimensional embedding: [evidence span withheld — anchor status: not_found]
- [methods] mapQuery filter step: [evidence span withheld — anchor status: not_found]
