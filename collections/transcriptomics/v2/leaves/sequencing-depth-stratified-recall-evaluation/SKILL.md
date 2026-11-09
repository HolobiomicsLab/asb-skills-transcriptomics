---
name: sequencing-depth-stratified-recall-evaluation
description: Use when a large single-cell dataset with deposited predefined cell labels
  is available and you need to quantify whether per-cell UMI counts (sequencing depth)
  bias the accuracy of an automatic annotation tool such as deCS.correlation().
when_to_use_negative:
- The query dataset lacks deposited predefined cell labels, so recall cannot be computed
  and only annotation output is possible
- Cell type annotation is intended at cluster level rather than single-cell resolution,
  since this skill evaluates per-cell depth effects
- The tissue or cell types of interest are not represented in the available reference
  panels (BlueprintEncode, DICE, MonacoImmune, HCL, HCAF), which the article notes
  may bias annotation results
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: deCS
  role: Annotates each cell via deCS.correlation() against multiple reference panels
    whose recall is then compared across sequencing-depth strata
  repo: https://github.com/bsml320/deCS
- name: Seurat
  role: Preprocesses the BIC count matrix (NormalizeData, FindVariableFeatures, ScaleData,
    RunPCA) to produce the scaled expression matrix used as deCS input and provides
    per-cell UMI counts for depth binning
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:582476f12594589924c97b6aae57b0e7a1e767311a393b7af4ebb5b4438deee3
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# sequencing-depth-stratified-recall-evaluation

## Summary

Evaluates how sequencing depth affects automatic cell type annotation accuracy by splitting scRNA-seq cells into low/median/high UMI-count groups and computing annotation recall against predefined labels. The deCS paper applied this to 63,103 bronchoalveolar immune cells (BICs) from COVID-19 patients to show low-depth cells are annotated only slightly less accurately.

## When to use

Use when a large single-cell dataset with deposited predefined cell labels is available and you need to quantify whether per-cell UMI counts (sequencing depth) bias the accuracy of an automatic annotation tool such as deCS.correlation().

## When NOT to use

- The query dataset lacks deposited predefined cell labels, so recall cannot be computed and only annotation output is possible
- Cell type annotation is intended at cluster level rather than single-cell resolution, since this skill evaluates per-cell depth effects
- The tissue or cell types of interest are not represented in the available reference panels (BlueprintEncode, DICE, MonacoImmune, HCL, HCAF), which the article notes may bias annotation results

## Inputs

- scRNA-seq count matrix with predefined cell labels (e.g., 63,103 COVID-19 BIC cells from healthy n=3, moderate n=3, severe n=6 patients)
- Per-cell UMI counts (sequencing depth metadata) for depth stratification
- deCS built-in reference panels loaded via data(): BlueprintEncode_main, BlueprintEncode_fine, MonacoImmune_main, MonacoImmune_fine, Human_cell_landscape (HCL_z_score)
- Seurat-normalized and scaled per-cell expression matrix (markers_expression input)

## Outputs

- Per-cell annotation table for all 63,103 cells (top matches, Cell_labels per reference and integrated)
- Depth-stratified recall comparison table (recall per low/median/high UMI group per reference and integrated result), e.g., balf_decs_depth_recall_results.csv

## How to apply

Download the labeled scRNA-seq dataset (here the 63,103-cell COVID-19 BALF BIC dataset from github.com/zhangzlab/covid_balf) and preprocess it in Seurat with CreateSeuratObject, NormalizeData, FindVariableFeatures, ScaleData, and RunPCA. Run deCS.correlation() at single-cell resolution, passing the per-cell scaled expression matrix as markers_expression and each built-in reference panel in turn (BlueprintEncode_main, BlueprintEncode_fine, MonacoImmune_main, MonacoImmune_fine, HCL_z_score via data(Human_cell_landscape)), using cor_threshold to flag low-confidence cells as 'Undetermined cells' and integrating annotations across references. Bin cells into low, median, and high UMI-count depth groups based on per-cell total counts. Within each depth group, compute annotation recall against the predefined deposited labels for each reference and the integrated result. Write per-cell annotations and the depth-stratified recall table to CSV (e.g., balf_decs_depth_recall_results.csv) and check the trend matches the reported finding that low depth has only slight impact with minor confusions between closely related types.

## Related tools

- **deCS** (Annotates each cell via deCS.correlation() against multiple reference panels whose recall is then compared across sequencing-depth strata) — https://github.com/bsml320/deCS
- **Seurat** (Preprocesses the BIC count matrix (NormalizeData, FindVariableFeatures, ScaleData, RunPCA) to produce the scaled expression matrix used as deCS input and provides per-cell UMI counts for depth binning)

## Examples

```
pbmc <- CreateSeuratObject(counts = pbmc.data) %>% NormalizeData() %>% FindVariableFeatures() %>% ScaleData() %>% RunPCA(); deCS.correlation(pbmc@assays$RNA@scale.data, MonacoImmune_main)
```

## Evaluation signals

- Output annotation table contains exactly 63,103 rows/cells, matching the BIC test dataset size
- Recall is computed within each of the three UMI-depth groups (low, median, high) against the predefined labels from covid_balf
- Depth-stratified recall reproduces the paper's trend: low sequencing depth has only a slight impact on accuracy, with well-annotated cell types across all depth groups
- Misclassified cells concentrate in closely related types (e.g., ~10 B cells annotated as macrophages), not random cross-lineage errors
- Low-confidence cells are consistently flagged as 'Undetermined cells' by the cor_threshold parameter in every reference run

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- Minor misclassifications occur between closely related cell types (e.g., B cells annotated as macrophages at low depth)

## Evidence

- [abstract] systematically evaluated the annotation accuracy under different conditions, including reference panels, sequencing depth, and feature selection strategies: "systematically evaluated the annotation accuracy under different conditions, including reference panels, sequencing depth, and feature selection strategies"
- [methods] The raw data and predefined label are available from Github covid_balf: "The raw data and predefined label are available from [Github](https://github.com/zhangzlab/covid_balf)"
- [methods] give cor_threshold to avoid mis-annotation (regard as Undetermined cells): "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
- [readme] deCS collected several cell type reference panels, including BlueprintEncode, DICE, MonacoImmune, human cell landscape, human cell atlas of fetal: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal"
- [discussion] deCS does not take tissue information (except for HCAF) into account which may bias the annotation results: "deCS does not take tissue information (except for HCAF) into account which may bias the annotation results"
