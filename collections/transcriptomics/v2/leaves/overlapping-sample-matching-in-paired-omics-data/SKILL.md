---
name: overlapping-sample-matching-in-paired-omics-data
description: Use when a cohort contains individuals with both bulk RNA-seq and scRNA-seq/snRNA-seq
  data and decomposition requires identifying which single-cell samples correspond
  to which bulk samples.
when_to_use_negative:
- No individuals have both bulk and single-cell measurements from the same tissue
  — overlapping-sample matching cannot anchor transformations; consider the marker-based
  (BisqueMarker) mode instead
- The single-cell data are from a different tissue than the bulk data, violating the
  Bisque assumption that both are measured from the same tissue
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3518
tools:
- name: BisqueRNA
  role: Provides GetOverlappingSamples, GetOverlappingGenes, FilterUnexpressedGenes,
    FilterZeroVarianceGenes, and ReferenceBasedDecomposition (use.overlap=TRUE) for
    matching paired samples and decomposing bulk expression
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet container format required for both bulk and single-cell
    inputs so sample IDs can be matched across objects
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:74816b6b72541ee29dfb602fa3f5db7ad8a20aae28514b2b380908a2100d6576
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# overlapping-sample-matching-in-paired-omics-data

## Summary

Matches individuals that have both bulk RNA-seq and single-cell/single-nucleus RNA-seq measurements so that paired samples can be linked for reference-based decomposition (e.g., in BisqueRNA's use.overlap=TRUE mode).

## When to use

Use when a cohort contains individuals with both bulk RNA-seq and scRNA-seq/snRNA-seq data and decomposition requires identifying which single-cell samples correspond to which bulk samples.

## When NOT to use

- No individuals have both bulk and single-cell measurements from the same tissue — overlapping-sample matching cannot anchor transformations; consider the marker-based (BisqueMarker) mode instead
- The single-cell data are from a different tissue than the bulk data, violating the Bisque assumption that both are measured from the same tissue

## Inputs

- Bulk RNA-seq expression matrix as a Biobase ExpressionSet (bulk.eset) with individual IDs in sampleNames
- Single-cell/single-nucleus counts matrix as a Biobase ExpressionSet (sc.eset) with phenoData containing SubjectName and cellType columns
- Vector of subject/individual names (subject.names) shared between the bulk and single-cell datasets

## Outputs

- Set of overlapping individuals present in both bulk and single-cell ExpressionSets
- Filtered, gene-matched bulk and sc ExpressionSets restricted to overlapping genes/samples
- Decomposition result (res$bulk.props) from ReferenceBasedDecomposition run with use.overlap=TRUE

## How to apply

Format both bulk and single-cell data as Biobase ExpressionSet objects, ensuring the single-cell phenoData carries a SubjectName (individual) column and that the individual IDs appear in both sc.eset$SubjectName and sampleNames(bulk.eset). Use BisqueRNA::GetOverlappingSamples with the sc.eset, bulk.eset, and subject.names arguments to identify the individuals measured in both modalities, and GetOverlappingGenes to restrict both ExpressionSets to the shared gene set. Apply FilterUnexpressedGenes and FilterZeroVarianceGenes to clean the expression sets before matching. Pass use.overlap=TRUE to BisqueRNA::ReferenceBasedDecomposition so the matched individuals are used to learn gene-specific bulk expression transformations; without overlap identification, the transformation step cannot be anchored to paired measurements.

## Related tools

- **BisqueRNA** (Provides GetOverlappingSamples, GetOverlappingGenes, FilterUnexpressedGenes, FilterZeroVarianceGenes, and ReferenceBasedDecomposition (use.overlap=TRUE) for matching paired samples and decomposing bulk expression) — github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet container format required for both bulk and single-cell inputs so sample IDs can be matched across objects)

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)
```

## Evaluation signals

- sc.eset$SubjectName values and sampleNames(bulk.eset) intersect to yield the expected set of paired individuals (e.g., the 6 adipose individuals with both bulk and snRNA-seq)
- GetOverlappingGenes returns identical gene sets for the bulk and sc ExpressionSets after filtering
- ReferenceBasedDecomposition reports use.overlap=TRUE and returns bulk.props for all expected cell types and individuals
- Leave-one-out decomposition accuracy on matched pairs reaches the reported benchmark (Pearson R = 0.923, RMSD = 0.074 for Bisque on the adipose LOOCV)

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable
- The framework assumes single-cell-based estimates of cell proportions accurately reflect the true proportions; accuracy may decrease if the proportions captured by single-cell experiments differ significantly from true physiological distributions
- The cell composition of the labeled single-cell data should match the expected physiological composition of the tissue

## Evidence

- [methods] GetOverlappingSamples identifies the samples present in both datasets: "- `GetOverlappingSamples(sc.eset, bulk.eset, subject.names, verbose)`"
- [methods] Bisque requires ExpressionSet format and runs reference-based decomposition with overlap: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [readme] Reference-based mode assumes single-cell and bulk counts come from the same tissue and matched samples improve results: "We assume that both single-cell and bulk counts are measured from the same tissue. Specifically, the cell composition of the labeled single-cell data should match the expected physiological"
- [discussion] Limitation when few individuals have single-cell data: "First, if the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable."
- [readme] Results are accessed via res$bulk.props: [evidence span withheld — anchor status: external_doc]
