---
name: adipose-snrna-seq-expressionset-conversion
description: Use when starting a Bisque decomposition or bias-simulation task with
  adipose snRNA-seq counts plus bulk RNA-seq samples, and the inputs must be packaged
  as ExpressionSet objects with sample-to-individual and cell-type labels.
when_to_use_negative:
- Data are plain count matrices without individual and cell-type labels — overlap-based
  transformation learning (use.overlap=TRUE) cannot be enabled, so use marker-based
  decomposition (BisqueMarker) instead
- The downstream method does not accept ExpressionSet (e.g., CIBERSORT/BSEQ-sc or
  MuSiC comparators consume matrices directly), so this conversion is only needed
  for the BisqueRNA path
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: Bisque
  role: Toolkit providing ExpressionSet conversion utilities (SeuratToExpressionSet)
    and the decomposition entry points that consume the converted objects
  repo: github.com/cozygene/bisque
- name: BisqueRNA
  role: R package whose ReferenceBasedDecomposition consumes the sc.eset and bulk.eset;
    SeuratToExpressionSet converts Seurat objects
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Supplies the ExpressionSet container; ExpressionSet(assayData=..., phenoData=...)
    constructs both objects
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:8c2c50423d53d6cddc821a051becfdbc2109878b25bbd4e3d05099750fb5aab4
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# adipose-snRNA-seq-expressionset-conversion

## Summary

Prepare adipose snRNA-seq counts with cell-type and individual labels, and the matched bulk expression matrix, as Biobase ExpressionSet objects (sc.eset and bulk.eset) so BisqueRNA's ReferenceBasedDecomposition can run with use.overlap=TRUE on the overlapping samples.

## When to use

Use when starting a Bisque decomposition or bias-simulation task with adipose snRNA-seq counts plus bulk RNA-seq samples, and the inputs must be packaged as ExpressionSet objects with sample-to-individual and cell-type labels.

## When NOT to use

- Data are plain count matrices without individual and cell-type labels — overlap-based transformation learning (use.overlap=TRUE) cannot be enabled, so use marker-based decomposition (BisqueMarker) instead
- The downstream method does not accept ExpressionSet (e.g., CIBERSORT/BSEQ-sc or MuSiC comparators consume matrices directly), so this conversion is only needed for the BisqueRNA path

## Inputs

- adipose snRNA-seq count matrix (genes x cells) with per-cell cell-type labels
- individual/sample name per cell, enabling overlap matching to bulk samples
- bulk RNA-seq expression matrix for the same adipose individuals
- optional: Seurat object containing the adipose snRNA-seq data

## Outputs

- sc.eset: Biobase ExpressionSet of single-nucleus counts with cell-type and individual phenoData
- bulk.eset: Biobase ExpressionSet of bulk expression
- readiness checks: overlapping gene set and overlapping individual set shared by both ExpressionSets

## How to apply

Load the adipose snRNA-seq count matrix and attach phenoData encoding cell-type labels and individual (sample) names using the delimiter convention required by BisqueRNA (e.g., sample-celltype). Build the phenoData as an AnnotatedDataFrame and construct the single-nucleus ExpressionSet via Biobase::ExpressionSet(assayData=sc.counts.matrix, phenoData=sc.pdata); if the data live in a Seurat object, use BisqueRNA::SeuratToExpressionSet(seurat.obj, delimiter="-", position=2, version="v3"). In parallel, convert the bulk expression matrix via Biobase::ExpressionSet(assayData=bulk.matrix). The individual labels are essential because use.overlap=TRUE lets Bisque learn gene-specific transformations from samples with both bulk and single-nucleus measurements; cell-type labels drive GenerateSCReference and CalculateSCCellProportions. Verify that both ExpressionSet objects share overlapping genes (GetOverlappingGenes) and overlapping individuals (GetOverlappingSamples) before decomposition.

## Related tools

- **Bisque** (Toolkit providing ExpressionSet conversion utilities (SeuratToExpressionSet) and the decomposition entry points that consume the converted objects) — github.com/cozygene/bisque
- **BisqueRNA** (R package whose ReferenceBasedDecomposition consumes the sc.eset and bulk.eset; SeuratToExpressionSet converts Seurat objects) — github.com/cozygene/bisque
- **Biobase** (Supplies the ExpressionSet container; ExpressionSet(assayData=..., phenoData=...) constructs both objects)

## Examples

```
library(Biobase); bulk.eset <- Biobase::ExpressionSet(assayData=bulk.matrix); sc.eset <- Biobase::ExpressionSet(assayData=sc.counts.matrix, phenoData=sc.pdata)
```

## Evaluation signals

- Both objects are valid ExpressionSet instances whose assayData dimensions match the source count matrices (genes x cells and genes x samples)
- sc.eset phenoData contains individual names and cell-type labels parseable with the expected delimiter (position=2 when splitting on '-')
- GetOverlappingSamples(sc.eset, bulk.eset, subject.names) returns the expected set of adipose individuals measured in both modalities, and GetOverlappingGenes returns a non-empty gene intersection
- ReferenceBasedDecomposition with use.overlap=TRUE runs without label errors and returns res$bulk.props whose columns are individuals and rows are cell types
- Estimated proportions for the unbiased (sigma=0) case correlate positively with the true adipose cell-type proportions before any bias perturbation is applied

## Limitations

- If the number of individuals with single-cell data is small, the reference profile and gene-specific transformations may become unreliable
- The adipose snRNA-seq reference must reflect expected physiological cell composition; if a cell type is severely underrepresented by single-nucleus capture, Bisque accuracy may decrease
- Label delimiter/position mismatches (e.g., wrong delimiter or position in SeuratToExpressionSet) will silently corrupt individual-to-bulk sample mapping used by use.overlap=TRUE

## Evidence

- [methods] Convert bulk expression matrix to ExpressionSet: "bulk.eset <- Biobase::ExpressionSet(assayData = bulk.matrix)"
- [methods] Convert scRNA-seq data with cell type and individual labels to ExpressionSet: "sc.eset <- Biobase::ExpressionSet(assayData=sc.counts.matrix,
                                  phenoData=sc.pdata)"
- [methods] Convert Seurat object to ExpressionSet: "sc.eset <- BisqueRNA::SeuratToExpressionSet(seurat.obj, delimiter="-", position=2, version="v3")"
- [methods] Reference-based decomposition call requiring ExpressionSet inputs: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [readme] README statement that single-cell and bulk data should come from the same tissue with matching composition: "We assume that both single-cell and bulk counts are measured from the same tissue. Specifically, the cell composition of the labeled single-cell data should match the expected physiological"
