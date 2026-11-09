---
name: eigenvalue-ratio-gene-selection
description: Use when running marker-based decomposition (BisqueRNA::MarkerBasedDecomposition)
  and you must choose how many/which unique markers per cell type support a rank-1
  abundance estimate from bulk expression.
when_to_use_negative:
- A single-cell reference profile is available and reference-based decomposition (ReferenceBasedDecomposition)
  can be used instead
- Absolute proportions comparable across cell types are required — BisqueMarker estimates
  relative abundance differences that cannot be compared across cell types
- Marker genes overlap between cell types and cannot be made unique per cluster via
  GetUniqueMarkers
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BisqueRNA
  role: Provides MarkerBasedDecomposition and the GetUniqueMarkers filtering utility
    used to enforce one unique marker set per cell type before eigenvalue-ratio selection
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Supplies the ExpressionSet container format that the bulk expression matrix
    must be converted into before decomposition
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:26dc4774a60d4202e7a9f92d4cd8d5fff25b603e1e344c7bf405cb840a1c4c2d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - Bisque
  - BisqueRNA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# eigenvalue-ratio-gene-selection

## Summary

A marker-selection step from BisqueMarker that chooses the marker-gene set per cell type by maximizing the first/second eigenvalue ratio of the weighted marker expression matrix, so that the first PC reliably captures cell-type abundance trends rather than noise or competing signals.

## When to use

Use when running marker-based decomposition (BisqueRNA::MarkerBasedDecomposition) and you must choose how many/which unique markers per cell type best support a rank-1 abundance estimate from bulk expression.

## When NOT to use

- A single-cell reference profile is available and reference-based decomposition (ReferenceBasedDecomposition) can be used instead
- Absolute proportions comparable across cell types are required — BisqueMarker estimates relative abundance differences that cannot be compared across cell types
- Marker genes overlap between cell types and cannot be made unique per cluster via GetUniqueMarkers

## Inputs

- Bulk expression matrix (samples x genes), convertible to a Biobase ExpressionSet (bulk.eset)
- Marker data frame with gene, cluster (cell type), and avg_logFC weight columns

## Outputs

- Per-cell-type selected marker gene sets (marker counts maximizing first/second eigenvalue ratio)
- First-PC relative abundance estimates per cell type (bulk.props), written to bisque_marker_estimates.csv

## How to apply

First restrict the marker data frame (gene, cluster, avg_logFC weight columns) to one unique marker set per cell type using GetUniqueMarkers, so no gene is shared between cell types. For each cell type, scale the bulk marker-gene expression columns by their log-fold-change weights (weighted mode, w_col = avg_logFC) and evaluate candidate marker counts: for each candidate set, run PCA (svd/prcomp from the stats package) over bulk samples and compute the ratio of the first to second eigenvalue. Select the marker count that maximizes this first/second eigenvalue ratio, since a dominant leading eigenvalue indicates the marker matrix carries a single strong abundance signal. Then extract the first PC per cell type as the relative abundance estimate and apply the correction regressing out the first global expression PC across samples so estimates do not reflect technical variation.

## Related tools

- **BisqueRNA** (Provides MarkerBasedDecomposition and the GetUniqueMarkers filtering utility used to enforce one unique marker set per cell type before eigenvalue-ratio selection) — github.com/cozygene/bisque
- **Biobase** (Supplies the ExpressionSet container format that the bulk expression matrix must be converted into before decomposition)

## Examples

```
res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=TRUE)
```

## Evaluation signals

- The selected marker set per cell type maximizes the ratio of the first to second eigenvalue of the weighted marker expression matrix relative to other candidate marker counts
- First/second eigenvalue ratio is substantially greater than 1 for the chosen set, indicating a dominant single signal
- Resulting bulk.props estimates show positive Pearson correlation with scaled true proportions, cor(estimates, t(scale(t(true.props))))
- Marker sets contain no gene assigned to more than one cell type after GetUniqueMarkers
- Estimates are written for all samples and all cell types to bisque_marker_estimates.csv with consistent dimensions

## Limitations

- BisqueMarker estimates relative differences in abundances that cannot be compared across cell types (they are not proportions)
- Marker-based decomposition requires a reliable marker gene set; single-cell data is not required but is typically used to identify markers
- Estimates may reflect technical variation if the global first-PC covariate correction is omitted

## Evidence

- [full_text] BisqueMarker estimates cell-type abundance by extracting the first PC from a weighted PCA of each cell type's marker-gene expression matrix: [evidence span withheld — anchor status: not_found]
- [methods] GetUniqueMarkers and eigenvalue ratio selection: "- `GetUniqueMarkers(x, gene_col)`"
- [methods] Marker-based decomposition call: "res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F)"
- [readme] README on marker-based mode scope: [evidence span withheld — anchor status: external_doc]
- [intro] Article description of BisqueMarker as marker-only mode: "we propose BisqueMarker, a semi-supervised model that extracts trends in cellular composition from normalized bulk expression samples using only cell-specific marker genes"
