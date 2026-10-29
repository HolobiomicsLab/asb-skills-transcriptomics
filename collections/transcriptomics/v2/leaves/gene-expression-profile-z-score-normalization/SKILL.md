---
name: gene-expression-profile-z-score-normalization
description: Use when preparing a Seurat-clustered scRNA-seq query whose top-N marker
  genes' average expression must be z-scored (e.g. via t(scale(t(...))) in R) before
  running deCS.correlation against a reference panel.
when_to_use_negative:
- The query input is already a scaled/z-scored matrix (e.g. pbmc_cluster_marker_z_score
  from the PBMC example) — re-scaling is unnecessary and may distort values
- The query is a marker gene list rather than an expression profile — use deCS.fisher's
  exact-test enrichment instead of the correlation workflow
- The goal is single-cell-level rather than cluster-level annotation — deCS defines
  cell types at the cell cluster level
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: deCS
  role: Consumes the z-scored marker expression matrix via deCS.correlation() to assign
    cell type labels by Pearson correlation against reference panels (e.g. MonacoImmune_main)
  repo: https://github.com/bsml320/deCS
- name: Seurat
  role: Upstream normalization, clustering, and marker identification whose AverageExpression
    output feeds the z-score step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:9d1c7617e93355e6816df43c610d121bb4b65840d562be9c13460db9cb4b6d19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - Seurat
  - deCS
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-expression-profile-z-score-normalization

## Summary

Converts cluster-level marker-gene average expression into a scaled (z-scored) matrix with clusters as columns, the required input format for deCS.correlation-based cell type annotation in the deCS scRNA-seq annotation pipeline.

## When to use

Use when preparing a Seurat-clustered scRNA-seq query whose top-N marker genes' average expression must be z-scored (e.g. via t(scale(t(...))) in R) before running deCS.correlation against a reference panel.

## When NOT to use

- The query input is already a scaled/z-scored matrix (e.g. pbmc_cluster_marker_z_score from the PBMC example) — re-scaling is unnecessary and may distort values
- The query is a marker gene list rather than an expression profile — use deCS.fisher's exact-test enrichment instead of the correlation workflow
- The goal is single-cell-level rather than cluster-level annotation — deCS defines cell types at the cell cluster level

## Inputs

- Seurat object with normalized data and defined clusters (from NormalizeData/FindVariableFeatures/ScaleData/RunPCA/FindNeighbors/FindClusters)
- Cluster marker table from FindAllMarkers (only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)
- Cluster-average expression matrix from AverageExpression, log2(x + 1) transformed
- Union of top-N (e.g. top 10 avg_log2FC) marker genes per cluster

## Outputs

- Z-scored marker-gene expression matrix (genes as rows, clusters as columns), e.g. pbmc_cluster_marker_z_score
- Downstream: deCS.correlation annotation table with Query, Max_correlation, Top1, Top2, Top3, Cell_labels columns

## How to apply

First process raw data through the standard Seurat pipeline (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors, FindClusters), then extract cluster markers with FindAllMarkers(only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25) and keep the top 10 markers per cluster by avg_log2FC. Compute the log2 cluster-average expression, restrict rows to the union of the selected marker genes, and z-score each gene's profile across clusters using the transpose-scale-transpose idiom t(scale(t(as.data.frame(pbmc_cluster_marker_average)))). The rationale is that deCS.correlation expects a scaled marker-gene expression matrix (genes x clusters) so that Pearson correlation against a pre-calculated cell type specificity score reference panel (e.g. MonacoImmune_main t-score) is computed on comparable scales. Verify the result has clusters as columns (e.g. Naive CD4 T, CD14+ Mono, B, NK, Platelet) and row values matching the example (e.g. RHOC = -0.758 for Naive CD4 T) before passing it to deCS.correlation.

## Related tools

- **deCS** (Consumes the z-scored marker expression matrix via deCS.correlation() to assign cell type labels by Pearson correlation against reference panels (e.g. MonacoImmune_main)) — https://github.com/bsml320/deCS
- **Seurat** (Upstream normalization, clustering, and marker identification whose AverageExpression output feeds the z-score step)

## Examples

```
pbmc_cluster_marker_z_score = t(scale(t(as.data.frame(pbmc_cluster_marker_average))))
```

## Evaluation signals

- Output matrix has clusters as columns and only the union of top-10 marker genes as rows, matching the pbmc_cluster_marker_z_score head() values (e.g. RHOC = -0.7581827 for Naive CD4 T, CD2 = 1.7349728 for Memory CD4 T)
- Each gene row across clusters has mean approximately 0 and unit standard deviation after t(scale(t(...)))
- Downstream deCS.correlation runs without error on the PBMC query and returns per-cluster Max_correlation values in the plausible ~0.4-0.9 range (e.g. NK = 0.8930805)
- Resulting Cell_labels match expected assignments: Naive/Memory CD4 T -> CD4+ T cells, B -> B cells, NK -> NK cells, CD14+/FCGR3A+ Mono -> Monocytes
- write.table of the result with sep = '\t', quote = FALSE produces a well-formed tab-delimited table

## Limitations

- Quality of the z-scored matrix depends on upstream unsupervised clustering and feature selection (FindVariableFeatures, FindAllMarkers cutoffs); deCS results can be biased when feature selection is poor
- The downstream deCS annotation does not take tissue information (except for HCAF) into account, which may bias results
- deCS fails to characterize novel cell types as a mapping-based annotation toolkit and is challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells

## Evidence

- [readme] Union of marker genes and z-score calculation (readme): [evidence span withheld — anchor status: external_doc]
- [readme] Input format expectation for deCS.correlation (readme): "`markers_expression` is scaled gene expression matrix for human scRNA-seq or bulk RNA-seq, users can see detail preprocess steps from"
- [methods] Upstream Seurat normalization pipeline (methods): "NormalizeData() %>%
  FindVariableFeatures() %>%
  ScaleData() %>%
  RunPCA()"
- [readme] deCS purpose grounded in expression profiles (readme): "we present deCS, for automatic cell type annotations based on a comprehensive collection of human cell type expression profiles"
