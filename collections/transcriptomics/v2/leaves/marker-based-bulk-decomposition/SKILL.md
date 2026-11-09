---
name: marker-based-bulk-decomposition
description: Use when you have a bulk RNA-seq expression matrix and a marker gene
  list (gene, cluster, optional fold-change weight) per cell type, but no scRNA-seq/snRNA-seq
  reference profile to decompose cell-type composition trends.
when_to_use_negative:
- A snRNA-seq/scRNA-seq reference profile with matched cell type and individual labels
  IS available — use reference-based decomposition (BisqueRNA::ReferenceBasedDecomposition)
  instead, which learns gene-specific transformations and yields more accurate estimates
- You need absolute cell-type proportions comparable across cell types — BisqueMarker
  estimates relative differences in abundance across individuals only, not proportions
- Bulk and single-cell data come from different tissues, violating the same-tissue
  assumption of decomposition
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: BisqueRNA
  role: R package providing MarkerBasedDecomposition, the marker-based semi-supervised
    decomposition function; results in res$bulk.props
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet container format required for bulk expression input
    to Bisque
- name: Bisque
  role: Parent toolkit; BisqueMarker is its marker-only mode of operation when a reference
    profile is unavailable
  repo: github.com/cozygene/bisque
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:a62c6e0d65a14964b052ac0cdf7e7e97449c6e2665da71c690f9bbf06a264587
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# marker-based-bulk-decomposition

## Summary

BisqueMarker (BisqueRNA::MarkerBasedDecomposition) is the semi-supervised mode of Bisque that extracts relative cell-type abundance trends from normalized bulk RNA-seq expression using only cell-specific marker genes, for cases where no single-cell reference profile is available.

## When to use

Use when you have a bulk RNA-seq expression matrix and a marker gene list (gene, cluster, optional fold-change weight) per cell type, but no scRNA-seq/snRNA-seq reference profile to decompose cell-type composition trends.

## When NOT to use

- A snRNA-seq/scRNA-seq reference profile with matched cell type and individual labels IS available — use reference-based decomposition (BisqueRNA::ReferenceBasedDecomposition) instead, which learns gene-specific transformations and yields more accurate estimates
- You need absolute cell-type proportions comparable across cell types — BisqueMarker estimates relative differences in abundance across individuals only, not proportions
- Bulk and single-cell data come from different tissues, violating the same-tissue assumption of decomposition

## Inputs

- Bulk RNA-seq (or microarray) expression matrix with genes as rows and samples as columns, convertible to a Biobase ExpressionSet
- Marker gene data frame with columns: gene, cluster (cell type label), and optional weight column (e.g., fold-change)

## Outputs

- Relative cell-type abundance estimates per sample in the res$bulk.props slot of the returned result list
- Association statistics from regressing cell-type abundances against sample phenotypes (e.g., written to bisque_marker_associations.csv)

## How to apply

Convert the bulk expression matrix to a Biobase ExpressionSet with genes as rows and samples as columns (Biobase::ExpressionSet). Build a marker data frame with gene, cluster (cell type), and optional weight (e.g., fold-change) columns, and restrict markers to those overlapping the bulk genes. Run BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F) — weighted=TRUE folds marker weights (fold-changes) into the model — and extract relative abundances from the returned res$bulk.props slot. Regress the per-cell-type relative abundances against sample phenotypes to detect composition–phenotype associations; validated in adipose tissue against Bisque's reference-based results, reproducing the same significant associations (adipocyte-BMI, macrophage-BMI, T-cell-Matsuda index) reported in Supplementary Table 1.

## Related tools

- **BisqueRNA** (R package providing MarkerBasedDecomposition, the marker-based semi-supervised decomposition function; results in res$bulk.props) — github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet container format required for bulk expression input to Bisque)
- **Bisque** (Parent toolkit; BisqueMarker is its marker-only mode of operation when a reference profile is unavailable) — github.com/cozygene/bisque

## Examples

```
res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F); props <- res$bulk.props
```

## Evaluation signals

- Qualitative acceptance: the same phenotype associations (adipocyte-BMI, macrophage-BMI, T-cell-Matsuda index) are significant under marker-based mode as under reference-based Bisque, per Supplementary Table 1
- res$bulk.props is non-empty and contains rows for each marker-supplied cell type (cluster) and columns for each bulk sample
- Only markers overlapping the bulk gene set are used; verify marker filtering retained the intended overlap
- Estimates are relative abundances (not proportions summing to 1) and should not be compared across cell types
- Association regression output (bisque_marker_associations.csv) contains estimates and significance statistics per cell-type–phenotype pair

## Limitations

- BisqueMarker estimates relative differences in abundances that cannot be compared across cell types — these are relative abundances/trends, not proportions
- Single-cell data is not required but marker gene quality drives performance; markers can be derived from single-cell data or prior knowledge
- Validation of associations is qualitative (same significance pattern as reference-based Bisque) rather than a direct accuracy check against true proportions

## Evidence

- [methods] Marker-based decomposition call: "res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F)"
- [methods] ExpressionSet conversion: "bulk.eset <- Biobase::ExpressionSet(assayData = bulk.matrix)"
- [intro] BisqueMarker description: "we propose BisqueMarker, a semi-supervised model that extracts trends in cellular composition from normalized bulk expression samples using only"
- [readme] Relative abundance caveat: [evidence span withheld — anchor status: external_doc]
- [readme] Results slot: [evidence span withheld — anchor status: external_doc]
- [discussion] Cross-cell-type comparability limitation: "it should be noted that this method estimates relative differences in abundances that cannot be compared across cell types."
