---
name: expressionset-construction-from-single-cell-and-bulk-counts
description: Use when raw bulk RNA-seq and single-cell/single-nucleus counts from
  the same tissue need to be packaged as ExpressionSet objects for BisqueRNA decomposition
  with overlapping individuals.
when_to_use_negative:
- No matched individuals are available between the single-cell and bulk datasets and
  the reference-based overlap mode (use.overlap=TRUE) is intended, since overlapping
  samples drive the transformation learning
- Only a set of known marker genes is available and no single-cell reference exists
  — BisqueRNA::MarkerBasedDecomposition is the appropriate mode instead
- The single-cell data cell composition is known to be severely biased relative to
  physiological proportions (e.g., a cell type severely underrepresented), which the
  article warns against
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: Biobase
  role: Provides the ExpressionSet container format required by Bisque for both bulk
    and single-cell data
- name: BisqueRNA
  role: Supplies ReferenceBasedDecomposition, SeuratToExpressionSet, and filter/overlap
    helper functions (FilterUnexpressedGenes, FilterZeroVarianceGenes, GetOverlappingSamples,
    GetOverlappingGenes) used to prepare the ExpressionSets
  repo: github.com/cozygene/bisque
- name: Bisque
  role: Decomposition toolkit whose reference-based mode consumes the constructed
    ExpressionSets
  repo: github.com/cozygene/bisque
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:681c14a058b78e333c413875f4cc7a89ba7fabafcfee10a0aad498878e445581
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# expressionset-construction-from-single-cell-and-bulk-counts

## Summary

Constructs Biobase ExpressionSet objects for paired bulk RNA-seq and snRNA-seq/scRNA-seq count data so that BisqueRNA reference-based decomposition can consume them; single-cell phenoData must carry SubjectName (individual) and cellType columns and sample IDs must align between sc.eset and bulk.eset.

## When to use

Use when raw bulk RNA-seq and single-cell/single-nucleus counts from the same tissue need to be packaged as ExpressionSet objects for BisqueRNA decomposition with overlapping individuals.

## When NOT to use

- No matched individuals are available between the single-cell and bulk datasets and the reference-based overlap mode (use.overlap=TRUE) is intended, since overlapping samples drive the transformation learning
- Only a set of known marker genes is available and no single-cell reference exists — BisqueRNA::MarkerBasedDecomposition is the appropriate mode instead
- The single-cell data cell composition is known to be severely biased relative to physiological proportions (e.g., a cell type severely underrepresented), which the article warns against

## Inputs

- Bulk RNA-seq count/expression matrix
- scRNA-seq or snRNA-seq count matrix with per-cell metadata (SubjectName, cellType)
- Seurat object (alternative input via SeuratToExpressionSet)

## Outputs

- bulk.eset (Biobase ExpressionSet)
- sc.eset (Biobase ExpressionSet with SubjectName/cellType phenoData)
- Filtered ExpressionSets (unexpressed and zero-variance genes removed) and single-cell reference profile inputs

## How to apply

Build the bulk ExpressionSet with Biobase::ExpressionSet(assayData = bulk.matrix) and the single-cell ExpressionSet with assayData=sc.counts.matrix plus a phenoData containing SubjectName and cellType columns; alternatively convert a Seurat object with BisqueRNA::SeuratToExpressionSet(seurat.obj, delimiter="-", position=2, version="v3"). Ensure the individual identifiers appear both in sc.eset$SubjectName and in sampleNames(bulk.eset) so GetOverlappingSamples can find the paired adipose individuals. Then apply BisqueRNA's filter helpers — FilterUnexpressedGenes, FilterZeroVarianceGenes, CountsToCPM, GetOverlappingGenes, GetOverlappingSamples — before calling GenerateSCReference and CalculateSCCellProportions to build the reference profile. The rationale is that Bisque requires ExpressionSet format and uses the overlapping individuals and genes to learn gene-specific bulk expression transformations that correct single-cell vs bulk technology biases.

## Related tools

- **Biobase** (Provides the ExpressionSet container format required by Bisque for both bulk and single-cell data)
- **BisqueRNA** (Supplies ReferenceBasedDecomposition, SeuratToExpressionSet, and filter/overlap helper functions (FilterUnexpressedGenes, FilterZeroVarianceGenes, GetOverlappingSamples, GetOverlappingGenes) used to prepare the ExpressionSets) — github.com/cozygene/bisque
- **Bisque** (Decomposition toolkit whose reference-based mode consumes the constructed ExpressionSets) — github.com/cozygene/bisque

## Examples

```
library(Biobase); bulk.eset <- Biobase::ExpressionSet(assayData=bulk.matrix); res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)
```

## Evaluation signals

- sc.eset$SubjectName and sampleNames(bulk.eset) contain the same individual identifiers, so GetOverlappingSamples returns the expected paired individuals (e.g., the 6 adipose individuals)
- phenoData of sc.eset exposes both SubjectName and cellType columns without missing values
- FilterUnexpressedGenes and FilterZeroVarianceGenes run without errors and remove only genes absent or invariant across samples
- CalculateSCCellProportions and GenerateSCReference succeed on the filtered sc.eset and produce per-individual proportions and a reference profile covering the expected cell types
- ReferenceBasedDecomposition with markers=NULL and use.overlap=TRUE returns res$bulk.props with one proportion per held-out individual and cell type

## Limitations

- If the number of individuals with single-cell data is small, the reference profile and gene-specific transformations may become unreliable
- A key assumption is that single-cell-based estimates of cell proportions accurately reflect the true proportions; accuracy may decrease if single-cell-captured proportions differ significantly from physiological distributions
- Marker-based mode (if used from the same ExpressionSets) estimates relative abundances that cannot be compared across cell types

## Evidence

- [methods] Bisque requires expression data in the ExpressionSet format from the Biobase package.: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [methods] sc.eset construction with phenoData: "sc.eset <- Biobase::ExpressionSet(assayData=sc.counts.matrix,
                                  phenoData=sc.pdata)"
- [methods] reference-based decomposition call: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [readme] README on reference-based assumptions and matched samples: "While we don't explicitly require matched samples, we expect having samples with both single-cell and bulk expression measured will provide more accurate results."
- [readme] README on result access: [evidence span withheld — anchor status: external_doc]
