---
name: annotation-result-table-export
description: Use when a deCS.correlation or deCS.fisher annotation run has produced
  a result data frame and you need a machine-readable, tab-delimited annotation table
  (e.g. hipsc_decs_result.txt) with row-count and label checks.
when_to_use_negative:
- The query has not yet been annotated — export only applies to a completed deCS result
  data frame
- The output must be consumed by tools requiring quoted CSV or structured formats
  (e.g. .h5ad, JSON) rather than unquoted TSV
- Query cells are malignant/cancer cells, where deCS's mapping-based annotation is
  expected to be unreliable and exported labels may be misleading
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: deCS
  role: Produces the annotation result table that is exported; deCS.correlation()
    returns the Query/Max_correlation/Top1-3/Cell_labels data frame written by write.table
  repo: https://github.com/bsml320/deCS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:b127e4c13be4ed6d413ee9d551743f537d580692d2c0779a1cc907474987162c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - deCS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# annotation-result-table-export

## Summary

Export deCS cell type annotation results (Query, Max_correlation, Top1–Top3, Cell_labels) as a plain tab-delimited text table using write.table(..., sep = '\t', quote = F), enabling downstream review and validation of per-sample or per-cluster cell type calls.

## When to use

Use when a deCS.correlation or deCS.fisher annotation run has produced a result data frame and you need a machine-readable, tab-delimited annotation table (e.g. hipsc_decs_result.txt) with row-count and label checks.

## When NOT to use

- The query has not yet been annotated — export only applies to a completed deCS result data frame
- The output must be consumed by tools requiring quoted CSV or structured formats (e.g. .h5ad, JSON) rather than unquoted TSV
- Query cells are malignant/cancer cells, where deCS's mapping-based annotation is expected to be unreliable and exported labels may be misleading

## Inputs

- deCS annotation result data frame from deCS.correlation() or deCS.fisher() with columns Query, Max_correlation, Top1, Top2, Top3, Cell_labels
- z-scaled markers_expression matrix (log-transformed, human gene symbols) of query samples or clusters

## Outputs

- Tab-delimited annotation table (e.g. hipsc_decs_result.txt) with one row per query sample/cluster and columns Query, Max_correlation, Top1, Top2, Top3, Cell_labels

## How to apply

After running deCS.correlation(markers_expression, ref_panel) — e.g. on z-scaled bulk RNA-seq profiles of hiPSC-NPC (n=13) and hiPSC-neuron (n=11) samples against MonacoImmune_main, BlueprintEncode_main, or Human_cell_atlas_of_fetal references — the returned data frame has fixed columns: Query, Max_correlation, Top1, Top2, Top3, Cell_labels. Write it with write.table(result, file = 'hipsc_decs_result.txt', sep = '\t', quote = F) so field names stay unquoted and parseable. Validate the export by checking the expected row count (one row per query cluster/sample, e.g. 24 rows for 13 NPC + 11 neuron samples) and that Cell_labels reflect biologically expected assignments (progenitor/hESC-like for NPCs, neuronal types for neurons). Optionally apply cor_threshold upstream so low-confidence calls are flagged as 'Undetermined cells' rather than silently exported.

## Related tools

- **deCS** (Produces the annotation result table that is exported; deCS.correlation() returns the Query/Max_correlation/Top1-3/Cell_labels data frame written by write.table) — https://github.com/bsml320/deCS

## Examples

```
write.table(hipsc_decs_result, file = "hipsc_decs_result.txt", sep = "\t", quote = F)
```

## Evaluation signals

- Output file exists and is tab-delimited with header columns Query, Max_correlation, Top1, Top2, Top3, Cell_labels
- Row count matches the number of query samples/clusters (e.g. 24 rows for 13 hiPSC-NPC + 11 hiPSC-neuron samples)
- Max_correlation values are Pearson coefficients within [-1, 1]; low-confidence rows flagged 'Undetermined cells' if cor_threshold was set
- Cell_labels match biologically expected assignments: NPC samples to progenitor/hESC-like types, neuron samples to neuronal cell types
- No quoted fields in the file (quote = F), so downstream parsers split cleanly on tabs

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results and hence the exported Cell_labels
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types, so exported labels are restricted to reference panel types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells

## Evidence

- [readme] write.table of annotation table with fixed columns: "write.table(pbmc_deCS_cor_panel_A, file = "pbmc_deCS_result.txt", sep = "\t", quote = F)"
- [readme] Result table column schema: "Query Max_correlation            Top1            Top2            Top3     Cell_labels"
- [methods] cor_threshold to flag low-confidence calls: "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
- [readme] Reference panels loaded via data(): "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal"
