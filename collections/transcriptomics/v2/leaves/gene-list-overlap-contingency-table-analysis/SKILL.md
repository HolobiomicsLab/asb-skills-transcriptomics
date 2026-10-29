---
name: gene-list-overlap-contingency-table-analysis
description: Use when the query input is a list of genes (cluster marker genes with
  'cluster' and 'gene' columns, or trait-associated genes) rather than an expression
  profile, and a cell type label is needed for each gene set.
when_to_use_negative:
- Query is a scaled gene expression profile (z-score matrix per cluster) — use deCS.correlation,
  which uses Pearson/Spearman correlation instead of contingency-table overlap
- The cell type of interest is not represented in any deCS reference panel (BlueprintEncode,
  DICE, MonacoImmune, HCL, HCAF) or expected to be a novel cell type, since deCS is
  a mapping-based toolkit
- Cancer cell annotation, where expression profile differences between healthy reference
  and malignant cells bias results
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: deCS
  role: Provides the deCS.fisher function that builds 2x2 contingency tables of query-vs-CTgenes
    overlap, computes Fisher's exact test P values and intersection ratios, and assigns
    cell type labels
  repo: https://github.com/bsml320/deCS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:c21f76109388b0fb592d4b73fc1e5940584bc18cea75700fe7ed3dae4e748dd0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - deCS
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-list-overlap-contingency-table-analysis

## Summary

Annotate query gene lists (e.g., cluster marker genes) to cell types using deCS.fisher, which tests overlap between the query set and each cell type's top-ranked specific genes (CTgenes, default top 5%) via Fisher's exact test and assigns the cell type with the highest intersection ratio (IR = Q∩R/Q).

## When to use

Use when the query input is a list of genes (cluster marker genes with 'cluster' and 'gene' columns, or trait-associated genes) rather than an expression profile, and a cell type label is needed for each gene set.

## When NOT to use

- Query is a scaled gene expression profile (z-score matrix per cluster) — use deCS.correlation, which uses Pearson/Spearman correlation instead of contingency-table overlap
- The cell type of interest is not represented in any deCS reference panel (BlueprintEncode, DICE, MonacoImmune, HCL, HCAF) or expected to be a novel cell type, since deCS is a mapping-based toolkit
- Cancer cell annotation, where expression profile differences between healthy reference and malignant cells bias results

## Inputs

- Query marker gene list table with columns 'cluster' and 'gene' (e.g. pbmc_top10_markers_list)
- Pre-calculated cell type specificity score reference panel (e.g. MonacoImmune_main_t_score from deCS built-in data: BlueprintEncode, DICE, MonacoImmune, HCL, HCAF)

## Outputs

- Annotation table (Query, Top candidates, IR, P value, Cell_labels) written to decs_fisher_annotation.tsv
- Per-cluster top_n ranked candidate cell types with intersection ratios and adjusted P values

## How to apply

Load the query marker gene table (columns 'cluster' and 'gene', e.g. pbmc_top10_markers_list from Seurat FindAllMarkers top_n = 10 by avg_log2FC) and a pre-calculated specificity-score reference panel (e.g. MonacoImmune_main_t_score) from the installed deCS package. For each cell type, define CTgenes by thresholding t-score/z-score at ref_panel_markers_ratio (default top 5% of genes). For each query cluster and cell type, build a 2x2 contingency table of the overlap and compute the Fisher's exact test P value plus the intersection ratio IR = Q∩R/Q. Apply multiple-testing correction (p.adjust.methods, e.g. 'bonferroni') and gate assignments with p_threshold = 1e-3, cell_type_threshold = 0.05, and intersection_threshold; clusters passing no cutoff are labeled 'Undetermined cells'. Assign each cluster the significant cell type with the highest IR, retaining top_n ranked candidates and IR values, then write the annotation table (Query, Top candidates, IR, P value, Cell_labels) to decs_fisher_annotation.tsv.

## Related tools

- **deCS** (Provides the deCS.fisher function that builds 2x2 contingency tables of query-vs-CTgenes overlap, computes Fisher's exact test P values and intersection ratios, and assigns cell type labels) — https://github.com/bsml320/deCS

## Examples

```
pbmc_deCS_fisher <- deCS.fisher(pbmc_top10_markers_list, MonacoImmune_main_t_score)
```

## Evaluation signals

- Calling deCS.fisher(pbmc_top10_markers_list, MonacoImmune_main_t_score) reproduces the reference PBMC annotation (e.g. Naive CD4 T → CD4+ T cells, CD14+ Mono → Monocytes, B → B cells)
- All significant assignments have adjusted P values below p_threshold (1e-3 after p.adjust.methods correction)
- Assigned cell type has the maximum intersection ratio (IR = Q∩R/Q) among significant candidates for each cluster
- Clusters passing no cutoff (p_threshold, cell_type_threshold = 0.05, intersection_threshold) are labeled 'Undetermined cells' rather than force-assigned
- Output table contains the expected schema: Query, Top candidates, IR, P value, Cell_labels

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells
- Annotation outcome depends on ref_panel_markers_ratio cutoff (default top 5% of genes by t-score/z-score defining CTgenes)

## Evidence

- [supplementary] In the gene-list mode of deCS, the query gene set is tested against each cell type's CTGenes (top 5% by default) via Fisher's exact test: [evidence span withheld — anchor status: not_found]
- [intro] deCS supports both gene expression profiles and gene lists as query input: "It supports input for either gene expression profiles (e.g., a query scRNA-seq dataset with clusters to be annotated) or list of genes"
- [methods] We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes: "We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes"
- [methods] p_threshold = 1e-3, cell_type_threshold = 0.05: "p_threshold = 1e-3, cell_type_threshold = 0.05"
- [readme] Depending on the type of query data, we implemented two test approaches: Correlation analysis and Fisher's exact test for cell type enrichment analysis.: "Depending on the type of query data, we implemented two test approaches: Correlation analysis and Fisher's exact test for cell type enrichment analysis."
