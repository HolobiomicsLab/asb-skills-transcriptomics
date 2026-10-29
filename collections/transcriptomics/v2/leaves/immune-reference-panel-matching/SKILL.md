---
name: immune-reference-panel-matching
description: Use when scRNA-seq clusters (e.g., PBMC 3k with 9 Seurat clusters) need
  automatic cell type labels against a curated human immune reference panel, or when
  comparing concordance of correlation- vs enrichment-based annotation approaches.
when_to_use_negative:
- The query contains novel cell types absent from the reference panels — deCS is a
  mapping-based annotation toolkit and fails to characterize novel cell types
- Annotating cancer/malignant cells, where expression profile differences between
  healthy and malignant cells make deCS challenging
- Tissue-specific annotation is required — deCS does not take tissue information (except
  HCAF) into account, which may bias results
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: deCS
  role: Core R package providing deCS.correlation() and deCS.fisher() for cluster-level
    cell type annotation against built-in immune reference panels
  repo: https://github.com/bsml320/deCS
- name: Seurat
  role: Preprocessing and clustering of the query scRNA-seq data (NormalizeData, FindVariableFeatures,
    ScaleData, RunPCA, FindNeighbors, FindClusters, FindAllMarkers, AverageExpression)
- name: dplyr
  role: Selecting top-10 markers per cluster by avg_log2FC and constructing the marker
    gene list
- name: reshape2
  role: Required dependency (>= 1.4.4) for deCS plotting/reshaping annotation output
  repo: https://github.com/bsml320/deCS
- name: ggplot2
  role: Required dependency (>= 3.3.2) for rendering the annotation heatmap (deCS_Cor_annotation_A.pdf)
  repo: https://github.com/bsml320/deCS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:ebc665419d1e91240510d4e21e00f67387d3681229951ad51846ba257ca38d0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - Seurat
  - deCS
  - dplyr
  - ggplot2
  - reshape2
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# immune-reference-panel-matching

## Summary

Annotates scRNA-seq cell clusters by matching cluster-level marker expression (z-scores) or top marker gene lists against pre-calculated immune reference panels such as MonacoImmune (10 main / 29 fine cell types), using deCS's two statistical approaches: Pearson correlation (PCC) and Fisher's exact test enrichment.

## When to use

Use when scRNA-seq clusters (e.g., PBMC 3k with 9 Seurat clusters) need automatic cell type labels against a curated human immune reference panel, or when comparing concordance of correlation- vs enrichment-based annotation approaches.

## When NOT to use

- The query contains novel cell types absent from the reference panels — deCS is a mapping-based annotation toolkit and fails to characterize novel cell types
- Annotating cancer/malignant cells, where expression profile differences between healthy and malignant cells make deCS challenging
- Tissue-specific annotation is required — deCS does not take tissue information (except HCAF) into account, which may bias results

## Inputs

- Cluster-average z-scored marker expression matrix (e.g., pbmc_cluster_marker_z_score from pbmc_example.rda)
- Top-10 cluster marker gene list (e.g., pbmc_top10_markers_list from FindAllMarkers with min.pct=0.25, logfc.threshold=0.25)
- Built-in deCS reference panels: MonacoImmune_main_t_score, MonacoImmune_fine_t_score, BlueprintEncode, DICE, Human_cell_landscape, Human_cell_atlas_of_fetal
- CellMatch marker gene database (CellMatch_markers)

## Outputs

- Annotation table per cluster with Max_correlation, Top1/Top2/Top3 cell types, and final Cell_labels (written via write.table to pbmc_deCS_result.txt, tab-separated, no quotes)
- PCC/IR concordance heatmap (deCS_Cor_annotation_A.pdf, 8 x 8 inches)
- Fisher's exact test enrichment results with p-values and cell type fractions

## How to apply

Preprocess the query with the standard Seurat pipeline (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors dims=1:10, FindClusters resolution=0.5), then either (a) compute cluster-average log2 expression for union of top-10 markers (avg_log2FC), z-score it per gene, and run deCS.correlation() with markers_expression and ref_panel = MonacoImmune_main_t_score (or MonacoImmune_fine_t_score for fine labels); or (b) build the top-10 marker list per cluster and run deCS.fisher() with the same ref_panel. Fisher runs use parameters such as p_threshold = 0.01 with cell_type_threshold = 0.5, or p_threshold = 1e-3 with cell_type_threshold = 0.05, and a ref_panel_markers_ratio cutoff (e.g., top 5% genes by t-score/z-score as CTgenes); a cor_threshold guards against mis-annotation ('Undetermined cells'). Rationale: deCS annotates at cluster level rather than single-cell level, integrating multiple references (BlueprintEncode, DICE, MonacoImmune, HCL, HCAF) to reduce computation time and improve accuracy when query and reference cell type compositions are not conserved.

## Related tools

- **deCS** (Core R package providing deCS.correlation() and deCS.fisher() for cluster-level cell type annotation against built-in immune reference panels) — https://github.com/bsml320/deCS
- **Seurat** (Preprocessing and clustering of the query scRNA-seq data (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters, FindAllMarkers, AverageExpression))
- **dplyr** (Selecting top-10 markers per cluster by avg_log2FC and constructing the marker gene list)
- **reshape2** (Required dependency (>= 1.4.4) for deCS plotting/reshaping annotation output) — https://github.com/bsml320/deCS
- **ggplot2** (Required dependency (>= 3.3.2) for rendering the annotation heatmap (deCS_Cor_annotation_A.pdf)) — https://github.com/bsml320/deCS

## Examples

```
pbmc_deCS_cor_panel_A <- deCS.correlation(pbmc_cluster_marker_z_score, MonacoImmune_main_t_score); write.table(pbmc_deCS_cor_panel_A, file = "pbmc_deCS_result.txt", sep = "\t", quote = F)
```

## Evaluation signals

- All 9 PBMC clusters must receive identical main Cell_labels from deCS.correlation() and deCS.fisher() with MonacoImmune_main_t_score (PCC/IR concordance)
- Expected main labels: Naive/Memory CD4 T -> CD4+ T cells, CD14+/FCGR3A+ Mono -> Monocytes, B -> B cells, CD8 T -> CD8+ T cells, NK -> NK cells, DC -> Dendritic cells
- Max_correlation values should fall in the ~0.70–0.90 range (e.g., NK 0.8930805, Memory CD4 T 0.8621523, Platelet 0.7281423)
- Most clusters map to a single main label with significantly higher similarity scores than remaining cell types; Platelet -> Progenitors is the expected low-concordance case
- Output table written to pbmc_deCS_result.txt must be tab-separated with quote = F and match the published annotation rows

## Limitations

- deCS does not take tissue information (except HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells
- Performance depends strongly on the reference database used

## Evidence

- [methods] two deCS statistical approaches: "Depending on the type of query data, we implemented two test approaches: Correlation analysis and Fisher's exact test for cell type enrichment analysis."
- [readme] correlation call: "pbmc_deCS_cor_panel_A <- deCS.correlation(pbmc_cluster_marker_z_score, MonacoImmune_main_t_score)"
- [methods] reference panels: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune"
- [intro] cluster-level rationale: "deCS runs fast, allows the integration of cell annotation from multiple references, and defines cell types at the cell cluster level rather than the single-cell level"
- [methods] filter parameters: "p_threshold = 0.01, cell_type_threshold = 0.5"
- [discussion] limitation: "deCS does not take tissue information (except for HCAF) into account which may bias the annotation results"
