---
name: correlation-based-cell-type-annotation
description: Use when a normalized, scaled gene expression profile (cluster-average
  z-scored markers or per-cell scaled matrix) needs automatic cell type labels against
  human reference panels such as MonacoImmune, BlueprintEncode, DICE, HCL, or HCAF.
when_to_use_negative:
- The query is a list of marker genes rather than an expression profile — use deCS.fisher()
  (Fisher's exact test cell type enrichment) instead
- The goal is to discover or characterize novel cell types — deCS is a mapping-based
  tool that can only assign labels from the reference panels
- Annotating malignant/cancer cells, where healthy-reference-based expression profiles
  differ substantially and bias the annotation
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: deCS
  role: Provides deCS.correlation() for PCC-based cell type assignment and the built-in
    reference panels and CellMatch marker data
  repo: https://github.com/bsml320/deCS
- name: Seurat
  role: 'Preprocessing of the query scRNA-seq dataset: NormalizeData, FindVariableFeatures,
    ScaleData, RunPCA, FindNeighbors, FindClusters, FindAllMarkers, AverageExpression'
- name: reshape2
  role: deCS dependency (>= 1.4.4) for matrix reshaping of annotation results
- name: ggplot2
  role: deCS dependency (>= 3.3.2) used for the correlation annotation plotting output
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_008
  - task_002
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:d505163c125f462e729f8f72ec34aec4af8e17e4b60fba34018533e5eaf02dd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - CHETAH
  - Cell BLAST
  - CellAssign
  - SciBet
  - Seurat
  - SingleR
  - deCS
  - dplyr
  - ggplot2
  - reshape2
  - scCATCH
  - scMatch
  - scPred
  - scVI
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# correlation-based-cell-type-annotation

## Summary

Annotate single-cell or bulk RNA-seq query clusters by computing Pearson correlation coefficients (PCC) between a z-scored query marker-expression matrix and pre-calculated cell type specificity t-score/z-score reference panels using the deCS.correlation() function.

## When to use

Use when a normalized, scaled gene expression profile (cluster-average z-scored markers or per-cell scaled matrix) needs automatic cell type labels against human reference panels such as MonacoImmune, BlueprintEncode, DICE, HCL, or HCAF.

## When NOT to use

- The query is a list of marker genes rather than an expression profile — use deCS.fisher() (Fisher's exact test cell type enrichment) instead
- The goal is to discover or characterize novel cell types — deCS is a mapping-based tool that can only assign labels from the reference panels
- Annotating malignant/cancer cells, where healthy-reference-based expression profiles differ substantially and bias the annotation

## Inputs

- Scaled query gene expression matrix (markers_expression; cluster-average z-score of top markers with clusters as columns, or per-cell scaled matrix)
- Pre-calculated cell type specificity score reference panel (ref_panel; e.g. MonacoImmune_main_t_score, BlueprintEncode_main_t_score, DICE_main_t_score, HCL_z_score, HCAF_z_score)
- Filter parameters: cor_threshold, p_threshold, cell_type_threshold

## Outputs

- Annotation table with columns Query, Max_correlation, Top1, Top2, Top3, Cell_labels (tab-delimited, e.g. pbmc_deCS_result.txt)
- Optional annotation heatmap PDF (deCS_Cor_annotation_A.pdf, 8 x 8 inches)

## How to apply

Preprocess the query in Seurat (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors(dims=1:10), FindClusters(resolution=0.5)), then compute top cluster markers with FindAllMarkers(only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) and derive the cluster-average z-score matrix via log2(AverageExpression+1) plus row scaling. Load a built-in reference panel with data() (e.g. data(MonacoImmune_main) for the main t-score panel) and call deCS.correlation(markers_expression, ref_panel), which ranks reference cell types by PCC and reports Query, Max_correlation, Top1, Top2, Top3, and Cell_labels. Apply cor_threshold (plus p_threshold and cell_type_threshold defaults around 0.01/0.5) so low-confidence queries are labeled 'Undetermined cells'. Write results with write.table(..., sep='\t', quote=F) and optionally wrap the call in pdf('deCS_Cor_annotation_A.pdf', 8, 8) to render the annotation heatmap. The Fisher-based deCS.fisher() on top-10 marker gene lists can be run in parallel to check concordance of main labels.

## Related tools

- **deCS** (Provides deCS.correlation() for PCC-based cell type assignment and the built-in reference panels and CellMatch marker data) — https://github.com/bsml320/deCS
- **Seurat** (Preprocessing of the query scRNA-seq dataset: NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters, FindAllMarkers, AverageExpression)
- **reshape2** (deCS dependency (>= 1.4.4) for matrix reshaping of annotation results)
- **ggplot2** (deCS dependency (>= 3.3.2) used for the correlation annotation plotting output)

## Examples

```
data(MonacoImmune_main); pbmc_deCS_cor_panel_A <- deCS.correlation(pbmc_cluster_marker_z_score, MonacoImmune_main_t_score); write.table(pbmc_deCS_cor_panel_A, file = "pbmc_deCS_result.txt", sep = "\t", quote = F)
```

## Evaluation signals

- Output table schema contains exactly the columns Query, Max_correlation, Top1, Top2, Top3, Cell_labels and one row per query cluster/cell
- Max_correlation values fall in a plausible range (approximately 0.4–0.9 in the published PBMC example run)
- PBMC 3k cluster labels match expected assignments: Naive/Memory CD4 T -> CD4+ T cells, CD14+/FCGR3A+ Mono -> Monocytes, B -> B cells, CD8 T -> CD8+ T cells, NK -> NK cells, DC -> Dendritic cells, Platelet -> Progenitors (low-concordance case)
- Main cell type labels are concordant between deCS.correlation (PCC) and deCS.fisher (IR) for all 9 PBMC clusters
- Low-confidence queries are flagged as 'Undetermined cells' when correlation falls below cor_threshold

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells
- Annotation accuracy depends on the reference database used; expanding the references is critical for improving annotation accuracy

## Evidence

- [readme] we calculate pearson correlation coefficient (PCC) or Spearman's rank correlation coefficient with each of the cell type in the reference dataset, then the most relevant cell type(s) will be identified: "we calculate pearson correlation coefficient (PCC) or Spearman's rank correlation coefficient with each of the cell type in the reference dataset"
- [methods] give cor_threshold to avoid mis-annotation (regard as 'Undetermined cells'): "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
- [abstract] Compared to many existing state-of-the-art annotation tools, deCS significantly reduced computation time and increased accuracy.: "Compared to many existing state-of-the-art annotation tools, deCS significantly reduced computation time and increased accuracy."
- [readme] deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal"
- [intro] deCS reduced computation time and improved annotation accuracy in most tissues compared to other state-of-the-art methods: "deCS reduced computation time and improved annotation accuracy in most tissues compared to other state-of-the-art methods"
