---
name: gene-specific-bulk-expression-transformation
description: Use when decomposing bulk RNA-seq into cell-type proportions using a
  scRNA-seq/snRNA-seq reference where technical differences between single-cell and
  bulk chemistries introduce gene-specific biases that break direct proportionality
  assumptions.
when_to_use_negative:
- No single-cell reference profile is available and only known marker genes exist
  — use BisqueMarker (MarkerBasedDecomposition) instead, noting its abundances cannot
  be compared across cell types
- Very few individuals have single-cell data, making the reference profile and gene-specific
  transformations unreliable
- Known severe bias in the single-cell measurements, e.g. a cell type of interest
  is severely underrepresented relative to true physiological composition
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: BisqueRNA
  role: R package implementing ReferenceBasedDecomposition with the SupervisedTransformBulk/SemisupervisedTransformBulk
    gene-specific transformations and NNLS decomposition
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet container required for both bulk and single-cell
    inputs
- name: limSolve
  role: Solves the per-sample non-negative least-squares decomposition enforcing non-negative
    coefficients summing to one
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:f398aacad39ca2d4d2047148ec2112adf1aad52f1f12321122dc00ed3225fcb9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSEQ-sc
  - Biobase
  - Bisque
  - BisqueRNA
  - CIBERSORT
  - MuSiC
  - limSolve
  - stats
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# gene-specific-bulk-expression-transformation

## Summary

Bisque's core statistical step: per-gene regression that learns transformations of the bulk expression scale to correct gene-specific biases between single-cell/nucleus reference profiles and bulk RNA-seq, followed by NNLS decomposition to estimate cell-type proportions.

## When to use

Use when decomposing bulk RNA-seq into cell-type proportions using a scRNA-seq/snRNA-seq reference where technical differences between single-cell and bulk chemistries introduce gene-specific biases that break direct proportionality assumptions.

## When NOT to use

- No single-cell reference profile is available and only known marker genes exist — use BisqueMarker (MarkerBasedDecomposition) instead, noting its abundances cannot be compared across cell types
- Very few individuals have single-cell data, making the reference profile and gene-specific transformations unreliable
- Known severe bias in the single-cell measurements, e.g. a cell type of interest is severely underrepresented relative to true physiological composition

## Inputs

- Bulk RNA-seq count matrix as a Biobase ExpressionSet (bulk.eset)
- scRNA-seq/snRNA-seq counts with cellType and SubjectName labels as a Biobase ExpressionSet (sc.eset); Seurat v3 objects can be converted with BisqueRNA::SeuratToExpressionSet(delimiter='-', position=2)
- Flag use.overlap (TRUE when paired bulk/single-cell samples are available, FALSE otherwise)

## Outputs

- Estimated cell-type proportions per bulk sample in res$bulk.props (individuals x cell types)
- Learned per-gene transformation coefficients mapping single-cell reference scale to bulk scale
- Validation statistics, e.g. Pearson correlation (R) and RMSD of estimated vs reference proportions (e.g. R = 0.924, RMSD = 0.029 on DLPFC LOOCV)

## How to apply

Build a cell-type reference profile by averaging per-individual read-count abundances within each cell type (GenerateSCReference) and compute single-cell-derived proportions per individual (CalculateSCCellProportions). Then, for each gene, apply SupervisedTransformBulk by regressing bulk expression on the single-cell-derived cell-type proportions across the overlapping individuals (use.overlap=TRUE), learning gene-specific coefficients that map the sc reference to the bulk scale; when matched bulk/sc samples are unavailable (use.overlap=FALSE), use the semisupervised shrinkage-variance variant SemisupervisedTransformBulk instead. With the transformed bulk data and reference profile, solve a non-negative least-squares decomposition per bulk sample (e.g. via limSolve) with coefficients constrained to be non-negative and sum to one, yielding estimates in res$bulk.props. In practice this is invoked through BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE), after converting inputs to Biobase ExpressionSet objects and applying filter steps (FilterUnexpressedGenes, FilterZeroVarianceGenes, GetOverlappingGenes, CountsToCPM).

## Related tools

- **BisqueRNA** (R package implementing ReferenceBasedDecomposition with the SupervisedTransformBulk/SemisupervisedTransformBulk gene-specific transformations and NNLS decomposition) — github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet container required for both bulk and single-cell inputs)
- **limSolve** (Solves the per-sample non-negative least-squares decomposition enforcing non-negative coefficients summing to one)

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE); props <- res$bulk.props
```

## Evaluation signals

- Correlate estimated proportions (res$bulk.props) with true or single-nucleus-derived reference proportions; on the DLPFC 8-individual LOOCV benchmark Bisque should reach R = 0.924 and RMSD = 0.029
- Check all estimated coefficients/proportions are non-negative and each bulk sample's proportions sum to one
- Cross-check a from-scratch SupervisedTransformBulk + NNLS implementation against BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)
- Confirm the overlapping individuals' IDs appear in both sc.eset$SubjectName and sampleNames(bulk.eset) before learning transformations
- Verify genes were filtered (unexpressed, zero-variance) and CPM-normalized so per-gene regressions operate on comparable scales

## Limitations

- If the number of individuals with single-cell data is small, the reference profile and gene-specific transformations may become unreliable
- A key assumption is that single-cell-based estimates of cell proportions accurately reflect the true proportions to be estimated
- Bisque provides estimates of cell proportions as reported by the single-cell technology used to generate the reference; accuracy may decrease if captured proportions differ significantly from true physiological distributions

## Evidence

- [abstract] gene-specific bulk transformations: "utilizes single-cell RNA-seq (scRNA-seq) or single-nucleus RNA-seq (snRNA-seq) data to generate a reference expression profile and learn gene-specific bulk expression transformations to robustly"
- [intro] differences may introduce gene-specific biases: "these differences may introduce gene-specific biases that break down the correlation between cell-type-specific and bulk tissue measurements"
- [methods] SupervisedTransformBulk signature: "SupervisedTransformBulk(gene, Y.train, X.train, X.pred)"
- [methods] SemisupervisedTransformBulk signature: "SemisupervisedTransformBulk(gene, Y.train, X.pred)"
- [methods] ReferenceBasedDecomposition call: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [methods] ExpressionSet requirement: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [readme] README usage and output slot: [evidence span withheld — anchor status: external_doc]
- [readme] readme reference-based mode: "This method utilizes single-cell data to decompose bulk expression.
We assume that both single-cell and bulk counts are measured from the same tissue."
