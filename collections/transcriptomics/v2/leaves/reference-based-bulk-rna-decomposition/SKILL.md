---
name: reference-based-bulk-rna-decomposition
description: Use when you have bulk RNA-seq samples from a tissue plus paired (or
  at least same-tissue) single-cell or single-nucleus RNA-seq data with SubjectName
  and cellType labels, and need cell-type proportion estimates for downstream phenotype
  associations or benchmarking.
when_to_use_negative:
- No single-cell reference is available and only marker genes exist — use BisqueRNA::MarkerBasedDecomposition
  instead, noting its abundances are relative and cannot be compared across cell types
- Very few individuals have single-cell data, making the reference profile and gene-specific
  transformations unreliable
- The single-cell data's cell composition is known to deviate strongly from physiological
  proportions (e.g., severe underrepresentation of a cell type of interest)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: BisqueRNA
  role: R package implementing ReferenceBasedDecomposition and MarkerBasedDecomposition;
    proportions returned in the bulk.props slot
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet format required for bulk.eset and sc.eset inputs
- name: Bisque
  role: Reference implementation of gene-specific bulk expression transformations
    for decomposition benchmarking (e.g., SemisupervisedTransformBulk variant)
  repo: github.com/cozygene/bisque
- name: CIBERSORTx
  role: Baseline method for comparison in LOOCV decomposition accuracy benchmarks
- name: MuSiC
  role: Baseline weighted NNLS regression method for comparison in decomposition benchmarks
- name: BSEQ-sc
  role: Baseline method generating a single-cell reference for the CIBERSORT model,
    used for comparison
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:653641ede31087e85d2bf43c28ca99196df72e8e9f7f67bd60acfbb7b9208ffb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSEQ-sc
  - Biobase
  - Bisque
  - BisqueMarker
  - BisqueRNA
  - CIBERSORT
  - CIBERSORTx
  - MuSiC
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# reference-based-bulk-rna-decomposition

## Summary

Estimates cell-type proportions in bulk RNA-seq by integrating scRNA-seq/snRNA-seq reference profiles with gene-specific bulk expression transformations using Bisque's ReferenceBasedDecomposition, correcting for technical biases between single-cell and bulk chemistries.

## When to use

Use when you have bulk RNA-seq samples from a tissue plus paired (or at least same-tissue) single-cell or single-nucleus RNA-seq data with SubjectName and cellType labels, and need cell-type proportion estimates for downstream phenotype associations or benchmarking.

## When NOT to use

- No single-cell reference is available and only marker genes exist — use BisqueRNA::MarkerBasedDecomposition instead, noting its abundances are relative and cannot be compared across cell types
- Very few individuals have single-cell data, making the reference profile and gene-specific transformations unreliable
- The single-cell data's cell composition is known to deviate strongly from physiological proportions (e.g., severe underrepresentation of a cell type of interest)

## Inputs

- Bulk RNA-seq expression matrix (counts or expression) convertible to a Biobase ExpressionSet
- scRNA-seq/snRNA-seq counts matrix with per-cell SubjectName (individual) and cellType metadata
- Optional: set of cell-type marker genes for BisqueRNA::MarkerBasedDecomposition when no reference profile exists
- Optional: Seurat object convertible via BisqueRNA::SeuratToExpressionSet

## Outputs

- res$bulk.props: per-sample, per-cell-type estimated proportions (or relative abundances for marker-based mode)
- Single-cell-derived true proportions from CalculateSCCellProportions for validation
- Accuracy metrics such as Pearson correlation (cor) and RMSD against true proportions
- Per-cell-type proportion estimates usable in linear-mixed models against phenotypes (BMI, Matsuda index) with age, sex, kinship covariates

## How to apply

Format the bulk expression matrix and single-cell counts (with phenoData containing SubjectName and cellType columns) as Biobase ExpressionSet objects so sample IDs appear in both sc.eset$SubjectName and sampleNames(bulk.eset). Identify paired samples and shared genes with GetOverlappingSamples/GetOverlappingGenes and apply FilterUnexpressedGenes and FilterZeroVarianceGenes to the expression sets. Run BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE), optionally preceded by CalculateSCCellProportions and GenerateSCReference, and extract estimated proportions from the res$bulk.props slot. Bisque learns gene-specific bulk expression transformations (via SupervisedTransformBulk/SemisupervisedTransformBulk) that correct for single-cell vs bulk technology biases, which is why it outperforms CIBERSORTx, MuSiC, BSEQ-sc, and CIBERSORT under technical variation. Accuracy is judged by correlating estimates against single-cell-derived true proportions (e.g., global Pearson R = 0.923, RMSD = 0.074 in adipose LOOCV) or by replication of known phenotype associations (e.g., adipocyte proportion vs BMI, p = 0.030).

## Related tools

- **BisqueRNA** (R package implementing ReferenceBasedDecomposition and MarkerBasedDecomposition; proportions returned in the bulk.props slot) — github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet format required for bulk.eset and sc.eset inputs)
- **Bisque** (Reference implementation of gene-specific bulk expression transformations for decomposition benchmarking (e.g., SemisupervisedTransformBulk variant)) — github.com/cozygene/bisque
- **CIBERSORTx** (Baseline method for comparison in LOOCV decomposition accuracy benchmarks)
- **MuSiC** (Baseline weighted NNLS regression method for comparison in decomposition benchmarks)
- **BSEQ-sc** (Baseline method generating a single-cell reference for the CIBERSORT model, used for comparison)

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE); props <- res$bulk.props
```

## Evaluation signals

- Leave-one-out cross-validation on the 6 paired adipose individuals reproduces Bisque global accuracy of Pearson R = 0.923 and RMSD = 0.074 (Table 2)
- Estimated proportions correlate strongly with single-cell-derived true proportions via cor(as.vector(ref.based.estimates), as.vector(true.props))
- Decomposed adipocyte proportions show the expected significant negative association with BMI (p = 0.030) and macrophage-BMI p < 0.001, T-cell-Matsuda p = 0.002 in linear-mixed models
- Schema check: bulk.props rows are samples and columns are cell types, and SubjectName values in sc.eset overlap sampleNames(bulk.eset) for use.overlap=TRUE
- Preprocessing invariants: FilterUnexpressedGenes and FilterZeroVarianceGenes leave only genes shared between bulk and single-cell ExpressionSets

## Limitations

- If the number of individuals with single-cell data is small, the reference profile and gene-specific transformations may become unreliable
- A key assumption is that single-cell-based estimates of cell proportions accurately reflect the true proportions; accuracy may decrease if single-cell-captured proportions differ significantly from true physiological distributions
- Bisque provides estimates of cell proportions as reported by the single-cell technology used to generate the reference data
- Marker-based mode estimates relative abundance differences that cannot be compared across cell types

## Evidence

- [methods] Reference-based decomposition call with use.overlap=TRUE: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [methods] ExpressionSet input format requirement: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [methods] Validation by correlation with true proportions: "r <- cor(as.vector(ref.based.estimates), 
         as.vector(true.props[row.names(ref.based.estimates),colnames(ref.based.estimates)]))"
- [abstract] Rationale: gene-specific transformations improve performance under technical variation: "These transformations significantly improve decomposition performance compared to existing methods when there is significant technical variation in the generation of the reference profile"
- [readme] bulk.props result slot: [evidence span withheld — anchor status: external_doc]
- [discussion] Small-sample limitation: "if the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable."
