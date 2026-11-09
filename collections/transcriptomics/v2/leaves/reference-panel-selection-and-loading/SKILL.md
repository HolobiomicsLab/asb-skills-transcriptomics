---
name: reference-panel-selection-and-loading
description: Use when annotating single-cell clusters or bulk RNA-seq samples (e.g.
  iPSC-derived NPC/neuron profiles) against deCS built-in reference panels and you
  must choose and load the correct reference before running deCS.correlation().
when_to_use_negative:
- The query cells are novel cell types with no reference counterpart — deCS is a mapping-based
  toolkit that fails to characterize novel cell types
- The query is a cancer/malignant cell sample where expression profiles differ strongly
  from the healthy reference panels
- The query input is a plain marker gene list rather than an expression profile —
  use the deCS.fisher() enrichment route instead of deCS.correlation()
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: deCS
  role: Provides the built-in cell type specificity score reference panels and the
    deCS.correlation() function that maps scaled query profiles to reference cell
    types
  repo: https://github.com/bsml320/deCS
- name: Seurat
  role: Used upstream to normalize (NormalizeData), scale (ScaleData) and average
    cluster expression before building the markers_expression matrix
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:85c7f113f585a310a6650a50dcfdbedf0379d6fdd468df72ded6180f8f322137
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - Seurat
  - deCS
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# reference-panel-selection-and-loading

## Summary

Load deCS's pre-calculated cell type specificity score reference panels (BlueprintEncode, DICE, MonacoImmune, HCL, HCAF) via data() and select the panel appropriate to the query tissue or developmental stage, then feed the scaled markers_expression matrix into deCS.correlation() to obtain query-to-cell-type annotations.

## When to use

Use when annotating single-cell clusters or bulk RNA-seq samples (e.g. iPSC-derived NPC/neuron profiles) against deCS built-in reference panels and you must choose and load the correct reference before running deCS.correlation().

## When NOT to use

- The query cells are novel cell types with no reference counterpart — deCS is a mapping-based toolkit that fails to characterize novel cell types
- The query is a cancer/malignant cell sample where expression profiles differ strongly from the healthy reference panels
- The query input is a plain marker gene list rather than an expression profile — use the deCS.fisher() enrichment route instead of deCS.correlation()

## Inputs

- markers_expression: z-scaled expression matrix (human gene symbols as rownames) for scRNA-seq clusters or bulk RNA-seq samples
- deCS built-in reference panel loaded via data() (e.g. MonacoImmune_main, BlueprintEncode_main, DICE_main, Human_cell_landscape, Human_cell_atlas_of_fetal)
- Optional cor_threshold parameter to flag low-confidence annotations

## Outputs

- Annotation data.frame with columns Query, Max_correlation, Top1, Top2, Top3, Cell_labels (one row per query sample/cluster)
- Tab-delimited result file, e.g. hipsc_decs_result.txt or pbmc_deCS_result.txt

## How to apply

After installing deCS with devtools::install_github("bsml320/deCS"), load the pre-calculated cell type specificity score panels with data(), e.g. data(MonacoImmune_main), data(BlueprintEncode_main), data(DICE_main), data(Human_cell_landscape), and data(Human_cell_atlas_of_fetal); the article stresses that expanding the references is critical for improving annotation accuracy, so match the panel to the query biology (e.g. use Human_cell_atlas_of_fetal/HCAF for fetal or progenitor-like samples such as iPSC-derived NPCs). Prepare the query as a markers_expression matrix: log-transform cluster-average or sample-level expression, keep human gene symbols, and z-scale rows with t(scale(t(...))) so it is comparable to the reference t-score/z-score panels. Run deCS.correlation(markers_expression, ref_panel), which computes Pearson correlation coefficients between each query profile and each reference cell type; optionally set cor_threshold so low-confidence matches are reported as 'Undetermined cells'. Write the resulting table (Query, Max_correlation, Top1, Top2, Top3, Cell_labels) with write.table(..., sep = "\t", quote = F).

## Related tools

- **deCS** (Provides the built-in cell type specificity score reference panels and the deCS.correlation() function that maps scaled query profiles to reference cell types) — https://github.com/bsml320/deCS
- **Seurat** (Used upstream to normalize (NormalizeData), scale (ScaleData) and average cluster expression before building the markers_expression matrix)

## Examples

```
library(deCS); data(MonacoImmune_main); data(Human_cell_atlas_of_fetal); res <- deCS.correlation(markers_expression, Human_cell_atlas_of_fetal); write.table(res, file = "hipsc_decs_result.txt", sep = "\t", quote = F)
```

## Evaluation signals

- The loaded reference panel object exists in the R session after data() (e.g. MonacoImmune_main_t_score) and shares human gene symbols with the query matrix
- The deCS.correlation output contains exactly the columns Query, Max_correlation, Top1, Top2, Top3, Cell_labels with one row per query cluster/sample
- Max_correlation values fall in [-1, 1], with confident calls typically well above cor_threshold; below-threshold queries are labeled 'Undetermined cells'
- In the iPSC-NPC/neuron benchmark, the output has 24 rows (13 hiPSC-NPC + 11 hiPSC-neuron) and NPC samples map to fetal progenitor/hESC-like types while neurons map to neuronal cell types

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- Annotation accuracy depends strongly on the reference database used; expanding the references is critical for improving accuracy
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells

## Evidence

- [methods] deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal et al.: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal et al."
- [abstract] expanding the references is critical for improving annotation accuracy: "Our results demonstrate that expanding the references is critical for improving annotation accuracy."
- [readme] deCS.correlation computes Pearson correlation with each reference cell type: "we calculate pearson correlation coefficient (PCC) or Spearman's rank correlation coefficient with each of the cell type in the reference dataset"
- [methods] cor_threshold to avoid mis-annotation as Undetermined cells: "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
- [readme] write annotation table with write.table tab-separated without quotes: "write.table(pbmc_deCS_cor_panel_A, file = "pbmc_deCS_result.txt", sep = "\t", quote = F)"
