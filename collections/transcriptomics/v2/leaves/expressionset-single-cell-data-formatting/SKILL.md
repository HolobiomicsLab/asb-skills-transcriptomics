---
name: expressionset-single-cell-data-formatting
description: Use when preparing scRNA-seq/snRNA-seq counts with cell-type and subject
  labels plus paired bulk RNA-seq counts for BisqueRNA decomposition, e.g. a Seurat
  v3 object or count matrices needing ExpressionSet conversion.
when_to_use_negative:
- Input data are already Biobase ExpressionSet objects with correct cellType/SubjectName
  phenoData and matching sample names — re-conversion is unnecessary
- No matched single-cell and bulk samples exist and you plan marker-based decomposition
  only, which requires marker genes rather than sample-matched ExpressionSets
- The single-cell object is not Seurat v3 (SeuratToExpressionSet with version="v3"
  would not apply; use the count-matrix + phenoData route instead)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: Biobase
  role: Provides the ExpressionSet container required by Bisque for both bulk and
    single-cell expression data
  repo: https://github.com/cozygene/bisque
- name: BisqueRNA
  role: Supplies SeuratToExpressionSet for converting a Seurat v3 object into an ExpressionSet
    with subject labels parsed via delimiter and position
  repo: https://github.com/cozygene/bisque
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:5f0fa1429504ecf51e7d0d372def7cde5e098ca460cf0dac24e26c28ca147ad5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cozygene/bisque
  tools:
  - BSEQ-sc
  - Biobase
  - Bisque
  - BisqueRNA
  - CIBERSORT
  - MuSiC
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# expressionset-single-cell-data-formatting

## Summary

Formats single-nucleus/single-cell and paired bulk RNA-seq count matrices into Biobase ExpressionSet objects, attaching cellType and SubjectName labels so BisqueRNA can match individuals across the single-cell reference and bulk samples during decomposition.

## When to use

Use when preparing scRNA-seq/snRNA-seq counts with cell-type and subject labels plus paired bulk RNA-seq counts for BisqueRNA decomposition, e.g. a Seurat v3 object or count matrices needing ExpressionSet conversion.

## When NOT to use

- Input data are already Biobase ExpressionSet objects with correct cellType/SubjectName phenoData and matching sample names — re-conversion is unnecessary
- No matched single-cell and bulk samples exist and you plan marker-based decomposition only, which requires marker genes rather than sample-matched ExpressionSets
- The single-cell object is not Seurat v3 (SeuratToExpressionSet with version="v3" would not apply; use the count-matrix + phenoData route instead)

## Inputs

- snRNA-seq or scRNA-seq count matrix with cellType and SubjectName labels per cell (or a Seurat v3 object)
- Paired bulk RNA-seq count matrix with sample names matching the single-cell subject IDs
- Delimiter and position specifying how subject IDs are encoded in single-cell cell names (e.g. delimiter '-', position 2)

## Outputs

- sc.eset: Biobase ExpressionSet of single-cell counts with phenoData containing cellType and SubjectName
- bulk.eset: Biobase ExpressionSet of bulk counts
- Filtered/normalized ExpressionSets (unexpressed and zero-variance genes removed, CPM-normalized) with verified subject-ID overlap

## How to apply

Convert the single-cell counts into an ExpressionSet whose phenoData carries cellType and SubjectName columns; if starting from Seurat v3, use BisqueRNA::SeuratToExpressionSet with delimiter '-' and position 2 so subject IDs are parsed from cell barcodes (e.g. '101-2' yields subject 101). Convert the paired bulk counts with Biobase::ExpressionSet(assayData = bulk.matrix). Apply the documented filtering utilities — FilterUnexpressedGenes, FilterZeroVarianceGenes, CountsToCPM — and confirm each individual's ID appears in both sc.eset$SubjectName and sampleNames(bulk.eset), since BisqueRNA matches overlapping samples by these names (GetOverlappingSamples) when use.overlap=TRUE to learn gene-specific bulk transformations. Well-formed ExpressionSets are the required input format for both ReferenceBasedDecomposition and MarkerBasedDecomposition.

## Related tools

- **Biobase** (Provides the ExpressionSet container required by Bisque for both bulk and single-cell expression data) — https://github.com/cozygene/bisque
- **BisqueRNA** (Supplies SeuratToExpressionSet for converting a Seurat v3 object into an ExpressionSet with subject labels parsed via delimiter and position) — https://github.com/cozygene/bisque

## Examples

```
sc.eset <- BisqueRNA::SeuratToExpressionSet(seurat.obj, delimiter="-", position=2, version="v3"); bulk.eset <- Biobase::ExpressionSet(assayData = bulk.matrix)
```

## Evaluation signals

- sc.eset phenoData exposes non-empty cellType and SubjectName columns, and subject IDs parse correctly (e.g. 'sampleID-cellType' splitting at position 2)
- All 8 paired individuals' IDs appear in both sc.eset$SubjectName and sampleNames(bulk.eset)
- Post-filter ExpressionSets contain no unexpressed or zero-variance genes and are CPM-normalized
- BisqueRNA::ReferenceBasedDecomposition runs without 'overlapping samples' errors and returns estimates in res$bulk.props

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable
- A key assumption of the framework is that single-cell-based estimates of cell proportions accurately reflect the true proportions; the single-cell composition should match the expected physiological composition of the tissue

## Evidence

- [methods] Bisque requires expression data in the ExpressionSet format from the Biobase package.: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [methods] Convert Seurat object to ExpressionSet: "sc.eset <- BisqueRNA::SeuratToExpressionSet(seurat.obj, delimiter="-", position=2, version="v3")"
- [methods] Bulk ExpressionSet conversion: "bulk.eset <- Biobase::ExpressionSet(assayData = bulk.matrix)"
- [methods] Decomposition call depends on formatted eset objects: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [readme] README assumption about matched single-cell and bulk composition: "the cell composition of the labeled single-cell data should match the expected physiological composition"
