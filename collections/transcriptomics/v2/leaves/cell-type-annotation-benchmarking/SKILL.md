---
name: cell-type-annotation-benchmarking
description: Use when reproducing Table 1-style accuracy comparisons of cell type
  annotators across multiple tissue scRNA-seq datasets, or when evaluating how sequencing
  depth affects single-cell annotation accuracy on a labeled test set.
when_to_use_negative:
- The query involves cancer cells, since deCS is expected to struggle due to expression
  profile differences between healthy and malignant cells
- The goal is discovering novel cell types, since deCS is a mapping-based annotation
  toolkit and fails to characterize novel cell types
- Input cells are not human, as the built-in references (BlueprintEncode, DICE, MonacoImmune,
  HCL, HCAF) are human-only panels
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: deCS
  role: Primary annotator under evaluation; deCS.correlation for expression profiles
    and deCS.fisher for marker gene lists against built-in reference panels
  repo: https://github.com/bsml320/deCS
- name: Seurat
  role: Preprocessing, clustering, and marker identification (NormalizeData, FindVariableFeatures,
    ScaleData, RunPCA, FindNeighbors, FindClusters, FindAllMarkers, AverageExpression)
- name: SingleR
  role: Baseline annotator benchmarked on the same eight test datasets
- name: CHETAH
  role: Baseline annotator benchmarked on the same eight test datasets
- name: scCATCH
  role: Baseline annotator benchmarked on the same eight test datasets
  repo: https://github.com/ZJUFanLab/scCATCH
- name: scPred
  role: Baseline annotator benchmarked on the same eight test datasets
- name: SciBet
  role: Baseline annotator benchmarked on the same eight test datasets
- name: Cell BLAST
  role: Baseline annotator benchmarked on the same eight test datasets
- name: covid_balf
  role: Source of the 63,103-cell bronchoalveolar immune cell test dataset with predefined
    labels and annotation metadata (all.cell.annotation.meta.txt)
  repo: https://github.com/zhangzlab/covid_balf
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:ce7b249c92368c125f7433bffcf7a27f3426d2110f43f5c07b183b1411c9b463
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - CHETAH
  - Cell BLAST
  - CellAssign
  - Harmony
  - LIGER
  - MACA
  - MASI
  - SCCAF
  - Scanorama
  - SciBet
  - Seurat
  - SingleR
  - Symphony
  - covid_balf
  - deCS
  - scArches
  - scCATCH
  - scMatch
  - scNym
  - scPred
  - scVI
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# cell-type-annotation-benchmarking

## Summary

Benchmark deCS cell type annotation against state-of-the-art tools (SingleR, CHETAH, scPred, SciBet, Cell BLAST, etc.) across eight test scRNA-seq datasets and on single-cell-level annotation of the COVID-19 bronchoalveolar immune cell dataset, stratified by sequencing depth.

## When to use

Use when reproducing Table 1-style accuracy comparisons of cell type annotators across multiple tissue scRNA-seq datasets, or when evaluating how sequencing depth affects single-cell annotation accuracy on a labeled test set.

## When NOT to use

- The query involves cancer cells, since deCS is expected to struggle due to expression profile differences between healthy and malignant cells
- The goal is discovering novel cell types, since deCS is a mapping-based annotation toolkit and fails to characterize novel cell types
- Input cells are not human, as the built-in references (BlueprintEncode, DICE, MonacoImmune, HCL, HCAF) are human-only panels

## Inputs

- Test scRNA-seq datasets with predefined cell labels (PBMC 3k from 10X Genomics or pbmc_example.rda, COVID-19 BALF from github.com/zhangzlab/covid_balf, cardiac cells from GEO GSE106118, plus remaining tissue datasets)
- deCS built-in reference panels (BlueprintEncode_main/fine, DICE_main/fine, MonacoImmune_main/fine, Human_cell_landscape, Human_cell_atlas_of_fetal)
- Cluster marker gene lists (top 10 per cluster by avg_log2FC) and cluster-average z-score expression matrices
- CellMatch marker database (CellMatch_markers)

## Outputs

- Per-tool, per-dataset annotation accuracy table reproducing Table 1 (deCS correlation mode averaging ~93.6% on solid tissues)
- Per-cell annotation table with Top1/Top2/Top3 matches, Max_correlation, and Cell_labels for all 63,103 BIC cells
- Depth-stratified recall comparison (low/median/high UMI groups) written to balf_decs_depth_recall_results.csv

