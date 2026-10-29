---
name: multiple-testing-correction-of-enrichment-pvalues
description: 'Use when annotating clusters from a marker gene list with deCS.fisher:
  per-cell-type Fisher tests yield many raw p-values that must be corrected (e.g.
  bonferroni) before calling significant cell type assignments.'
when_to_use_negative:
- The query is a gene expression profile (scaled z-score matrix) rather than a gene
  list — use deCS.correlation, which ranks cell types by Pearson/Spearman correlation
  with cor_threshold and does not use Fisher p-value correction
- The goal is single-cell-level annotation rather than cluster-level annotation —
  deCS defines cell types at the cell cluster level
- The expected cell type is novel/not present in the reference panels — deCS is a
  mapping-based toolkit and cannot characterize novel cell types
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: deCS
  role: R package whose deCS.fisher() function performs the Fisher's exact tests,
    applies p.adjust.methods correction, and filters assignments by p_threshold/cell_type_threshold
  repo: https://github.com/bsml320/deCS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:d503240798d644794e0c3e2523624d21288d540743179fd94458f3b798d8803f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - deCS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Multiple-testing correction of enrichment p-values

## Summary

In deCS's gene-list (Fisher's exact test) mode, raw enrichment p-values from testing a query marker gene list against each cell type's CTgenes are corrected for multiple testing across cell types (p.adjust.methods, e.g. 'bonferroni'), and corrected p-values gate final cell type assignments via p_threshold (e.g. 1e-3) together with cell_type_threshold and intersection_threshold filters.

## When to use

Use when annotating clusters from a marker gene list with deCS.fisher: per-cell-type Fisher tests yield many raw p-values that must be corrected (e.g. bonferroni) before calling significant cell type assignments.

## When NOT to use

- The query is a gene expression profile (scaled z-score matrix) rather than a gene list — use deCS.correlation, which ranks cell types by Pearson/Spearman correlation with cor_threshold and does not use Fisher p-value correction
- The goal is single-cell-level annotation rather than cluster-level annotation — deCS defines cell types at the cell cluster level
- The expected cell type is novel/not present in the reference panels — deCS is a mapping-based toolkit and cannot characterize novel cell types

## Inputs

- Query marker gene list table with columns 'cluster' and 'gene' (e.g. pbmc_top10_markers_list)
- Pre-calculated cell type specificity score reference panel (e.g. MonacoImmune_main_t_score)
- Parameter values: ref_panel_markers_ratio (e.g. 0.05), top_n, p.adjust.methods (e.g. 'bonferroni'), intersection_threshold, p_threshold (e.g. 1e-3), cell_type_threshold (e.g. 0.05)

## Outputs

- Annotation table (Query, Top candidates, IR, P value, Cell_labels) written to decs_fisher_annotation.tsv
- Corrected p-values and IR values for the top_n ranked cell type candidates per query cluster
- 'Undetermined cells' labels for clusters passing no significance cutoff

## How to apply

For each query cluster and each cell type in the reference panel (e.g. MonacoImmune_main_t_score with CTgenes defined as the top 5% by t-score, ref_panel_markers_ratio = 0.05), build the 2x2 contingency table of overlap between query genes and CTgenes and compute the Fisher's exact test p-value plus the intersection ratio IR = |Q∩R|/|Q|. Then apply multiple-testing correction across the cell-type panel via p.adjust.methods (e.g. 'bonferroni'). Filter assignments using p_threshold (e.g. 1e-3), cell_type_threshold (e.g. 0.05), and intersection_threshold; clusters passing no cutoff are labeled 'Undetermined cells'. Among significant candidates, assign the cell type with the highest IR, keeping top_n ranked candidates with their IR values and corrected p-values in the output table written to decs_fisher_annotation.tsv.

## Related tools

- **deCS** (R package whose deCS.fisher() function performs the Fisher's exact tests, applies p.adjust.methods correction, and filters assignments by p_threshold/cell_type_threshold) — https://github.com/bsml320/deCS

## Examples

```
pbmc_deCS_fisher <- deCS.fisher(pbmc_top10_markers_list, MonacoImmune_main_t_score, p.adjust.methods = "bonferroni", p_threshold = 1e-3, cell_type_threshold = 0.05)
```

## Evaluation signals

- Calling deCS.fisher(pbmc_top10_markers_list, MonacoImmune_main_t_score) reproduces the reference annotation behavior on the PBMC example
- All reported candidate cell types have adjusted p-values below p_threshold (e.g. 1e-3); raw uncorrected p-values should not appear as the gating criterion
- Assigned Cell_labels correspond to the candidate with the highest IR among significant cell types, with top_n candidates ranked by IR in the output table
- Clusters whose cell-type enrichment fails p_threshold, cell_type_threshold, or intersection_threshold are labeled 'Undetermined cells' rather than force-assigned
- Output file decs_fisher_annotation.tsv contains the expected schema: Query, Top candidates, IR, P value, Cell_labels

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells
- Correlation-mode results require cor_threshold to avoid mis-annotation (regarded as 'Undetermined cells'), analogous to the p-value gating in Fisher mode

## Evidence

- [methods] fisher p_threshold and cell_type_threshold defaults: "p_threshold = 1e-3, cell_type_threshold = 0.05"
- [methods] deCS.fisher signature with correction and filter parameters: "deCS.fisher(markers_list, ref_panel, type, ref_panel_markers_ratio, top_n, p.adjust.methods, label, intersection_threshold, p_threshold, cell_type_threshold, plot_figure)"
- [other] workflow description of correction and IR-based assignment: [evidence span withheld — anchor status: not_found]
- [methods] undetermined cells labeling when cutoffs fail: "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
- [other] Fisher mode tests query genes against CTgenes per cell type: [evidence span withheld — anchor status: not_found]
