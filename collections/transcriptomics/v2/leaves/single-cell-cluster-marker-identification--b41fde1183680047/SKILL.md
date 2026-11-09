---
name: single-cell-cluster-marker-identification
description: Use when a clustered scRNA-seq dataset (e.g., Seurat object with FindClusters-defined
  clusters) needs cluster-level marker genes to feed downstream annotation tools such
  as deCS that accept either gene lists or z-scored expression profiles.
when_to_use_negative:
- The query is already a precomputed marker list or z-scored expression matrix (e.g.,
  pbmc_example.rda) — skip directly to deCS.correlation()/deCS.fisher()
- Cells are unclustered or the analysis is at single-cell rather than cluster level
  — deCS defines cell types at the cluster level and requires predefined clusters
- The target is novel/cancer cell type discovery — deCS is a mapping-based tool limited
  to its reference panels (BlueprintEncode, DICE, MonacoImmune, HCL, HCAF)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Normalization, clustering, and FindAllMarkers-based cluster marker identification
    producing the top-10 marker list and cluster-average expression
- name: deCS
  role: 'Downstream consumer of markers: deCS.fisher() takes the marker gene list,
    deCS.correlation() takes the z-scored marker expression matrix against MonacoImmune
    reference panels'
  repo: https://github.com/bsml320/deCS
- name: dplyr
  role: group_by(cluster) %>% top_n(n = 10, wt = avg_log2FC) selection of top-10 markers
    per cluster
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:143bf2591ec8a94a0cae650e9028c69631d48c370b4376d38f7db88ecca1d7e2
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-cluster-marker-identification

## Summary

Identifies cluster-specific marker genes from a clustered scRNA-seq object (e.g., Seurat PBMC 3k) using FindAllMarkers and derives both a top-10 marker gene list per cluster and a cluster-average z-scored marker expression matrix, the two query inputs required by deCS.correlation() and deCS.fisher() for cell type annotation.

## When to use

Use when a clustered scRNA-seq dataset (e.g., Seurat object with FindClusters-defined clusters) needs cluster-level marker genes to feed downstream annotation tools such as deCS that accept either gene lists or z-scored expression profiles.

## When NOT to use

- The query is already a precomputed marker list or z-scored expression matrix (e.g., pbmc_example.rda) — skip directly to deCS.correlation()/deCS.fisher()
- Cells are unclustered or the analysis is at single-cell rather than cluster level — deCS defines cell types at the cluster level and requires predefined clusters
- The target is novel/cancer cell type discovery — deCS is a mapping-based tool limited to its reference panels (BlueprintEncode, DICE, MonacoImmune, HCL, HCAF)

## Inputs

- Clustered Seurat object (e.g., pbmc3k with 9 clusters from FindClusters resolution 0.5)
- Raw 10X count matrix loaded via Read10X(data.dir = 'Example_data/1.1.PBMC/hg19/')
- Preprocessed example data pbmc_example.rda containing pbmc_cluster_marker_z_score and pbmc_top10_markers_list

## Outputs

- pbmc.markers: data.frame of positive cluster markers (FindAllMarkers, min.pct = 0.25, logfc.threshold = 0.25)
- pbmc_top10_markers_list: top-10 marker genes per cluster by avg_log2FC, formatted for deCS.fisher(markers_list, ref_panel)
- pbmc_cluster_marker_z_score: row-wise z-scored log2-transformed cluster-average expression matrix of union marker genes, for deCS.correlation(markers_expression, ref_panel)

## How to apply

After standard Seurat preprocessing (NormalizeData, FindVariableFeatures, ScaleData, RunPCA) and clustering (FindNeighbors dims 1:10, FindClusters resolution 0.5, RunUMAP), run FindAllMarkers with only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25, then take the top 10 genes per cluster by avg_log2FC via group_by(cluster) %>% top_n(n = 10, wt = avg_log2FC). Separately compute cluster-average expression as log2(AverageExpression(pbmc)[[1]] + 1), subset to the union of top-10 marker genes, and row-wise z-score it with t(scale(t(...))) to obtain pbmc_cluster_marker_z_score. The gene list feeds deCS.fisher() (Fisher's exact test enrichment) while the z-score matrix feeds deCS.correlation() (PCC-based matching against reference panels such as MonacoImmune_main_t_score). Feature selection is essential — the article notes marker choice strongly affects annotation accuracy.

## Related tools

- **Seurat** (Normalization, clustering, and FindAllMarkers-based cluster marker identification producing the top-10 marker list and cluster-average expression)
- **deCS** (Downstream consumer of markers: deCS.fisher() takes the marker gene list, deCS.correlation() takes the z-scored marker expression matrix against MonacoImmune reference panels) — https://github.com/bsml320/deCS
- **dplyr** (group_by(cluster) %>% top_n(n = 10, wt = avg_log2FC) selection of top-10 markers per cluster)

## Examples

```
pbmc.markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25); top10 <- pbmc.markers %>% group_by(cluster) %>% top_n(n = 10, wt = avg_log2FC)
```

## Evaluation signals

- pbmc.markers should contain only positive markers (only.pos = TRUE) with min.pct >= 0.25 and logfc.threshold = 0.25
- pbmc_top10_markers_list should contain exactly 10 genes per each of the 9 PBMC clusters
- pbmc_cluster_marker_z_score rows should be the union of top-10 markers; sanity-check values like RHOC ≈ 2.258 for FCGR3A+ Mono and S100A9 ≈ 2.650 for CD14+ Mono (high positive z-scores for known markers)
- Downstream deCS.correlation() against MonacoImmune_main_t_score should yield the expected labels: CD4+ T cells (Naive/Memory CD4 T), Monocytes (CD14+/FCGR3A+ Mono), B cells, CD8+ T cells, NK cells, Dendritic cells, with Platelet → Progenitors as the low-concordance case
- PCC and Fisher's exact test (deCS.fisher with same ref_panel) must agree on main cell type labels for all 9 clusters

## Limitations

- Marker/feature selection strategy strongly influences downstream annotation accuracy, as deCS performance was evaluated under different feature selection strategies
- deCS fails to characterize novel cell types since it is a mapping-based annotation toolkit dependent on reference panels
- deCS does not take tissue information (except HCAF) into account, which may bias annotation results, and cancer cells are challenging due to expression profile differences from healthy references

## Evidence

- [readme] Identify cluster specific expressed genes: "pbmc.markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)"
- [readme] top-10 marker selection by avg_log2FC: "top10 <- pbmc.markers %>% group_by(cluster) %>% top_n(n = 10, wt = avg_log2FC)"
- [readme] z-score computation of cluster marker averages: [evidence span withheld — anchor status: external_doc]
- [discussion] feature selection is essential for accurate annotation: "In addition, feature selection is also an essential step to obtain the accurate results"
- [intro] two query input modes supported by deCS: "It supports input for either gene expression profiles (e.g., a query scRNA-seq dataset with clusters to be annotated) or list of genes"
