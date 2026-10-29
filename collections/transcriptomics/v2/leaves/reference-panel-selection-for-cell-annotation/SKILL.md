---
name: reference-panel-selection-for-cell-annotation
description: Use when annotating single-cell clusters or gene lists against a human
  cell type reference panel and you must choose which built-in deCS panel (main vs
  fine) and CTgene cutoff (e.g. ref_panel_markers_ratio = 0.05) to apply.
when_to_use_negative:
- 'The query is a cancer or malignant-cell dataset: deCS is a mapping-based toolkit
  that may be challenging for cancer cells due to expression profile differences between
  healthy and malignant cells'
- 'The cell type of interest is likely novel/uncharacterized: deCS fails to characterize
  novel cell types because it maps to fixed reference panels'
- The query data is already a finished cell type label file or a non-human species
  expression matrix outside the collected human reference panels
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: deCS
  role: Supplies the built-in reference panels and the deCS.correlation / deCS.fisher
    annotation functions that consume the selected panel
  repo: https://github.com/bsml320/deCS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:ca4d2f6661f76245ca85adffb1aaff07ba126e667c39469afd34fcb310565f80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - deCS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# reference-panel-selection-for-cell-annotation

## Summary

Selecting and parameterizing human cell type reference panels (BlueprintEncode, DICE, MonacoImmune, HCL, HCAF) inside deCS to define cell type-specific genes (CTgenes) from t-score/z-score specificity scores, which drives accurate cluster-level scRNA-seq annotation.

## When to use

Use when annotating single-cell clusters or gene lists against a human cell type reference panel and you must choose which built-in deCS panel (main vs fine) and CTgene cutoff (e.g. ref_panel_markers_ratio = 0.05) to apply.

## When NOT to use

- The query is a cancer or malignant-cell dataset: deCS is a mapping-based toolkit that may be challenging for cancer cells due to expression profile differences between healthy and malignant cells
- The cell type of interest is likely novel/uncharacterized: deCS fails to characterize novel cell types because it maps to fixed reference panels
- The query data is already a finished cell type label file or a non-human species expression matrix outside the collected human reference panels

## Inputs

- Marker gene list table with columns 'cluster' and 'gene' (e.g. pbmc_top10_markers_list from Seurat FindAllMarkers top_n)
- Scaled gene expression z-score matrix of cluster markers (e.g. pbmc_cluster_marker_z_score)
- deCS built-in reference specificity score panels: BlueprintEncode_main/_fine, DICE_main/_fine, MonacoImmune_main/_fine, Human_cell_landscape (HCL_z_score), Human_cell_atlas_of_fetal (HCAF_z_score)
- Parameters: ref_panel_markers_ratio (e.g. 0.05), top_n, p.adjust.methods, intersection_threshold, p_threshold (e.g. 1e-3), cell_type_threshold (e.g. 0.05)

## Outputs

- Annotation table with Query, Max_correlation or Top candidates, IR, P value, and Cell_labels columns (e.g. deCS_Cor_annotation_A.pdf or decs_fisher_annotation.tsv)
- 'Undetermined cells' labels for clusters passing no significance/overlap cutoff

## How to apply

Load a pre-calculated cell type specificity score panel shipped with deCS (e.g. data(MonacoImmune_main) or MonacoImmune_fine, HCL_z_score, HCAF_z_score), choosing the 'main' panel for coarse cell types and the 'fine' panel for subtypes. Define CTgenes for each cell type by thresholding the t-score/z-score columns at the user-set ref_panel_markers_ratio (default top 5% of genes). The article's evaluation shows that expanding the reference collection is critical for annotation accuracy, so multiple panels can be applied and results integrated, since deCS annotates at the cell cluster level rather than per cell. Match panel tissue scope to the query (e.g. MonacoImmune for immune PBMC data; HCAF is the only tissue-aware panel). For gene-list queries, the CTgenes feed a Fisher's exact test where the cell type with the highest intersection ratio IR = |Q∩R|/|Q| is assigned; for expression-profile queries, the same specificity-score panels are correlated with the query's scaled z-score matrix.

## Related tools

- **deCS** (Supplies the built-in reference panels and the deCS.correlation / deCS.fisher annotation functions that consume the selected panel) — https://github.com/bsml320/deCS

## Examples

```
data(MonacoImmune_main); pbmc_deCS_cor_panel_A <- deCS.correlation(pbmc_cluster_marker_z_score, MonacoImmune_main_t_score)
```

## Evaluation signals

- Calling deCS.fisher(pbmc_top10_markers_list, MonacoImmune_main_t_score) reproduces the reference PBMC annotation behavior with significant enrichments at adjusted P below p_threshold
- Annotation of the 9-cluster PBMC example yields expected Cell_labels (e.g. Naive/Memory CD4 T → CD4+ T cells, CD14+/FCGR3A+ Mono → Monocytes, B → B cells, NK → NK cells, DC → Dendritic cells)
- Each cell type's CTgenes set equals approximately the top ref_panel_markers_ratio fraction (e.g. 5%) of genes ranked by t-score/z-score in the loaded panel
- All output labels are either a reference-panel cell type or 'Undetermined cells'; no assignments survive with adjusted P ≥ p_threshold or IR below intersection_threshold
- Switching from main to fine panels changes label granularity consistently (coarse labels become the expected subtypes) without changing coarse-level assignments

## Limitations

- deCS does not take tissue information into account (except for HCAF), which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells
- No software version for deCS is stated in the discussion

## Evidence

- [methods] We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes: "We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes"
- [abstract] Our results demonstrate that expanding the references is critical for improving annotation accuracy.: "Our results demonstrate that expanding the references is critical for improving annotation accuracy."
- [methods] deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal"
- [discussion] deCS does not take tissue information (except for HCAF) into account which may bias the annotation results: "deCS does not take tissue information (except for HCAF) into account which may bias the annotation results"
- [methods] give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells"): "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
