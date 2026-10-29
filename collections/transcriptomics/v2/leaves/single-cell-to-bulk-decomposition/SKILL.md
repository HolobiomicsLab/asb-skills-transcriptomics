---
name: single-cell-to-bulk-decomposition
description: Use when you have bulk RNA-seq expression counts from a large cohort
  plus snRNA-seq/scRNA-seq data from a subset of the same tissue (e.g., DLPFC), and
  need estimated cell-type proportions for phenotype association testing.
when_to_use_negative:
- Only known marker genes are available and cross-cell-type comparison of absolute
  proportions is required — BisqueMarker estimates relative differences in abundances
  that cannot be compared across cell types.
- Very few individuals have single-cell data, making the reference profile and gene-specific
  transformations unreliable.
- Input is already cell-type-level single-cell data with no bulk cohort to decompose.
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: Bisque
  role: Reference-based decomposition of bulk RNA-seq into cell-type proportions using
    a single-cell reference with gene-specific transformations
  repo: github.com/cozygene/bisque
- name: BisqueRNA
  role: R package providing ReferenceBasedDecomposition and MarkerBasedDecomposition;
    results retrieved from bulk.props
  repo: https://CRAN.R-project.org/package=BisqueRNA
- name: Biobase
  role: Provides the ExpressionSet format required for bulk.eset and sc.eset inputs
- name: BisqueMarker
  role: Semi-supervised marker-based mode extracting relative cell-type abundance
    trends from normalized bulk expression using only marker genes
  repo: github.com/cozygene/bisque
- name: MuSiC
  role: Alternative NNLS-based decomposition method used for comparison in DLPFC replication
    (microglia trended positively with Braak stage but did not reach significance)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:724b3bcbb4ee9ca70fcd0d28e4a759f4bc9195ff12f1a622137fc58fa99de68d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - Bisque
  - BisqueMarker
  - BisqueRNA
  - MuSiC
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# single-cell-to-bulk-decomposition

## Summary

Reference-based decomposition of bulk RNA-seq into cell-type proportions using Bisque, which integrates scRNA-seq/snRNA-seq reference profiles with gene-specific bulk expression transformations; includes a marker-based fallback (BisqueMarker) when no reference is available.

## When to use

Use when you have bulk RNA-seq expression counts from a large cohort plus snRNA-seq/scRNA-seq data from a subset of the same tissue (e.g., DLPFC), and need estimated cell-type proportions for phenotype association testing.

## When NOT to use

- Only known marker genes are available and cross-cell-type comparison of absolute proportions is required — BisqueMarker estimates relative differences in abundances that cannot be compared across cell types.
- Very few individuals have single-cell data, making the reference profile and gene-specific transformations unreliable.
- Input is already cell-type-level single-cell data with no bulk cohort to decompose.

## Inputs

- Bulk RNA-seq expression count matrix (converted to a Biobase ExpressionSet, e.g., 628 DLPFC bulk samples)
- snRNA-seq/scRNA-seq counts with cell type and individual (SubjectName) labels, as an ExpressionSet or Seurat object
- Known cell-type marker gene list (for marker-based mode)
- Sample phenotype/covariate table (e.g., Braak stage, sex, age)

## Outputs

- Per-sample cell-type proportion estimates (res$bulk.props) for each decomposition method
- Regression coefficients and p-values linking estimated proportions to phenotypes (e.g., microglia vs Braak stage, p = 0.001 for Bisque)
- Method-comparison table across cell types (e.g., Bisque vs BisqueMarker)

## How to apply

Convert the bulk expression matrix and the single-cell counts (with SubjectName and cellType phenoData) into Biobase ExpressionSet objects, then run BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE) and retrieve estimated proportions from the returned list's bulk.props slot. When no reference profile is available, run BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F) with a known cell-type marker gene list, but note its estimates are relative within each cell type. Bisque learns gene-specific bulk expression transformations that correct for single-cell/bulk technology biases such as nuclear pre-mRNA in snRNA-seq. Downstream, fit linear regressions of each cell type's estimated proportion on the phenotype (e.g., Braak stage in the 628 DLPFC bulk samples) with covariates such as sex, age, and age-squared, and compile coefficients and p-values into a comparison table. Preprocessing helpers include FilterUnexpressedGenes, FilterZeroVarianceGenes, GetOverlappingGenes, and GetOverlappingSamples to harmonize genes and samples between single-cell and bulk data.

## Related tools

- **Bisque** (Reference-based decomposition of bulk RNA-seq into cell-type proportions using a single-cell reference with gene-specific transformations) — github.com/cozygene/bisque
- **BisqueRNA** (R package providing ReferenceBasedDecomposition and MarkerBasedDecomposition; results retrieved from bulk.props) — https://CRAN.R-project.org/package=BisqueRNA
- **Biobase** (Provides the ExpressionSet format required for bulk.eset and sc.eset inputs)
- **BisqueMarker** (Semi-supervised marker-based mode extracting relative cell-type abundance trends from normalized bulk expression using only marker genes) — github.com/cozygene/bisque
- **MuSiC** (Alternative NNLS-based decomposition method used for comparison in DLPFC replication (microglia trended positively with Braak stage but did not reach significance))

## Examples

```
library(BisqueRNA); res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE); props <- res$bulk.props
```

## Evaluation signals

- Replication check: only Bisque (reference-based) microglia–Braak association is significant and positive (p = 0.001) after correcting for sex, age, and age-squared; MuSiC shows the positive trend but not significance.
- BisqueMarker estimates (relative within each cell type) should NOT reproduce the same cross-cell-type association.
- Results are returned in res$bulk.props for both ReferenceBasedDecomposition and MarkerBasedDecomposition.
- Validate decomposition estimates by correlation with known true proportions (e.g., in simulated data via SimulateData).
- Single-cell and bulk data share overlapping genes/samples verified via GetOverlappingGenes and GetOverlappingSamples before decomposition.

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable.
- A key assumption is that single-cell-based estimates of cell proportions accurately reflect the true proportions; accuracy may decrease if single-cell-captured proportions differ from true physiological distributions, e.g., severe underrepresentation of a cell type of interest.
- BisqueMarker estimates relative differences in abundances that cannot be compared across cell types.
- Bisque provides estimates of cell proportions as reported by the single-cell technology used to generate the reference data.

## Evidence

- [methods] Reference-based decomposition call: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [methods] Marker-based decomposition call: "res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F)"
- [methods] ExpressionSet requirement: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [abstract] Gene-specific transformations: [evidence span withheld — anchor status: not_found]
- [discussion] Marker-based mode scope: "it should be noted that this method estimates relative differences in abundances that cannot be compared across cell types."
- [readme] Marker mode description: "This method utilizes marker genes alone to decompose bulk expression when a reference profile is not available."
- [readme] Output slot: [evidence span withheld — anchor status: external_doc]
- [discussion] Small-reference limitation: "if the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable."
