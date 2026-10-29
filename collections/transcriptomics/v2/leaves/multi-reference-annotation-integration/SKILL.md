---
name: multi-reference-annotation-integration
description: Use when annotating human scRNA-seq cells or clusters and a single reference
  is insufficient or composition differs from the query, requiring integration of
  annotations across multiple reference panels.
when_to_use_negative:
- Query cells are novel cell types not represented in human reference panels — deCS,
  as a mapping-based annotation toolkit, fails to characterize novel cell types
- Annotating cancer/malignant cells, where expression profile differences between
  healthy and malignant cells make deCS challenging
- Non-human scRNA-seq data, since deCS references (BlueprintEncode, DICE, MonacoImmune,
  HCL, HCAF) are human cell type panels
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: deCS
  role: Core annotation engine; deCS.correlation() matches the query expression matrix
    against each reference panel and integrates multi-reference cell type calls at
    cell or cluster level
  repo: https://github.com/bsml320/deCS
- name: Seurat
  role: Preprocessing of the query scRNA-seq count matrix (NormalizeData, FindVariableFeatures,
    ScaleData, RunPCA) to produce the scaled markers_expression input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:284d5d63610d37d49c83124577e0920ba2bb0ae10c15ad1b1a4ec82bbc1390ff
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multi-reference-annotation-integration

## Summary

Annotate scRNA-seq cells or clusters by running deCS.correlation() against multiple built-in human reference panels (BlueprintEncode, MonacoImmune, DICE, Human cell landscape, HCAF) and integrating the per-reference top matches into a consensus Cell_labels call, optionally flagging low-confidence cells via cor_threshold.

## When to use

Use when annotating human scRNA-seq cells or clusters and a single reference is insufficient or composition differs from the query, requiring integration of annotations across multiple reference panels.

## When NOT to use

- Query cells are novel cell types not represented in human reference panels — deCS, as a mapping-based annotation toolkit, fails to characterize novel cell types
- Annotating cancer/malignant cells, where expression profile differences between healthy and malignant cells make deCS challenging
- Non-human scRNA-seq data, since deCS references (BlueprintEncode, DICE, MonacoImmune, HCL, HCAF) are human cell type panels

## Inputs

- Scaled gene expression matrix (per-cell or cluster-level z-score of log2-transformed average expression of marker genes) from Seurat preprocessing
- deCS built-in reference panels (BlueprintEncode_main/fine, DICE_main/fine, MonacoImmune_main/fine, Human_cell_landscape, Human_cell_atlas_of_fetal) loaded via data()
- Optionally, deposited predefined cell labels (e.g., all.cell.annotation.meta.txt from covid_balf) for recall evaluation

## Outputs

- Per-query annotation table with Max_correlation, Top1/Top2/Top3 cell types, and consensus Cell_labels (e.g., balf_decs_depth_recall_results.csv)
- Depth-stratified recall comparison of annotation accuracy against predefined labels
- 'Undetermined cells' flags for queries below cor_threshold

## How to apply

Preprocess the query count matrix in Seurat (CreateSeuratObject, NormalizeData, FindVariableFeatures, ScaleData, RunPCA) and supply the scaled expression matrix (or cluster-level z-scored marker expression) as markers_expression. Load the deCS built-in references with data() calls (e.g., data(BlueprintEncode_main), data(MonacoImmune_fine), data(Human_cell_landscape)) and run deCS.correlation(markers_expression, ref_panel) once per panel, recording Top1–Top3 matches and Cell_labels per query. Apply the cor_threshold parameter so low-confidence queries are labeled 'Undetermined cells' rather than mis-annotated, since the paper shows expanding references improves accuracy. For expression-profile input the function computes Pearson or Spearman correlation against each cell type in the reference panel and reports the most relevant cell type(s). Integrate results across panels (e.g., BlueprintEncode, MonacoImmune, HCL) by reconciling per-reference labels, then validate recall against deposited predefined labels, stratifying by condition or sequencing depth (low/median/high UMI) if relevant.

## Related tools

- **deCS** (Core annotation engine; deCS.correlation() matches the query expression matrix against each reference panel and integrates multi-reference cell type calls at cell or cluster level) — https://github.com/bsml320/deCS
- **Seurat** (Preprocessing of the query scRNA-seq count matrix (NormalizeData, FindVariableFeatures, ScaleData, RunPCA) to produce the scaled markers_expression input)

## Examples

```
data(MonacoImmune_main); pbmc_deCS_cor_panel_A <- deCS.correlation(pbmc_cluster_marker_z_score, MonacoImmune_main_t_score)
```

## Evaluation signals

- Output annotation table covers every query (e.g., all 63,103 COVID-19 BIC cells) with a Cell_labels value or 'Undetermined cells'
- Majority of single cells match their best cell type against the deposited predefined labels (cf. Figure 4B of the deCS paper)
- Depth-stratified recall shows only slight accuracy loss at low UMI counts, with minor confusions between closely related types (e.g., ~10 B cells annotated as macrophages)
- Max_correlation values are in a plausible range (PBMC example shows ~0.70–0.89) and low-correlation queries are correctly flagged as Undetermined rather than forced assignments
- Cross-reference consistency: consensus Cell_labels agree across BlueprintEncode, MonacoImmune, and HCL runs for unambiguous cell types

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells
- Performance depends strongly on the reference database used, so expanding and integrating references is critical for accuracy

## Evidence

- [methods] cor_threshold: "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
- [readme] deCS references: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal"
- [readme] deCS.correlation method: "we calculate pearson correlation coefficient (PCC) or Spearman's rank correlation coefficient with each of the cell type in the reference dataset, then the most relevant cell type(s) will be"
- [intro] integration need: "a computational method that can efficiently integrate the annotation results among multiple references is urgently needed"
- [intro] accuracy depends on reference: "a large difference in performance among these methods, and such performance was strongly dependent on the reference database used"
