---
name: bulk-rnaseq-cpm-normalization
description: Use when raw bulk RNA-seq count matrices (e.g., DLPFC paired bulk samples)
  must be converted to Biobase ExpressionSet objects, filtered, and CPM-normalized
  prior to BisqueRNA::ReferenceBasedDecomposition or benchmarking against CIBERSORT/MuSiC/BSEQ-sc.
when_to_use_negative:
- Expression data are already normalized (e.g., TPM/FPKM or previously CPM-scaled),
  so applying CountsToCPM would double-normalize and distort proportion estimates
- There is no paired single-cell/single-nucleus reference to align with (no cellType/SubjectName
  labels), since this normalization step exists to prepare the bulk eset for overlap-based
  reference decomposition
- Too few individuals have single-cell data, in which case the reference profile and
  gene-specific transformations may become unreliable regardless of normalization
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: BisqueRNA
  role: Provides the CountsToCPM, FilterUnexpressedGenes, FilterZeroVarianceGenes,
    and GetOverlappingGenes functions used to normalize and filter the bulk ExpressionSet
    before decomposition
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Supplies the ExpressionSet container that Bisque requires for both bulk and
    single-cell count matrices
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:4f9eddb92613b820eefeee833b950cd3413b474e8c0a6818ddbd1f7b2ed1cba9
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bulk-rnaseq-cpm-normalization

## Summary

Normalize paired bulk RNA-seq count data to counts-per-million (CPM) and filter unexpressed and zero-variance genes with BisqueRNA utilities so the bulk ExpressionSet is comparable across individuals before reference-based cell-type decomposition.

## When to use

Use when raw bulk RNA-seq count matrices (e.g., DLPFC paired bulk samples) must be converted to Biobase ExpressionSet objects, filtered, and CPM-normalized prior to BisqueRNA::ReferenceBasedDecomposition or benchmarking against CIBERSORT/MuSiC/BSEQ-sc.

## When NOT to use

- Expression data are already normalized (e.g., TPM/FPKM or previously CPM-scaled), so applying CountsToCPM would double-normalize and distort proportion estimates
- There is no paired single-cell/single-nucleus reference to align with (no cellType/SubjectName labels), since this normalization step exists to prepare the bulk eset for overlap-based reference decomposition
- Too few individuals have single-cell data, in which case the reference profile and gene-specific transformations may become unreliable regardless of normalization

## Inputs

- Raw bulk RNA-seq count matrix for paired individuals (e.g., DLPFC bulk samples) as a Biobase ExpressionSet
- Single-nucleus (snRNA-seq) count matrix with cellType and SubjectName labels as a Biobase ExpressionSet (or Seurat v3 object convertible via BisqueRNA::SeuratToExpressionSet)

## Outputs

- CPM-normalized, gene-filtered bulk ExpressionSet restricted to genes overlapping the single-cell reference
- Normalized ExpressionSet ready as input to BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)

## How to apply

Load the bulk counts as a Biobase ExpressionSet (Bisque requires ExpressionSet format), then run BisqueRNA::FilterUnexpressedGenes to drop unexpressed genes and FilterZeroVarianceGenes to drop genes with no variance across samples. Restrict to genes shared between the single-cell (sc.eset) and bulk (bulk.eset) data using GetOverlappingGenes, since decomposition can only be performed on overlapping features. Apply BisqueRNA::CountsToCPM to scale each bulk sample by library size in counts per million, as done in the DLPFC LOOCV workflow (25% nuclei subsample per individual) before ReferenceBasedDecomposition with use.overlap=TRUE. Finally, confirm the 8 paired individuals' IDs appear in both sc.eset$SubjectName and sampleNames(bulk.eset) so SupervisedTransformBulk can learn gene-specific bulk expression transformations on the normalized data.

## Related tools

- **BisqueRNA** (Provides the CountsToCPM, FilterUnexpressedGenes, FilterZeroVarianceGenes, and GetOverlappingGenes functions used to normalize and filter the bulk ExpressionSet before decomposition) — github.com/cozygene/bisque
- **Biobase** (Supplies the ExpressionSet container that Bisque requires for both bulk and single-cell count matrices)

## Examples

```
bulk.eset <- Biobase::ExpressionSet(assayData = bulk.matrix); bulk.eset <- BisqueRNA::FilterUnexpressedGenes(bulk.eset); bulk.eset <- BisqueRNA::CountsToCPM(bulk.eset)
```

## Evaluation signals

- After FilterUnexpressedGenes and FilterZeroVarianceGenes, no genes remain with zero counts across all bulk samples or zero variance across individuals
- The normalized bulk eset contains only genes returned by GetOverlappingGenes(sc.eset, bulk.eset, markers) — genes absent from the single-cell reference are removed
- CPM scaling yields per-sample totals of 1e6 counts (or a constant scale), making bulk samples comparable across the 8 paired individuals
- Subject identifiers in sc.eset$SubjectName match sampleNames(bulk.eset) after normalization, allowing use.overlap=TRUE decomposition
- Downstream ReferenceBasedDecomposition runs without error and Bisque attains the reported DLPFC LOOCV accuracy (R = 0.924, RMSD = 0.029)

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable
- Bisque provides estimates of cell proportions reported by the single-cell technology used to generate the reference data; accuracy may decrease if single-cell-captured proportions differ significantly from true physiological distributions
- Cell composition of the labeled single-cell data should match the expected physiological composition (e.g., the 25% nuclei subsample constraint in the DLPFC benchmark)

## Evidence

- [methods] FilterUnexpressedGenes: "- `FilterUnexpressedGenes(eset, verbose)`"
- [methods] FilterZeroVarianceGenes: "- `FilterZeroVarianceGenes(eset, verbose)`"
- [methods] GetOverlappingGenes: "- `GetOverlappingGenes(sc.eset, bulk.eset, markers, verbose)`"
- [methods] CountsToCPM: "- `CountsToCPM(eset)`"
- [methods] ExpressionSet requirement: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [readme] Bisque purpose: "An R toolkit for accurate and efficient estimation of cell composition ('decomposition') from bulk expression data with single-cell information."
