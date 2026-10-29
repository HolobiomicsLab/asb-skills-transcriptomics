---
name: marker-based-cell-type-decomposition
description: Use when a bulk RNA-seq cohort needs cell-type proportion estimates but
  no matched single-cell reference profile exists and only cell-specific marker genes
  (with weights such as avg_logFC) are available.
when_to_use_negative:
- A matched single-cell/snRNA-seq reference profile is available for the same tissue
  — use reference-based decomposition (ReferenceBasedDecomposition) instead, which
  is more accurate with snRNA-seq data
- Absolute proportions comparable across cell types are required — BisqueMarker estimates
  relative abundance differences that cannot be compared between cell types
- The bulk and single-cell data come from different tissues — the README assumes single-cell
  and bulk counts are measured from the same tissue
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BisqueRNA
  role: Provides MarkerBasedDecomposition, the core weighted-PCA marker-based decomposition
    call returning bulk.props
  repo: github.com/cozygene/bisque
- name: BisqueMarker
  role: The semi-supervised model that extracts trends in cellular composition from
    normalized bulk expression using only cell-specific marker genes
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Supplies the ExpressionSet format required for bulk.eset input to BisqueRNA
- name: Bisque
  role: Toolkit housing both reference-based and marker-based decomposition modes
  repo: github.com/cozygene/bisque
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:f1b789fe151cdcec68daed585d112680f587e278245199692edb6ac98b3b1342
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - Bisque
  - BisqueMarker
  - BisqueRNA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# marker-based-cell-type-decomposition

## Summary

BisqueMarker's semi-supervised marker-based decomposition estimates cell-type relative abundance from normalized bulk RNA-seq expression using only known marker genes, via weighted PCA per cell type with a global first-PC covariate correction — used when no single-cell reference profile is available.

## When to use

Use when a bulk RNA-seq cohort needs cell-type proportion estimates but no matched single-cell reference profile exists and only cell-specific marker genes (with weights such as avg_logFC) are available.

## When NOT to use

- A matched single-cell/snRNA-seq reference profile is available for the same tissue — use reference-based decomposition (ReferenceBasedDecomposition) instead, which is more accurate with snRNA-seq data
- Absolute proportions comparable across cell types are required — BisqueMarker estimates relative abundance differences that cannot be compared between cell types
- The bulk and single-cell data come from different tissues — the README assumes single-cell and bulk counts are measured from the same tissue

## Inputs

- Bulk RNA-seq expression matrix convertible to a Biobase ExpressionSet (bulk.eset)
- Marker gene data frame with gene, cluster (cell type), and log-fold-change weight columns (e.g., from Seurat findMarkers output)

## Outputs

- bulk.props: relative abundance estimates per cell type per bulk sample
- bisque_marker_estimates.csv (per source card: table of all sample x cell-type estimates)
- Correlation r between marker-based estimates and scaled true proportions

## How to apply

Convert the bulk expression matrix to a Biobase ExpressionSet and assemble a marker data frame with gene, cluster, and log-fold-change weight columns. Restrict to one unique marker set per cell type (GetUniqueMarkers) and apply min_gene/max_gene marker-count selection that maximizes the first/second eigenvalue ratio of the marker expression matrix. Scale each marker-gene expression column by its weight (weighted mode, w_col = avg_logFC) and run PCA (svd/prcomp) over bulk samples, extracting the first principal component as the relative abundance estimate per cell type. Apply a covariate correction across samples using the first global expression PC so estimates do not reflect technical variation. Results land in res$bulk.props of the list returned by BisqueRNA::MarkerBasedDecomposition. Validate by correlating marker-based estimates with scaled true proportions (t(scale(t(true.props)))) and requiring a positive r consistent with the vignette's reported R value.

## Related tools

- **BisqueRNA** (Provides MarkerBasedDecomposition, the core weighted-PCA marker-based decomposition call returning bulk.props) — github.com/cozygene/bisque
- **BisqueMarker** (The semi-supervised model that extracts trends in cellular composition from normalized bulk expression using only cell-specific marker genes) — github.com/cozygene/bisque
- **Biobase** (Supplies the ExpressionSet format required for bulk.eset input to BisqueRNA)
- **Bisque** (Toolkit housing both reference-based and marker-based decomposition modes) — github.com/cozygene/bisque

## Examples

```
res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F); props <- res$bulk.props
```

## Evaluation signals

- Correlation between marker-based estimates and scaled true proportions (t(scale(t(true.props)))) is positive and consistent with the vignette's reported R value
- Marker-count selection (min_gene/max_gene) maximizes the first/second eigenvalue ratio of the marker expression matrix
- All returned estimates are present in res$bulk.props for every sample x cell-type combination
- Global first-PC covariate correction applied so estimates do not track technical variation in bulk expression
- In the adipose application, estimates replicate reported associations (e.g., T-cell vs Matsuda index p = 0.002) via marker-based decomposition

## Limitations

- BisqueMarker estimates relative abundance differences that cannot be compared across cell types
- Marker-based estimates are trends in abundance, not true physiological proportions
- If marker genes are poorly chosen or weights (avg_logFC) are unreliable, the first-PC abundance signal may not reflect cell composition

## Evidence

- [intro] Semi-supervised marker model: "we propose BisqueMarker, a semi-supervised model that extracts trends in cellular composition from normalized bulk expression samples using only cell-specific marker genes"
- [discussion] Weighted-PCA estimator description: [evidence span withheld — anchor status: not_found]
- [methods] Method call: "res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F)"
- [readme] Marker mode description in README: [evidence span withheld — anchor status: external_doc]
- [readme] Results access: [evidence span withheld — anchor status: external_doc]
