---
name: reference-based-cell-type-proportion-estimation
description: Use when bulk RNA-seq cohort samples need cell-type proportion estimates
  and a matched snRNA-seq/scRNA-seq reference exists (ideally with overlapping subjects),
  e.g. to test microglia proportions against Braak stage.
when_to_use_negative:
- No single-cell reference profile is available — use BisqueRNA::MarkerBasedDecomposition
  (BisqueMarker) instead, whose estimates are relative abundances that cannot be compared
  across cell types
- The single-cell reference severely underrepresents a cell type of interest, or the
  single-cell composition differs significantly from the true physiological distribution
- Too few individuals have single-cell data, making the reference profile and gene-specific
  transformations unreliable
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: BisqueRNA
  role: Runs ReferenceBasedDecomposition to estimate cell-type proportions from bulk
    ExpressionSet given a single-cell ExpressionSet reference; results retrieved from
    bulk.props
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet container format required for both bulk and single-cell
    inputs to Bisque
- name: BisqueMarker
  role: Marker-based alternative mode (MarkerBasedDecomposition) used for comparison
    when a reference profile is unavailable
  repo: github.com/cozygene/bisque
- name: Bisque
  role: The underlying method/toolkit this skill implements, learning gene-specific
    bulk expression transformations
  repo: github.com/cozygene/bisque
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:8814dc5cce05e0ea364c72d66f223388e5f459a386912edbefee7f38dbfabc60
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# reference-based-cell-type-proportion-estimation

## Summary

Estimates cell-type proportions in bulk RNA-seq samples using Bisque's reference-based decomposition, which integrates snRNA-seq/scRNA-seq reference profiles with gene-specific bulk expression transformations to correct single-cell sequencing biases. This enables downstream association testing of estimated proportions against phenotypes such as Braak stage.

## When to use

Use when bulk RNA-seq cohort samples need cell-type proportion estimates and a matched snRNA-seq/scRNA-seq reference exists (ideally with overlapping subjects), e.g. to test microglia proportions against Braak stage.

## When NOT to use

- No single-cell reference profile is available — use BisqueRNA::MarkerBasedDecomposition (BisqueMarker) instead, whose estimates are relative abundances that cannot be compared across cell types
- The single-cell reference severely underrepresents a cell type of interest, or the single-cell composition differs significantly from the true physiological distribution
- Too few individuals have single-cell data, making the reference profile and gene-specific transformations unreliable

## Inputs

- Bulk RNA-seq expression count matrix (e.g. DLPFC bulk samples) converted to a Biobase ExpressionSet
- snRNA-seq/scRNA-seq reference count matrix with SubjectName and cellType phenoData labels, converted to a Biobase ExpressionSet
- Covariate table (e.g. age, sex, age-squared) and phenotype values (e.g. Braak stage) per bulk sample

## Outputs

- Per-sample cell-type proportion estimates (bulk.props matrix) for all bulk samples
- Regression coefficients and p-values for cell-type proportion vs phenotype associations
- Comparison table of associations across methods

## How to apply

Convert the bulk count matrix and the single-nucleus reference (with SubjectName and cellType phenoData labels) into Biobase ExpressionSet objects, then run BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE) to learn gene-specific bulk expression transformations and retrieve proportions from the returned list's bulk.props slot. Optionally filter unexpressed or zero-variance genes (FilterUnexpressedGenes, FilterZeroVarianceGenes) and restrict to genes overlapping between bulk and single-cell data. Fit linear regressions of each estimated cell-type proportion on the phenotype (e.g. Braak stage) with covariates such as age, sex, and age-squared, and compile coefficients and p-values across cell types. Correctness is judged by whether expected biological trends replicate — e.g. a significant positive microglia–Braak association (p = 0.001) after covariate correction — and, where truth is available, by correlation of estimates with true proportions.

## Related tools

- **BisqueRNA** (Runs ReferenceBasedDecomposition to estimate cell-type proportions from bulk ExpressionSet given a single-cell ExpressionSet reference; results retrieved from bulk.props) — github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet container format required for both bulk and single-cell inputs to Bisque)
- **BisqueMarker** (Marker-based alternative mode (MarkerBasedDecomposition) used for comparison when a reference profile is unavailable) — github.com/cozygene/bisque
- **Bisque** (The underlying method/toolkit this skill implements, learning gene-specific bulk expression transformations) — github.com/cozygene/bisque

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE); props <- res$bulk.props
```

## Evaluation signals

- Decomposition returns non-missing proportions in res$bulk.props for every bulk sample and expected cell type
- Where true proportions exist, validate by correlation: r <- cor(as.vector(ref.based.estimates), as.vector(true.props))
- Replication check: microglia proportion shows a significant positive association with Braak stage after adjusting for sex, age, and age-squared (expected p = 0.001)
- Bisque (reference-based) estimates yield the expected cross-cell-type associations, unlike BisqueMarker estimates, which are relative within each cell type
- ExpressionSet inputs carry SubjectName and cellType phenoData columns and bulk/single-cell genes were successfully overlapped (GetOverlappingGenes)

## Limitations

- If the number of individuals with single-cell data is small, the reference profile and gene-specific transformations may become unreliable
- A key assumption is that single-cell-based estimates of cell proportions accurately reflect the true proportions; accuracy may decrease if single-cell-captured proportions differ significantly from physiological distributions
- Take caution with known significant bias in single-cell measurements, such as severe underrepresentation of a cell type of interest
- BisqueMarker's marker-based mode estimates relative abundance differences that cannot be compared across cell types

## Evidence

- [methods] Reference-based decomposition call: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [methods] ExpressionSet requirement: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [methods] Marker-based alternative: "res <- BisqueRNA::MarkerBasedDecomposition(bulk.eset, markers, weighted=F)"
- [readme] README usage and result slot: [evidence span withheld — anchor status: external_doc]
- [abstract] Method purpose and bias correction: "utilizes single-cell RNA-seq (scRNA-seq) or single-nucleus RNA-seq (snRNA-seq) data to generate a reference expression profile and learn gene-specific bulk expression transformations"
- [discussion] Marker-mode cross-cell-type caveat: "this method estimates relative differences in abundances that cannot be compared across cell types."
- [discussion] Failure mode with few single-cell individuals: "if the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable."
- [methods] Validation by correlation: "r <- cor(as.vector(ref.based.estimates), as.vector(true.props[row.names(ref.based.estimates),colnames(ref.based.estimates)]))"
