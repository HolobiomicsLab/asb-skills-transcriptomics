---
name: bulk-rnaseq-expressionset-conversion
description: Use when running BisqueRNA::ReferenceBasedDecomposition or MarkerBasedDecomposition
  on bulk RNA-seq counts with a single-cell reference, and the inputs are still raw
  matrices, Seurat objects, or data frames that must first be cast into Biobase ExpressionSet
  format.
when_to_use_negative:
- Data is already in ExpressionSet format — re-conversion is unnecessary; proceed
  directly to BisqueRNA::ReferenceBasedDecomposition
- You only intend to run marker-based decomposition and never need subject-level overlap
  matching — a minimal bulk.eset conversion suffices, but a mislabeled sc.eset with
  wrong SubjectName parsing would silently break use.overlap
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: Biobase
  role: Provides the ExpressionSet container and ExpressionSet() constructor that
    Bisque requires for both bulk and single-cell inputs
- name: BisqueRNA
  role: Supplies SeuratToExpressionSet() for converting Seurat v3 objects to ExpressionSet,
    and consumes the converted objects in ReferenceBasedDecomposition/MarkerBasedDecomposition
  repo: github.com/cozygene/bisque
- name: Bisque
  role: The decomposition toolkit whose functions require ExpressionSet-formatted
    inputs
  repo: github.com/cozygene/bisque
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:1d25b46867903c63b48c2cd37e20887f8b592f7b180ccdcb5a7ce8e3a4ec358f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - Bisque
  - BisqueMarker
  - BisqueRNA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bulk-rnaseq-expressionset-conversion

## Summary

Converts bulk RNA-seq count matrices and scRNA-seq/snRNA-seq reference data (with cell type and individual labels) into Biobase ExpressionSet objects so that BisqueRNA decomposition functions can consume them. Bisque requires expression data in ExpressionSet format, with SubjectName and cellType phenoData columns on the single-cell object.

## When to use

Use when running BisqueRNA::ReferenceBasedDecomposition or MarkerBasedDecomposition on bulk RNA-seq counts with a single-cell reference, and the inputs are still raw matrices, Seurat objects, or data frames that must first be cast into Biobase ExpressionSet format.

## When NOT to use

- Data is already in ExpressionSet format — re-conversion is unnecessary; proceed directly to BisqueRNA::ReferenceBasedDecomposition
- You only intend to run marker-based decomposition and never need subject-level overlap matching — a minimal bulk.eset conversion suffices, but a mislabeled sc.eset with wrong SubjectName parsing would silently break use.overlap

## Inputs

- Bulk RNA-seq count matrix (genes x samples) for e.g. 628 DLPFC bulk samples
- scRNA-seq/snRNA-seq counts matrix with cell type and individual (SubjectName) labels
- Seurat object (Seurat v3) holding the single-cell reference
- phenoData metadata with SubjectName and cellType columns

## Outputs

- Biobase ExpressionSet for bulk expression (bulk.eset)
- Biobase ExpressionSet for single-cell reference (sc.eset) with SubjectName and cellType phenoData

## How to apply

Build the bulk ExpressionSet directly from the count matrix via Biobase::ExpressionSet(assayData = bulk.matrix). For the single-cell reference, assemble a phenoData AnnotatedDataFrame carrying the cell type labels and subject identifiers (SubjectName), then call Biobase::ExpressionSet(assayData = sc.counts.matrix, phenoData = sc.pdata); the SubjectName column is what lets Bisque's use.overlap=TRUE option match individuals measured in both bulk and single-cell data. If the reference lives in a Seurat object, use BisqueRNA::SeuratToExpressionSet(seurat.obj, delimiter="-", position=2, version="v3"), which parses subject IDs from cell names using the delimiter and field position. In the DLPFC application, the bulk counts for the 628 samples and the snRNA-seq reference from the subset of individuals with single-nucleus data were both converted this way before decomposition.

## Related tools

- **Biobase** (Provides the ExpressionSet container and ExpressionSet() constructor that Bisque requires for both bulk and single-cell inputs)
- **BisqueRNA** (Supplies SeuratToExpressionSet() for converting Seurat v3 objects to ExpressionSet, and consumes the converted objects in ReferenceBasedDecomposition/MarkerBasedDecomposition) — github.com/cozygene/bisque
- **Bisque** (The decomposition toolkit whose functions require ExpressionSet-formatted inputs) — github.com/cozygene/bisque

## Examples

```
bulk.eset <- Biobase::ExpressionSet(assayData = bulk.matrix); sc.eset <- BisqueRNA::SeuratToExpressionSet(seurat.obj, delimiter="-", position=2, version="v3")
```

## Evaluation signals

- Both objects are valid ExpressionSet instances: assayData contains the full gene x sample count matrix and phenoData is attached for the sc.eset
- sc.eset phenoData includes SubjectName and cellType columns so Bisque can compute per-individual cell proportions and overlap matching
- SeuratToExpressionSet delimiter and position arguments correctly recover subject IDs (e.g. delimiter='-', position=2, version='v3')
- Genes in bulk.eset and sc.eset are on a comparable identifier system so downstream GetOverlappingGenes can intersect them
- Downstream ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE) runs without 'subject not found' errors and returns a res$bulk.props matrix for all 628 bulk samples

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable
- The cell composition of the labeled single-cell data should match the expected physiological composition; SeuratToExpressionSet requires subject IDs to be parseable from cell names with the given delimiter and position
- Bisque provides estimates of cell proportions reported by the single-cell technology used to generate the reference data (e.g. snRNA-seq biases such as nuclear pre-mRNA)

## Evidence

- [methods] Bisque requires expression data in the ExpressionSet format from the Biobase package.: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [methods] Convert scRNA-seq data with cell type and individual labels to ExpressionSet: "sc.eset <- Biobase::ExpressionSet(assayData=sc.counts.matrix,
                                  phenoData=sc.pdata)"
- [methods] Convert Seurat object to ExpressionSet: "sc.eset <- BisqueRNA::SeuratToExpressionSet(seurat.obj, delimiter="-", position=2, version="v3")"
- [readme] Reference-based decomposition requires matched single-cell and bulk inputs: "Specifically, the cell composition of the labeled single-cell data should match the expected physiological composition."
- [discussion] If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable.: "if the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable"
