---
name: weighted-principal-component-analysis
description: Use when bulk expression samples must be decomposed into cell-type relative
  abundances but no single-cell reference profile is available—only marker genes with
  per-gene weights (e.g., log fold-change) per cell type.
when_to_use_negative:
- A matched single-cell reference profile is available — use reference-based decomposition
  (ReferenceBasedDecomposition) instead, which is more accurate when scRNA-seq/snRNA-seq
  data exist
- Absolute proportions comparable across cell types are needed — BisqueMarker only
  estimates relative abundance differences that cannot be compared between cell types
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BisqueRNA
  role: Implements MarkerBasedDecomposition, the weighted-PCA marker-based mode executed
    on the bulk ExpressionSet
  repo: https://github.com/cozygene/bisque
- name: Bisque
  role: Toolkit providing the decomposition framework, including the BisqueMarker
    semi-supervised mode
  repo: https://github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet container required for the bulk.eset input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:21d714875bc4c96d2afa9a005df8827f01aef24269ea520193d3a52d3ba0d286
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cozygene/bisque
  tools:
  - Biobase
  - Bisque
  - BisqueRNA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# weighted-principal-component-analysis

## Summary

A semi-supervised decomposition technique used by BisqueMarker: each cell type's marker-gene expression matrix from bulk RNA-seq samples is scaled by marker weights (e.g., avg_logFC) and reduced via PCA (svd/prcomp), with the first principal component serving as a relative cell-type abundance estimate after correcting for the first global expression PC.

## When to use

Use when bulk expression samples must be decomposed into cell-type relative abundances but no single-cell reference profile is available—only marker genes with per-gene weights (e.g., log fold-change) per cell type.

## When NOT to use

- A matched single-cell reference profile is available — use reference-based decomposition (ReferenceBasedDecomposition) instead, which is more accurate when scRNA-seq/snRNA-seq data exist
- Absolute proportions comparable across cell types are needed — BisqueMarker only estimates relative abundance differences that cannot be compared between cell types

## Inputs

- Bulk expression matrix (genes x samples), converted to a Biobase ExpressionSet (bulk.eset)
- Marker data frame with gene, cluster (cell type), and log-fold-change weight columns

## Outputs

- bulk.props: relative abundance estimates for all samples and cell types (e.g., written to bisque_marker_estimates.csv)
- First principal components per cell type (from svd/prcomp)

## How to apply

Convert the bulk expression matrix to a Biobase ExpressionSet (bulk.eset) and assemble a marker data frame with gene, cluster (cell type), and log-fold-change weight columns. Restrict markers to one unique marker set per cell type via GetUniqueMarkers and apply min_gene/max_gene marker-count selection chosen to maximize the first/second eigenvalue ratio of the marker expression matrix. In weighted mode, scale each marker-gene expression column by its weight (w_col = avg_logFC), then run PCA over bulk samples using svd/prcomp from the stats package. Extract the first principal component as the relative abundance estimate for each cell type, and apply a correction using the first global expression PC as a covariate so estimates do not reflect technical variation. Validate by correlating marker-based estimates with scaled true proportions (t(scale(t(true.props)))) and requiring a positive correlation r consistent with the vignette's reported R value.

## Related tools

- **BisqueRNA** (Implements MarkerBasedDecomposition, the weighted-PCA marker-based mode executed on the bulk ExpressionSet) — https://github.com/cozygene/bisque
- **Bisque** (Toolkit providing the decomposition framework, including the BisqueMarker semi-supervised mode) — https://github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet container required for the bulk.eset input)

## Examples

```
bulk.eset <- Biobase::ExpressionSet(assayData = bulk.matrix); res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=TRUE); write.csv(res$bulk.props, 'bisque_marker_estimates.csv')
```

## Evaluation signals

- Correlation between marker-based bulk.props estimates and scaled true proportions (t(scale(t(true.props)))) is positive and consistent with the vignette's reported R value
- Marker counts per cell type were chosen within min_gene/max_gene bounds to maximize the first/second eigenvalue ratio of the marker expression matrix
- bulk.props contains entries for all samples and all cell types, written successfully to bisque_marker_estimates.csv
- Estimates no longer track the first global expression PC after covariate correction (i.e., technical variation is not driving the abundances)

## Limitations

- BisqueMarker estimates relative differences in abundances that cannot be compared across cell types
- Because it is marker-based and semi-supervised, estimates capture only relative abundance trends, not true physiological proportions

## Evidence

- [other] BisqueMarker estimates cell-type abundance by extracting the first PC from a weighted PCA of each cell type's marker-gene expression matrix: [evidence span withheld — anchor status: not_found]
- [methods] Marker-based decomposition call in methods: "res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F)"
- [intro] BisqueMarker semi-supervised model description: "we propose BisqueMarker, a semi-supervised model that extracts trends in cellular composition from normalized bulk expression samples using only"
- [discussion] Cross-cell-type comparability limitation: "it should be noted that this method estimates relative differences in abundances that cannot be compared across cell types."
- [readme] README description of marker-based mode: "This method utilizes marker genes alone to decompose bulk expression when a reference profile is not available."
