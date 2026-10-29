---
name: single-nucleus-rnaseq-decomposition
description: Use when bulk RNA-seq samples must be decomposed into cell-type proportions
  and a snRNA-seq/scRNA-seq reference with cellType and SubjectName labels is available
  for some individuals (e.g., the 8 paired DLPFC individuals), especially under known
  single-cell vs. bulk technical variation.
when_to_use_negative:
- No reference expression profile is available (no single-cell data) — use BisqueRNA::MarkerBasedDecomposition
  (BisqueMarker) with known marker genes instead, noting it yields relative abundances
  not comparable across cell types.
- There is known significant bias in the single-cell measurements, such as severe
  underrepresentation of a cell type of interest, since Bisque's estimates reflect
  proportions as reported by the single-cell technology used for the reference.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BisqueRNA
  role: Runs the reference-based decomposition (ReferenceBasedDecomposition) and the
    ExpressionSet conversion helpers (SeuratToExpressionSet, FilterUnexpressedGenes,
    FilterZeroVarianceGenes, CountsToCPM, SupervisedTransformBulk, CalculateSCCellProportions)
  repo: https://github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet container required for both bulk.eset and sc.eset
    inputs
- name: CIBERSORTx
  role: Comparison method in the DLPFC LOOCV benchmark (R = 0.671 vs. Bisque's R =
    0.924)
- name: CIBERSORT
  role: 'Comparison method: SVM regression on a purified-cell reference, run on the
    same LOOCV folds'
- name: BSEQ-sc
  role: 'Comparison method: single-cell reference profile plugged into the CIBERSORT
    model'
- name: MuSiC
  role: 'Comparison method: weighted non-negative least-squares regression on the
    same folds'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:2202ce406a26264ac00e91ff7ffebecd7f09373c69fcdeeae0b4eb3ca92c4ada
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
  - CIBERSORTx
  - MuSiC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-nucleus-rnaseq-decomposition

## Summary

Estimates cell-type proportions in bulk RNA-seq using a snRNA-seq/scRNA-seq reference via BisqueRNA::ReferenceBasedDecomposition, which learns gene-specific bulk expression transformations (SupervisedTransformBulk) to correct for technical differences between single-nucleus and bulk chemistries. This is the reference-based mode of Bisque, benchmarked on the DLPFC dataset against CIBERSORT, BSEQ-sc, and MuSiC.

## When to use

Use when bulk RNA-seq samples must be decomposed into cell-type proportions and a snRNA-seq/scRNA-seq reference with cellType and SubjectName labels is available for some individuals (e.g., the 8 paired DLPFC individuals), especially under known single-cell vs. bulk technical variation.

## When NOT to use

- No reference expression profile is available (no single-cell data) — use BisqueRNA::MarkerBasedDecomposition (BisqueMarker) with known marker genes instead, noting it yields relative abundances not comparable across cell types.
- There is known significant bias in the single-cell measurements, such as severe underrepresentation of a cell type of interest, since Bisque's estimates reflect proportions as reported by the single-cell technology used for the reference.

## Inputs

- snRNA-seq/scRNA-seq count matrix with per-cell cellType and SubjectName labels (ExpressionSet, or Seurat v3 object convertible via SeuratToExpressionSet)
- Bulk RNA-seq count matrix for individuals from the same tissue (converted to Biobase ExpressionSet)
- Overlap-usable subject IDs present in both sc.eset$SubjectName and sampleNames(bulk.eset)

## Outputs

- Per-individual, per-cell-type estimated proportions in res$bulk.props
- Pearson correlation (R) and RMSD of estimates vs. single-nucleus-derived reference proportions (e.g., DLPFC Table 3: R = 0.924, RMSD = 0.029)
- Summary table of per-method decomposition performance (e.g., table3_dlpfc_decomposition_performance.csv)

## How to apply

Convert both the snRNA-seq counts (with cellType and SubjectName metadata) and the paired bulk counts into Biobase ExpressionSet objects, using BisqueRNA::SeuratToExpressionSet with delimiter '-' and position 2 when starting from a Seurat v3 object. Apply the filter steps FilterUnexpressedGenes, FilterZeroVarianceGenes, and CountsToCPM (CPM normalization), and under CIBERSORTx-style constraints subsample 25% of nuclei per individual; verify every individual ID appears in both sc.eset$SubjectName and sampleNames(bulk.eset) so the overlap can be used. Run BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE), which leverages the overlapping individuals to learn gene-specific bulk expression transformations (SupervisedTransformBulk) rather than assuming single-cell and bulk measurements are directly proportional. Validate by correlating the returned res$bulk.props against single-nucleus-derived reference proportions (CalculateSCCellProportions) across individuals and cell types using Pearson R and RMSD; on the DLPFC leave-one-out benchmark the expected performance is R = 0.924 and RMSD = 0.029, exceeding CIBERSORTx (R = 0.671), MuSiC, BSEQ-sc, and CIBERSORT.

## Related tools

- **BisqueRNA** (Runs the reference-based decomposition (ReferenceBasedDecomposition) and the ExpressionSet conversion helpers (SeuratToExpressionSet, FilterUnexpressedGenes, FilterZeroVarianceGenes, CountsToCPM, SupervisedTransformBulk, CalculateSCCellProportions)) — https://github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet container required for both bulk.eset and sc.eset inputs)
- **CIBERSORTx** (Comparison method in the DLPFC LOOCV benchmark (R = 0.671 vs. Bisque's R = 0.924))
- **CIBERSORT** (Comparison method: SVM regression on a purified-cell reference, run on the same LOOCV folds)
- **BSEQ-sc** (Comparison method: single-cell reference profile plugged into the CIBERSORT model)
- **MuSiC** (Comparison method: weighted non-negative least-squares regression on the same folds)

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)
```

## Evaluation signals

- The 8 paired individuals' IDs all appear in both sc.eset$SubjectName and sampleNames(bulk.eset) before decomposition.
- Estimated proportions are retrieved from res$bulk.props and cover every cell population identified in the snRNA-seq data.
- On the DLPFC LOOCV benchmark, Pearson R ≈ 0.924 and RMSD ≈ 0.029 vs. single-nucleus-derived reference proportions, beating CIBERSORTx (R = 0.671), MuSiC, BSEQ-sc, and CIBERSORT.
- Unexpressed and zero-variance genes are filtered and counts are CPM-normalized prior to decomposition.
- With use.overlap=TRUE, gene-specific bulk transformations (SupervisedTransformBulk) are learned from the overlapping matched samples rather than assuming direct sc/bulk proportionality.

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable.
- A key assumption of the transformation framework is that single-cell-based estimates of cell proportions accurately reflect the true proportions being estimated; accuracy may decrease if single-cell-captured proportions differ significantly from true physiological distributions.
- Bisque provides estimates of cell proportions as reported by the single-cell technology used to generate the reference data.

## Evidence

- [methods] Reference-based decomposition call: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [abstract] Bisque input format and sc/bulk integration: "utilizes single-cell RNA-seq (scRNA-seq) or single-nucleus RNA-seq (snRNA-seq) data to generate a reference expression profile"
- [readme] README description of reference-based mode and tissue-matching assumption: "We assume that both single-cell and bulk counts are measured from the same tissue."
- [readme] Output location of proportions: [evidence span withheld — anchor status: external_doc]
- [discussion] Limitation: few single-cell individuals undermine reference and transformations: "if the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable."
