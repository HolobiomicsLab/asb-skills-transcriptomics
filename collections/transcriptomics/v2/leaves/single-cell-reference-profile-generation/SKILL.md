---
name: single-cell-reference-profile-generation
description: Use when you have scRNA-seq or snRNA-seq counts with cell-type and individual
  labels from the same tissue as your bulk RNA-seq samples and need a reference profile
  for cell-type proportion estimation (decomposition).
when_to_use_negative:
- When no reference profile is available (no single-cell data for the tissue) — use
  the marker-based BisqueRNA::MarkerBasedDecomposition path instead, which estimates
  relative abundances from marker genes only
- When the labeled single-cell data come from a different tissue or its cell composition
  does not match the expected physiological composition (README scope statement)
- When single-cell measurements have known severe bias, e.g. severe underrepresentation
  of a cell type of interest — the article advises caution since estimates will reflect
  the single-cell technology's proportions
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Bisque
  role: Toolkit whose reference-based mode uses the single-cell-derived reference
    profile plus gene-specific bulk transformations
  repo: github.com/cozygene/bisque
- name: BisqueRNA
  role: R implementation providing ReferenceBasedDecomposition and internal reference-generation
    helpers (GenerateSCReference, CalculateSCCellProportions)
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet container format required for bulk.eset and sc.eset
    inputs
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:707ac8dce9ac7ae50862ff63f49b3d9e99e690b43999304739ad37a4d9236770
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# single-cell-reference-profile-generation

## Summary

Builds a cell-type reference expression profile by averaging per-individual read-count abundances within each cell type from scRNA-seq/snRNA-seq data (GenerateSCReference), as the first stage of Bisque's reference-based decomposition, where the reference is combined with learned gene-specific bulk transformations before NNLS regression.

## When to use

Use when you have scRNA-seq or snRNA-seq counts with cell-type and individual labels from the same tissue as your bulk RNA-seq samples and need a reference profile for cell-type proportion estimation (decomposition).

## When NOT to use

- When no reference profile is available (no single-cell data for the tissue) — use the marker-based BisqueRNA::MarkerBasedDecomposition path instead, which estimates relative abundances from marker genes only
- When the labeled single-cell data come from a different tissue or its cell composition does not match the expected physiological composition (README scope statement)
- When single-cell measurements have known severe bias, e.g. severe underrepresentation of a cell type of interest — the article advises caution since estimates will reflect the single-cell technology's proportions

## Inputs

- scRNA-seq or snRNA-seq counts matrix with cell-type and individual labels (as Biobase ExpressionSet / sc.eset)
- bulk RNA-seq ExpressionSet from the same tissue (for gene-overlap restriction and optional overlapping samples)
- Optional: cell-type marker gene list (BisqueMarker path)
- Seurat object (convertible via BisqueRNA::SeuratToExpressionSet)

## Outputs

- Cell-type reference profile (averaged per-cell-type read-count abundances over overlapping genes)
- Per-individual single-cell-derived cell-type proportions (CalculateSCCellProportions)
- Estimated bulk cell-type proportions (res$bulk.props from ReferenceBasedDecomposition)

## How to apply

Convert the single-cell counts plus cell-type and individual phenoData into a Biobase ExpressionSet (sc.eset); Bisque requires ExpressionSet format. Preprocess the eset with the package's filter steps (FilterUnexpressedGenes, FilterZeroVarianceGenes) and restrict to genes overlapping the bulk data (GetOverlappingGenes). Then average per-individual read-count abundances within each cell type (GenerateSCReference logic) to form the cell-type reference profile, and compute per-individual single-cell-derived proportions (CalculateSCCellProportions). Critically, the reference alone is not applied directly to bulk data: Bisque pairs it with gene-specific transformations of the bulk expression (SupervisedTransformBulk when use.overlap=TRUE with matched samples; SemisupervisedTransformBulk otherwise) so that per-gene technical biases between single-cell and bulk chemistries are corrected before non-negative least-squares decomposition. For practice, the SimulateData vignette pattern (n.ind=10, n.genes=100, n.cells=500, cell.types like Neurons/Astrocytes/Oligodendrocytes/Microglia/Endothelial Cells) provides a labeled single-cell ExpressionSet to exercise the workflow.

## Related tools

- **Bisque** (Toolkit whose reference-based mode uses the single-cell-derived reference profile plus gene-specific bulk transformations) — github.com/cozygene/bisque
- **BisqueRNA** (R implementation providing ReferenceBasedDecomposition and internal reference-generation helpers (GenerateSCReference, CalculateSCCellProportions)) — github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet container format required for bulk.eset and sc.eset inputs)

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)
```

## Evaluation signals

- Correlate estimated proportions (res$bulk.props) with true proportions via cor(as.vector(ref.based.estimates), as.vector(true.props[...])) — the vignette's validation step
- Confirm estimates are non-negative and sum to one per bulk sample (NNLS constraint from limSolve)
- Check that only genes overlapping between sc.eset and bulk.eset (GetOverlappingGenes) appear in the reference profile
- Cross-check the reference-based run (markers=NULL, use.overlap=TRUE) against the same call on simulated SimulateData output
- In bias simulations (sigma up to 20, half-normal gene-specific linear perturbations), Bisque should remain robust at R ≈ 0.85 and RMSD ≈ 0.07 where comparators (CIBERSORT/BSEQ-sc, MuSiC) degrade

## Limitations

- If the number of individuals with single-cell data is small, the reference profile and gene-specific transformations may become unreliable
- A key assumption is that single-cell-based estimates of cell proportions accurately reflect the true proportions to be estimated
- Bisque provides estimates of cell proportions as reported by the single-cell technology used to generate the reference; accuracy may decrease if captured proportions differ significantly from true physiological distributions
- Marker-based (BisqueMarker) estimates are relative abundance differences that cannot be compared across cell types

## Evidence

- [abstract] utilizes single-cell RNA-seq (scRNA-seq) or single-nucleus RNA-seq (snRNA-seq) data to generate a reference expression profile and learn gene-specific bulk expression transformations: "utilizes single-cell RNA-seq (scRNA-seq) or single-nucleus RNA-seq (snRNA-seq) data to generate a reference expression profile and learn gene-specific bulk expression transformations"
- [methods] res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE): "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [methods] Bisque requires expression data in the ExpressionSet format from the Biobase package.: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [readme] This method utilizes single-cell data to decompose bulk expression.: "This method utilizes single-cell data to decompose bulk expression."
- [readme] the cell composition of the labeled single-cell data should match the expected physiological composition: "Specifically, the cell composition of the labeled single-cell data should match the expected physiological composition."
