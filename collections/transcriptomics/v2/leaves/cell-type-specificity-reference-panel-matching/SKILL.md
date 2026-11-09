---
name: cell-type-specificity-reference-panel-matching
description: Use when a preprocessed, z-scored cluster-level marker-gene expression
  matrix (clusters as columns) needs automatic cell type labels matched against a
  human cell type reference panel such as MonacoImmune or BlueprintEncode.
when_to_use_negative:
- Query consists of a gene list rather than expression profiles — use the Fisher's
  exact test mode (deCS.fisher) instead of correlation analysis
- Goal is characterizing novel cell types or annotating malignant/cancer cells, since
  deCS is a mapping-based toolkit that fails on novel types and struggles with healthy-vs-malignant
  expression differences
- Single-cell-level annotation with tissue-specific context is required — deCS annotates
  at the cluster level and ignores tissue information (except HCAF), which may bias
  results
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: deCS
  role: 'Core annotation tool: deCS.correlation() computes PCC between the query marker-gene
    z-score matrix and each cell type in the reference panel and returns top-ranked
    labels with confidence filters'
  repo: https://github.com/bsml320/deCS
- name: Seurat
  role: Used upstream to normalize, cluster, and extract top cluster marker genes
    (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters,
    FindAllMarkers, AverageExpression) that form the query input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:c82260d136919fd5e4482db5494693b37b7bc04f2bbca328925b8405a0490857
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - Seurat
  - deCS
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cell-type-specificity-reference-panel-matching

## Summary

Annotates cell clusters from a scRNA-seq query by computing Pearson correlation coefficients between the query's z-scored marker-gene expression matrix and a built-in cell type specificity score reference panel (e.g., MonacoImmune_main t-score), assigning the top-matching cell type label(s) per cluster.

## When to use

Use when a preprocessed, z-scored cluster-level marker-gene expression matrix (clusters as columns) needs automatic cell type labels matched against a human cell type reference panel such as MonacoImmune or BlueprintEncode.

## When NOT to use

- Query consists of a gene list rather than expression profiles — use the Fisher's exact test mode (deCS.fisher) instead of correlation analysis
- Goal is characterizing novel cell types or annotating malignant/cancer cells, since deCS is a mapping-based toolkit that fails on novel types and struggles with healthy-vs-malignant expression differences
- Single-cell-level annotation with tissue-specific context is required — deCS annotates at the cluster level and ignores tissue information (except HCAF), which may bias results

## Inputs

- z-scored cluster marker-gene expression matrix (e.g., pbmc_cluster_marker_z_score from pbmc_example.rda, clusters as columns)
- built-in cell type specificity score reference panel (t-score or z-score matrix, e.g., MonacoImmune_main_t_score, BlueprintEncode_main, DICE_fine, HCL_z_score, HCAF_z_score)

## Outputs

- per-cluster annotation data.frame with columns Query, Max_correlation, Top1, Top2, Top3, Cell_labels (e.g., written via write.table as tab-delimited pbmc_deCS_result.txt)

## How to apply

First preprocess the query via the standard Seurat pipeline (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters), identify cluster markers with FindAllMarkers, take the top markers (e.g., top 10 by avg_log2FC), average expression per cluster, and z-score across clusters (t(scale(t(...)))). Load a built-in reference panel with data(MonacoImmune_main) (or BlueprintEncode_main, DICE_main, Human_cell_landscape, Human_cell_atlas_of_fetal) and run deCS.correlation(pbmc_cluster_marker_z_score, MonacoImmune_main_t_score), which computes the PCC between each query cluster and every reference cell type. Control false annotation with cor_threshold (labels low-confidence queries 'Undetermined cells'), plus p_threshold and cell_type_threshold; optionally fine-tune for closely related cell types. The rationale is that deCS annotates at the cluster level, integrates multiple references, and runs faster than per-cell tools like SingleR, scCATCH, or CellAssign.

## Related tools

- **deCS** (Core annotation tool: deCS.correlation() computes PCC between the query marker-gene z-score matrix and each cell type in the reference panel and returns top-ranked labels with confidence filters) — https://github.com/bsml320/deCS
- **Seurat** (Used upstream to normalize, cluster, and extract top cluster marker genes (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters, FindAllMarkers, AverageExpression) that form the query input)

## Examples

```
pbmc_deCS_cor_panel_A <- deCS.correlation(pbmc_cluster_marker_z_score, MonacoImmune_main_t_score); write.table(pbmc_deCS_cor_panel_A, file="pbmc_deCS_result.txt", sep="\t", quote=F)
```

## Evaluation signals

- Function runs without error on the PBMC example and returns a data.frame with exactly the columns Query, Max_correlation, Top1, Top2, Top3, Cell_labels
- Cell_labels match expected PBMC assignments: Naive/Memory CD4 T -> CD4+ T cells, B -> B cells, NK -> NK cells, CD14+/FCGR3A+ Mono -> Monocytes, DC -> Dendritic cells
- Max_correlation values fall in the plausible example range ~0.4-0.9 (e.g., NK at 0.893, Memory CD4 T at 0.862)
- Low-confidence clusters are labeled 'Undetermined cells' when cor_threshold/p_threshold/cell_type_threshold cutoffs are applied
- Output file is tab-delimited without quotes when written via write.table(..., sep='\t', quote=F)

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells

## Evidence

- [readme] correlation via PCC on scaled query vs reference panel: "we provide function `deCS.correlation()` for cell type enrichment analysis. Simply, we calculate pearson correlation coefficient (PCC) or Spearman's rank correlation coefficient with each of the cell"
- [readme] query input format: "`markers_expression` is scaled gene expression matrix for human scRNA-seq or bulk RNA-seq"
- [methods] undetermined label threshold: "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
- [intro] cluster-level multi-reference rationale: "deCS runs fast, allows the integration of cell annotation from multiple references, and defines cell types at the cell cluster level rather than the single-cell level"
- [discussion] tissue bias limitation: "deCS does not take tissue information (except for HCAF) into account which may bias the annotation results"
