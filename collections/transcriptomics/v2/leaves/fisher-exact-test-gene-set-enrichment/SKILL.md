---
name: fisher-exact-test-gene-set-enrichment
description: Use when the deCS query input is a list of genes — e.g., a top-N cluster
  marker gene table with 'cluster' and 'gene' columns or trait-associated genes —
  rather than a gene expression profile matrix.
when_to_use_negative:
- The query is a full gene expression profile or scaled z-score matrix rather than
  a gene list — use deCS.correlation (Pearson/Spearman correlation) instead
- The biological sample is malignant/cancer tissue, where expression differences between
  healthy reference profiles and malignant cells bias the mapping-based annotation
- The goal is to discover novel cell types not represented in the reference panels,
  since deCS is a mapping-based toolkit
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3517
tools:
- name: deCS
  role: Provides deCS.fisher(), which performs the Fisher's exact test enrichment
    of query marker gene lists against CTgenes from built-in reference panels
  repo: https://github.com/bsml320/deCS
- name: Seurat
  role: Generates the input marker gene lists via FindAllMarkers(only.pos = TRUE,
    min.pct = 0.25, logfc.threshold = 0.25) and top_n(10, wt = avg_log2FC)
- name: scCATCH
  role: Competing cluster-annotation tool that matches cluster marker genes to the
    CellMatch reference; usable as a baseline comparison
  repo: https://github.com/ZJUFanLab/scCATCH
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:540ed68459ba11519a4a80a147c15bf5bf66cb768665f4d90fe34bc1402cc7f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - CHETAH
  - CellAssign
  - SciBet
  - Seurat
  - SingleR
  - deCS
  - scCATCH
  - scMatch
  - scPred
  - scVI
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# fisher-exact-test-gene-set-enrichment

## Summary

deCS.fisher annotates query gene lists (e.g., cluster marker genes) as cell types by testing enrichment against cell-type-specific genes (CTgenes, top 5% by t-score/z-score) from a reference panel via Fisher's exact test, assigning the cell type with the highest intersection ratio (IR).

## When to use

Use when the deCS query input is a list of genes — e.g., a top-N cluster marker gene table with 'cluster' and 'gene' columns or trait-associated genes — rather than a gene expression profile matrix.

## When NOT to use

- The query is a full gene expression profile or scaled z-score matrix rather than a gene list — use deCS.correlation (Pearson/Spearman correlation) instead
- The biological sample is malignant/cancer tissue, where expression differences between healthy reference profiles and malignant cells bias the mapping-based annotation
- The goal is to discover novel cell types not represented in the reference panels, since deCS is a mapping-based toolkit

## Inputs

- Marker gene list table with columns 'cluster' and 'gene' (e.g., pbmc_top10_markers_list)
- Pre-calculated cell type specificity score reference panel (BlueprintEncode_main, DICE_main, MonacoImmune_main, Human_cell_landscape, Human_cell_atlas_of_fetal, or fine-grained variants)
- Filter parameters: ref_panel_markers_ratio, top_n, p.adjust.methods, intersection_threshold, p_threshold, cell_type_threshold

## Outputs

- Annotation table with columns Query, Max_correlation/IR, Top1–TopN candidate cell types, and Cell_labels (e.g., decs_fisher_annotation.tsv)
- Cell labels per query cluster, with 'Undetermined cells' for clusters failing cutoffs

## How to apply

Load the query marker gene table (columns 'cluster' and 'gene', e.g., pbmc_top10_markers_list from FindAllMarkers with top_n(10, wt = avg_log2FC)) and a pre-calculated cell type specificity score reference panel (e.g., MonacoImmune_main_t_score). For each cell type, define CTgenes by thresholding the specificity scores at ref_panel_markers_ratio (default top 5% of genes by t-score/z-score). For each query cluster and cell type, build a 2x2 contingency table of query–CTgene overlap, compute the Fisher's exact test P value and the intersection ratio IR = |Q∩R|/|Q|, then apply multiple-testing correction (p.adjust.methods, e.g., 'bonferroni'). Filter assignments with p_threshold (e.g., 1e-3), cell_type_threshold (e.g., 0.05), and intersection_threshold; clusters passing no cutoff are labeled 'Undetermined cells', and each cluster receives the cell type with the maximum IR among significant candidates.

## Related tools

- **deCS** (Provides deCS.fisher(), which performs the Fisher's exact test enrichment of query marker gene lists against CTgenes from built-in reference panels) — https://github.com/bsml320/deCS
- **Seurat** (Generates the input marker gene lists via FindAllMarkers(only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25) and top_n(10, wt = avg_log2FC))
- **scCATCH** (Competing cluster-annotation tool that matches cluster marker genes to the CellMatch reference; usable as a baseline comparison) — https://github.com/ZJUFanLab/scCATCH

## Examples

```
pbmc_deCS_fisher <- deCS.fisher(pbmc_top10_markers_list, MonacoImmune_main_t_score, ref_panel_markers_ratio = 0.05, p.adjust.methods = 'bonferroni', p_threshold = 1e-3, cell_type_threshold = 0.05)
```

## Evaluation signals

- Calling deCS.fisher(pbmc_top10_markers_list, MonacoImmune_main_t_score) reproduces the reference PBMC annotation behavior, with significant enrichments defined by adjusted P below p_threshold
- Each query cluster's assigned Cell_labels equals the cell type with the maximum intersection ratio (IR = Q∩R/Q) among candidates passing p_threshold and cell_type_threshold
- All reported P values are multiple-testing corrected with the chosen p.adjust.methods (e.g., 'bonferroni')
- Clusters failing intersection_threshold, p_threshold, or cell_type_threshold are labeled 'Undetermined cells' rather than force-assigned
- The output table keeps the top_n ranked candidates with their IR values per cluster and is writable as decs_fisher_annotation.tsv

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells

## Evidence

- [methods] Fisher's exact test branch: [evidence span withheld — anchor status: not_found]
- [methods] deCS.fisher parameters: "deCS.fisher(markers_list, ref_panel, type, ref_panel_markers_ratio, top_n, p.adjust.methods, label, intersection_threshold, p_threshold, cell_type_threshold, plot_figure)"
- [methods] p_threshold defaults: "p_threshold = 1e-3, cell_type_threshold = 0.05"
- [methods] CTgenes definition: "We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes"
- [methods] cor_threshold vs fisher gating: "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
- [readme] README input mode: "Depending on the type of query data, we implemented two test approaches: Correlation analysis and Fisher's exact test for cell type enrichment analysis."