## How to apply

Preprocess each test dataset in Seurat (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors(dims=1:10), FindClusters(resolution=0.5)), extract top 10 cluster markers by avg_log2FC via FindAllMarkers(only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25), and compute the cluster-average log2 expression z-score matrix. Run deCS.correlation against built-in reference panels (BlueprintEncode, DICE, MonacoImmune, Human_cell_landscape, Human_cell_atlas_of_fetal) and deCS.fisher on marker gene lists, applying cor_threshold/p_threshold/cell_type_threshold to flag 'Undetermined cells'. Run SingleR, CHETAH, CellAssign, scMatch, scCATCH, scPred, SciBet, scVI, and Cell BLAST on the same data, then compare all annotations against predefined cell labels per dataset to compute accuracy per tool and mode. For the depth analysis, download the 63,103-cell COVID-19 BIC dataset from github.com/zhangzlab/covid_balf, run deCS.correlation at single-cell resolution with each of BlueprintEncode_main/fine, MonacoImmune_main/fine, and HCL_z_score, split cells into low/median/high UMI-count groups, and compute per-group recall. Rationale: the paper shows deCS's correlation mode averages ~93.6% accuracy across solid tissue benchmarks and that low sequencing depth only slightly impacts accuracy (e.g., 10 B cells misclassified as macrophages).

## Related tools

- **deCS** (Primary annotator under evaluation; deCS.correlation for expression profiles and deCS.fisher for marker gene lists against built-in reference panels) — https://github.com/bsml320/deCS
- **Seurat** (Preprocessing, clustering, and marker identification (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters, FindAllMarkers, AverageExpression))
- **SingleR** (Baseline annotator benchmarked on the same eight test datasets)
- **CHETAH** (Baseline annotator benchmarked on the same eight test datasets)
- **scCATCH** (Baseline annotator benchmarked on the same eight test datasets) — https://github.com/ZJUFanLab/scCATCH
- **scPred** (Baseline annotator benchmarked on the same eight test datasets)
- **SciBet** (Baseline annotator benchmarked on the same eight test datasets)
- **Cell BLAST** (Baseline annotator benchmarked on the same eight test datasets)
- **covid_balf** (Source of the 63,103-cell bronchoalveolar immune cell test dataset with predefined labels and annotation metadata (all.cell.annotation.meta.txt)) — https://github.com/zhangzlab/covid_balf

## Examples

```
pbmc_deCS_cor_panel_A <- deCS.correlation(pbmc_cluster_marker_z_score, MonacoImmune_main_t_score)
```

## Evaluation signals

- Reproduced deCS correlation-mode accuracy averages ~93.6% across the eight benchmark datasets, matching Table 1
- Per-cell annotation output covers all 63,103 BIC cells (Figure 4B trend reproduced)
- Depth-stratified recall shows low sequencing depth causes only slight accuracy loss, e.g. ~10 B cells misclassified as macrophages, with other cell types performing well (Figure 5A trend)
- PBMC demo output lists 9 clusters (Naive CD4 T, Memory CD4 T, CD14+ Mono, B, CD8 T, FCGR3A+ Mono, NK, DC, Platelet) with Cell_labels consistent with known PBMC identity, Max_correlation in the ~0.70–0.89 range
- Cells falling below cor_threshold are flagged as 'Undetermined cells' rather than force-assigned, per the deCS mis-annotation control

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells
- Cutoffs must be tuned via cor_threshold/p_threshold/cell_type_threshold to avoid mis-annotation; low-confidence cells become 'Undetermined cells'

## Evidence

- [results] Across the six solid tissue benchmark datasets, the deCS correlation analysis achieved an average annotation accuracy of 93.6%: [evidence span withheld — anchor status: not_found]
- [methods] cor_threshold quote: "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
- [readme] FindAllMarkers parameters: "pbmc.markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)"
- [abstract] reference panels: "Here, we present decoding Cell type Specificity (deCS), an automatic cell type annotation method augmented by a comprehensive collection of human cell type expression profiles and marker genes."
- [results] BIC dataset size and depth finding: [evidence span withheld — anchor status: not_found]
- [discussion] tissue limitation: "First, deCS does not take tissue information (except for HCAF) into account which may bias the annotation results"
