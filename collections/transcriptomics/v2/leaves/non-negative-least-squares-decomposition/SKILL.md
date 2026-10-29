---
name: non-negative-least-squares-decomposition
description: Use when bulk RNA-seq (ExpressionSet) and single-cell/snRNA-seq data
  from the same tissue are available — including overlapping individuals — and cell-type
  proportions must be estimated while correcting single-cell technical biases.
when_to_use_negative:
- No single-cell reference is available and only marker genes are known — use the
  marker-based mode, whose relative abundances cannot be compared across cell types.
- The single-cell data's cell-type composition is known to deviate strongly from physiological
  proportions (e.g. severe underrepresentation of a cell type of interest), since
  estimates reflect the single-cell technology's reported proportions.
- Very few individuals have single-cell data, making the reference profile and gene-specific
  transformations unreliable.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: BisqueRNA
  role: Provides ReferenceBasedDecomposition and internal helpers (GenerateSCReference,
    CalculateSCCellProportions, SupervisedTransformBulk, SemisupervisedTransformBulk,
    filter utilities) used for the NNLS decomposition
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Supplies the ExpressionSet container required for both bulk.eset and sc.eset
    inputs
- name: limSolve
  role: Solves the per-sample non-negative least-squares decomposition with non-negativity
    and sum-to-one constraints
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:b6ed8a8dd7e3925782ef49331096a10bd2c0d8259b9ada05e2332568f60bcdfd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - Bisque
  - BisqueRNA
  - limSolve
  - stats
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# non-negative-least-squares-decomposition

## Summary

Reference-based decomposition of bulk RNA-seq into cell-type proportions using Bisque: build a single-cell reference profile, learn gene-specific bulk transformations, then solve a non-negative least-squares (NNLS) regression per bulk sample with non-negativity and sum-to-one constraints.

## When to use

Use when bulk RNA-seq (ExpressionSet) and single-cell/snRNA-seq data from the same tissue are available — including overlapping individuals — and cell-type proportions must be estimated while correcting single-cell technical biases.

## When NOT to use

- No single-cell reference is available and only marker genes are known — use the marker-based mode, whose relative abundances cannot be compared across cell types.
- The single-cell data's cell-type composition is known to deviate strongly from physiological proportions (e.g. severe underrepresentation of a cell type of interest), since estimates reflect the single-cell technology's reported proportions.
- Very few individuals have single-cell data, making the reference profile and gene-specific transformations unreliable.

## Inputs

- bulk RNA-seq expression/count matrix converted to a Biobase ExpressionSet (bulk.eset)
- single-cell or single-nucleus RNA-seq counts with cell-type and individual labels as an ExpressionSet (sc.eset)
- optional Seurat object convertible via BisqueRNA::SeuratToExpressionSet
- optional marker gene list (markers=NULL for the reference-based path)

## Outputs

- Cell-type proportion estimates per bulk sample (res$bulk.props)
- Estimated-proportions table for downstream validation (e.g. decomposition_estimates.csv)
- Correlation of estimated vs. true/simulated proportions as a performance metric

## How to apply

Convert bulk and single-cell count matrices into Biobase ExpressionSet objects (sc.eset carries cell-type and individual labels, e.g. via BisqueRNA::SeuratToExpressionSet with delimiter '-' and position 2). Generate the reference profile by averaging per-individual read-count abundances within each cell type (GenerateSCReference) and compute single-cell-derived proportions per individual (CalculateSCCellProportions). Apply the per-gene supervised transform (SupervisedTransformBulk): for each gene, regress bulk expression on the single-cell-derived proportions across overlapping individuals to learn gene-specific coefficients mapping the sc reference to bulk scale; use the semisupervised shrinkage-variance variant (SemisupervisedTransformBulk) when use.overlap=FALSE. Pre-filter with FilterUnexpressedGenes, FilterZeroVarianceGenes, GetOverlappingGenes, and GetOverlappingSamples, and convert counts to CPM as needed. Finally solve the NNLS decomposition per bulk sample (e.g. via limSolve), enforcing non-negative coefficients that sum to one, and cross-check against BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE). The rationale is that differences between single-cell and bulk chemistries introduce gene-specific biases that break direct proportionality, so the transformations correct the reference before regression.

## Related tools

- **BisqueRNA** (Provides ReferenceBasedDecomposition and internal helpers (GenerateSCReference, CalculateSCCellProportions, SupervisedTransformBulk, SemisupervisedTransformBulk, filter utilities) used for the NNLS decomposition) — github.com/cozygene/bisque
- **Biobase** (Supplies the ExpressionSet container required for both bulk.eset and sc.eset inputs)
- **limSolve** (Solves the per-sample non-negative least-squares decomposition with non-negativity and sum-to-one constraints)

## Examples

```
library(BisqueRNA); res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE); props <- res$bulk.props
```

## Evaluation signals

- All estimated proportions are non-negative and sum to one within each bulk sample
- Estimated proportions correlate strongly with true (simulated) proportions via cor(as.vector(ref.based.estimates), as.vector(true.props[...]))
- From-scratch NNLS solution matches BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE) output
- Only overlapping genes between sc.eset and bulk.eset enter the regression (GetOverlappingGenes applied)
- Simulated benchmark reproducible via BisqueRNA::SimulateData(n.ind=10, n.genes=100, n.cells=500, cell.types=c('Neurons','Astrocytes','Oligodendrocytes','Microglia','Endothelial Cells'), avg.props=c(.5,.2,.2,.07,.03)) with individuals 6-10 kept in the single-cell ExpressionSet

## Limitations

- If the number of individuals with single-cell data is small, the reference profile and gene-specific transformations may become unreliable
- A key assumption is that single-cell-based estimates of cell proportions accurately reflect the true proportions; accuracy may decrease if single-cell-captured proportions differ significantly from true physiological distributions
- Bisque provides estimates of cell proportions as reported by the single-cell technology used to generate the reference data; take caution with known significant single-cell bias, such as severe underrepresentation of a cell type of interest

## Evidence

- [abstract] Bisque learns gene-specific bulk expression transformations that significantly improve decomposition performance compared to existing methods under significant technical variation: "These transformations significantly improve decomposition performance compared to existing methods when there is significant technical variation in the generation of the reference profile and"
- [methods] Reference-based decomposition call in methods: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [methods] SupervisedTransformBulk internal helper: "- `SupervisedTransformBulk(gene, Y.train, X.train, X.pred)`"
- [intro] MuSiC comparison describing NNLS-based decomposition: "MuSiC16 also leverages single-cell expression as a reference, instead using a weighted non-negative least-squares regression (NNLS) model for decomposition"
- [methods] ExpressionSet requirement for inputs: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [methods] Validation by correlation with true proportions: "r <- cor(as.vector(ref.based.estimates), 
         as.vector(true.props[row.names(ref.based.estimates),colnames(ref.based.estimates)]))"
- [readme] README description of reference-based mode assumptions: "We assume that both single-cell and bulk counts are measured from the same tissue. Specifically, the cell composition of the labeled single-cell data should match the expected physiological"
