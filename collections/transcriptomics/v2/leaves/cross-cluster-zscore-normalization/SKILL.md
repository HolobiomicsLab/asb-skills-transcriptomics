---
name: cross-cluster-zscore-normalization
description: Use when a query scRNA-seq dataset has been clustered and per-cluster
  marker gene expression must be put on a comparable, standardized scale (z-scores)
  before correlating against deCS reference panels or before ranking cell-type-specific
  genes.
when_to_use_negative:
- The reference or query panel is derived from bulk RNA-seq data — deCS uses regression
  t-statistics (t-score) for bulk references, not z-scores
- The input is already a scaled/normalized z-score matrix (double-scaling would distort
  the cross-cluster contrast)
- The goal is cluster-level annotation at the single-cell level rather than the cluster
  level — deCS defines cell types at the cell cluster level
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: deCS
  role: Consumes the z-scored query matrix via deCS.correlation() and ships pre-computed
    z-score reference panels (Human_cell_landscape, Human_cell_atlas_of_fetal)
  repo: https://github.com/bsml320/deCS
- name: Seurat
  role: Produces the cluster-averaged expression (AverageExpression) and cluster markers
    (FindAllMarkers) that are z-scored row-wise
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:59ff494f54089f5c3fcbeaba44d7d02a90758a91b72be8ce372a90393920aa4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - Seurat
  - deCS
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-cluster-zscore-normalization

## Summary

Row-wise z-score transformation via t(scale(t(...))) of averaged gene expression per cell cluster, used in deCS both to build scRNA-seq reference panels (HCL, HCAF) and to scale query cluster-level expression before correlation-based annotation.

## When to use

Use when a query scRNA-seq dataset has been clustered and per-cluster marker gene expression must be put on a comparable, standardized scale (z-scores) before correlating against deCS reference panels or before ranking cell-type-specific genes.

## When NOT to use

- The reference or query panel is derived from bulk RNA-seq data — deCS uses regression t-statistics (t-score) for bulk references, not z-scores
- The input is already a scaled/normalized z-score matrix (double-scaling would distort the cross-cluster contrast)
- The goal is cluster-level annotation at the single-cell level rather than the cluster level — deCS defines cell types at the cell cluster level

## Inputs

- Seurat object with clusters (e.g., PBMC 3k processed via NormalizeData/FindVariableFeatures/ScaleData/RunPCA/FindNeighbors/FindClusters)
- FindAllMarkers output with top markers selected by avg_log2FC (e.g., top 10 per cluster)
- Cluster-averaged expression matrix restricted to marker genes

## Outputs

- Row-wise z-scored marker-gene-by-cluster matrix (e.g., pbmc_cluster_marker_z_score) suitable as deCS.correlation() input
- Reference z-score matrices such as HCL_z_score and HCAF_z_score (internal deCS built-in data)

## How to apply

After standard Seurat preprocessing (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters) and FindAllMarkers, average expression per cluster with AverageExpression, log2-transform as log(AverageExpression + 1, 2), restrict to top marker genes (e.g., top 10 by avg_log2FC), then apply t(scale(t(...))) to compute row-wise z-scores across clusters. deCS uses the same convention for its single-cell reference panels: cluster-averaged expression from Human_cell_landscape and Human_cell_atlas_of_fetal is z-scored row-wise to yield HCL_z_score and HCAF_z_score matrices, whereas bulk references (BlueprintEncode, DICE, MonacoImmune) use regression t-statistics instead. The z-scored query matrix is then passed to deCS.correlation() for pearson/spearman correlation against a pre-calculated reference panel.

## Related tools

- **deCS** (Consumes the z-scored query matrix via deCS.correlation() and ships pre-computed z-score reference panels (Human_cell_landscape, Human_cell_atlas_of_fetal)) — https://github.com/bsml320/deCS
- **Seurat** (Produces the cluster-averaged expression (AverageExpression) and cluster markers (FindAllMarkers) that are z-scored row-wise)

## Examples

```
pbmc_cluster_average = log(AverageExpression(pbmc)[[1]] + 1, 2); pbmc_cluster_marker_z_score = t(scale(t(as.data.frame(pbmc_cluster_average[which(rownames(pbmc_cluster_average) %in% top10$gene), ]))))
```

## Evaluation signals

- Each row of the z-scored matrix has mean 0 and standard deviation 1 across clusters
- Known marker genes z-score highest in their expected cluster (e.g., S100A9/S100A8 peak in CD14+ Mono at ~2.65, FCER1A peaks in DC per the README output)
- Row count equals the number of top marker genes selected (e.g., top 10 per cluster union), columns equal the number of clusters
- deCS.correlation() returns Max_correlation values in a plausible range (e.g., ~0.70-0.89 for PBMC example) and a Top1 label matching expected cell types
- Within reference panels, each cell type retains the top 5% of genes ranked by z-score/t-score as CTgenes (ref_panel_markers_ratio cutoff)

## Limitations

- deCS does not take tissue information into account (except for HCAF), which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types absent from the reference panels
- Annotation may be challenging for cancer cells due to expression profile differences between healthy and malignant cells

## Evidence

- [readme] Cluster-averaged expression is log2-transformed then row-wise z-scored: "pbmc_cluster_average = log(AverageExpression(pbmc)[[1]] + 1, 2)
pbmc_cluster_marker_z_score = t(scale(t(as.data.frame(pbmc_cluster_marker_average))))"
- [methods] deCS ranks cell-type-specific genes by t-score/z-score with a top-5% cutoff: "We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes"
- [readme] deCS collects multiple reference panels including single-cell ones: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal"
- [intro] deCS annotation operates at the cluster level and integrates multiple references: "deCS runs fast, allows the integration of cell annotation from multiple references, and defines cell types at the cell cluster level rather than the single-cell level"
- [discussion] deCS limitations regarding tissue information: "deCS does not take tissue information (except for HCAF) into account which may bias the annotation results"
