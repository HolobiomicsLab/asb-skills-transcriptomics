---
name: mahalanobis-distance-mapping-metrics
description: Use when a Symphony-mapped query may contain cell types missing from
  the reference and you need quantitative per-cell or per-cluster scores (with AUC
  evaluation) to flag unmapped/novel populations.
when_to_use_negative:
- When the novel query population is expected to be very similar to an existing reference
  population — the article notes these metrics may be less informative in that case
- When the goal is de novo cell type discovery from the query alone rather than flagging
  cells that failed to map to a reference
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Symphony
  role: Provides mapQuery to embed the query and calcPerCellMappingMetric / calcPerClusterMappingMetric
    to compute Mahalanobis-based mapping confidence metrics
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Performs the reference integration whose compressed object underlies the reference
    embedding in which Mahalanobis distances are computed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:6e3ba980d12bc9008724782966652173e76edecac5be8207df192998104eb382
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# mahalanobis-distance-mapping-metrics

## Summary

Uses Symphony's per-cell and per-cluster Mahalanobis-distance-based mapping metrics to flag query cell types that are absent from the reference, quantifying detection performance as AUC for held-out cell types.

## When to use

Use when a Symphony-mapped query may contain cell types missing from the reference and you need quantitative per-cell or per-cluster scores (with AUC evaluation) to flag unmapped/novel populations.

## When NOT to use

- When the novel query population is expected to be very similar to an existing reference population — the article notes these metrics may be less informative in that case
- When the goal is de novo cell type discovery from the query alone rather than flagging cells that failed to map to a reference

## Inputs

- Symphony reference object built from an integrated reference atlas (e.g., Harmony-integrated PBMC or fetal liver reference)
- Query expression matrix for ablated/held-out scenarios (fetal liver non-immune, myeloid, Kupffer ablations; PBMC held-out types)
- Query cluster labels for per-cluster aggregation
- Reference embedding coordinates (Z_orig) for the query cells

## Outputs

- Per-cell Mahalanobis mapping metric values for every query cell
- Per-cluster mapping metric values aggregated over query_cluster_labels
- AUCs for held-out cell type detection (e.g., mapping_metric_aucs.csv)

## How to apply

Map each held-out or ablated query (e.g., fetal liver non-immune, myeloid, or Kupffer ablations; PBMC held-out-type scenarios) into its reference embedding with Symphony's mapQuery, then compute the per-cell Mahalanobis-distance-based mapping metric for every query cell using calcPerCellMappingMetric(reference, query, Z_orig, metric). Aggregate to a per-cluster metric with calcPerClusterMappingMetric(reference, query, query_cluster_labels, metric, u, lambda). Rank reference cell types by the mapping metric and compute AUCs for held-out type detection using the evaluate function; write results (e.g., non-immune 0.997/1.0 per-cell/per-cluster; Kupffer 0.633/0.963 per-cell/per-cluster) to a file such as mapping_metric_aucs.csv. The rationale is that query cells far from all reference clusters in the reference-derived low-dimensional embedding receive high metric values, so distinctly novel states separate cleanly while novel populations similar to existing reference types are harder to detect.

## Related tools

- **Symphony** (Provides mapQuery to embed the query and calcPerCellMappingMetric / calcPerClusterMappingMetric to compute Mahalanobis-based mapping confidence metrics) — https://github.com/immunogenomics/symphony
- **Harmony** (Performs the reference integration whose compressed object underlies the reference embedding in which Mahalanobis distances are computed)

## Examples

```
query = mapQuery(query_exp, query_metadata, reference, vars = vars); metric = calcPerCellMappingMetric(reference, query, query$Z_orig, metric = 'mahalanobis')
```

## Evaluation signals

- Per-cell AUC for detecting the held-out type should approach the reported values (non-immune = 0.997; Kupffer = 0.633)
- Per-cluster AUC should approach the reported values (non-immune = 1.0; Kupffer = 0.963)
- Held-out cell states very different from the reference should rank highest by the mapping metric across reference cell types
- Reproduced AUCs should be written to mapping_metric_aucs.csv with per-cell and per-cluster columns matching Supplementary Fig. 9 expectations

## Limitations

- These metrics may be less informative in cases where the novel population is very similar to an existing reference population
- Identification of novel cell types that have failed to map remains an unsolved future direction for mapping algorithms
- Symphony does not entirely supplant the need for users interested in novel cell type discovery to conduct de novo analyses of the query alone

## Evidence

- [supplementary] When query cell types are very different from the reference (mapping non-immune cells onto an immune-only reference), the mapping metrics clearly distinguish the missing cell states as novel: [evidence span withheld — anchor status: not_found]
- [discussion] we recognize that these metrics may be less informative in cases where the novel population is very similar to an existing reference population.: [evidence span withheld — anchor status: not_found]
- [discussion] Identification of novel cell-types that have failed to map is an important future direction for mapping algorithms.: "Identification of novel cell-types that have failed to map is an important future direction for mapping algorithms."
- [abstract] Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations to the query: "Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer"
- [methods] query = mapQuery(query_exp,: "query = mapQuery(query_exp,"
