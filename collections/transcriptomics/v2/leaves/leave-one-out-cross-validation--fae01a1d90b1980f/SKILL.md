---
name: leave-one-out-cross-validation
description: Use when comparing cell-composition estimation methods (Bisque, CIBERSORT,
  BSEQ-sc, MuSiC, CIBERSORTx) on a cohort where a subset of individuals has both bulk
  RNA-seq and single-nucleus/single-cell data, to get global R and RMSD.
when_to_use_negative:
- No individuals have both bulk and single-cell data, so no overlapping samples exist
  to learn the gene-specific bulk transformations — use the marker-based (BisqueMarker)
  mode instead
- Only a set of known marker genes is available with no reference profile, and relative
  (not comparable-across-cell-types) abundances suffice
- The goal is decomposing proportions for new samples rather than benchmarking estimator
  accuracy against single-cell-derived ground truth
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BisqueRNA
  role: Reference-based decomposition per LOOCV fold with markers=NULL and use.overlap=TRUE;
    estimates returned in res$bulk.props
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet containers required for bulk.eset and sc.eset inputs
- name: CIBERSORT
  role: 'Comparator: SVM regression on a purified-cell reference run on the same LOOCV
    folds'
- name: BSEQ-sc
  role: 'Comparator: single-cell reference profile plugged into the CIBERSORT model'
- name: MuSiC
  role: 'Comparator: weighted non-negative least-squares regression decomposition
    on the same folds'
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:1cfec186a1923e32aa2bf9d890f26743916291046db711c3b73aad7547916113
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# leave-one-out-cross-validation

## Summary

Benchmarks cell-type decomposition accuracy by holding out each individual with paired bulk RNA-seq and snRNA-seq data, building the reference from the remaining individuals, and scoring estimated proportions against single-cell-derived true proportions (Pearson R, RMSD).

## When to use

Use when comparing cell-composition estimation methods (Bisque, CIBERSORT, BSEQ-sc, MuSiC, CIBERSORTx) on a cohort where a subset of individuals has both bulk RNA-seq and single-nucleus/single-cell data, to get global R and RMSD.

## When NOT to use

- No individuals have both bulk and single-cell data, so no overlapping samples exist to learn the gene-specific bulk transformations — use the marker-based (BisqueMarker) mode instead
- Only a set of known marker genes is available with no reference profile, and relative (not comparable-across-cell-types) abundances suffice
- The goal is decomposing proportions for new samples rather than benchmarking estimator accuracy against single-cell-derived ground truth

## Inputs

- bulk.eset: Biobase ExpressionSet of bulk RNA-seq counts (one sample per individual)
- sc.eset: Biobase ExpressionSet of scRNA/snRNA-seq counts with phenoData columns SubjectName (individual) and cellType; can be built from a Seurat v3 object via BisqueRNA::SeuratToExpressionSet(seurat.obj, delimiter="-", position=2, version="v3")
- List of paired individual IDs present in both sc.eset$SubjectName and sampleNames(bulk.eset)
- Single-cell-derived true cell-type proportions per held-out individual (CalculateSCCellProportions)

## Outputs

- Per-fold, per-individual, per-cell-type proportion estimates in res$bulk.props for each method
- Per-method summary table of global Pearson correlation (R) and RMSD (e.g. table2_decomposition_metrics.csv, table3_dlpfc_decomposition_performance.csv)

## How to apply

Identify the paired individuals (6 subcutaneous adipose, 8 DLPFC in this paper) and verify each ID appears in both sc.eset$SubjectName and sampleNames(bulk.eset). For each LOOCV fold, drop one individual and run BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE) on the remaining pairs, letting the overlap samples drive the gene-specific bulk expression transformations (SupervisedTransformBulk); pre-filter with FilterUnexpressedGenes, FilterZeroVarianceGenes, and CPM normalization (e.g. the 25% nuclei subsampling constraint for DLPFC). Run the comparators on identical folds: CIBERSORT (SVM regression on a purified-cell reference), BSEQ-sc (single-cell reference in the CIBERSORT model), MuSiC (weighted NNLS), CIBERSORTx, and the semi-supervised SemisupervisedTransformBulk Bisque variant. Collect each method's res$bulk.props for the held-out individual, correlate with the single-cell-derived true proportions across all individuals and cell types, and compute Pearson R and RMSD per method. Acceptance is reproducing the paper's Tables 2–3: Bisque R=0.923/RMSD=0.074 (adipose) and R=0.924/RMSD=0.029 (DLPFC), with CIBERSORT at R=0.671 on DLPFC.

## Related tools

- **BisqueRNA** (Reference-based decomposition per LOOCV fold with markers=NULL and use.overlap=TRUE; estimates returned in res$bulk.props) — github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet containers required for bulk.eset and sc.eset inputs)
- **CIBERSORT** (Comparator: SVM regression on a purified-cell reference run on the same LOOCV folds)
- **BSEQ-sc** (Comparator: single-cell reference profile plugged into the CIBERSORT model)
- **MuSiC** (Comparator: weighted non-negative least-squares regression decomposition on the same folds)

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE); r <- cor(as.vector(res$bulk.props), as.vector(true.props[row.names(res$bulk.props),colnames(res$bulk.props)]))
```

## Evaluation signals

- Bisque reproduces Table 2 adipose accuracy: Pearson R = 0.923, RMSD = 0.074 across the 6 leave-one-out rounds
- Bisque reproduces Table 3 DLPFC accuracy: R = 0.924, RMSD = 0.029, beating CIBERSORTx (R = 0.671), MuSiC, BSEQ-sc, and CIBERSORT
- Every fold's held-out individual ID is found in both sc.eset$SubjectName and sampleNames(bulk.eset) before decomposition
- Each method's result object exposes res$bulk.props with one proportion per held-out individual per cell type
- Estimates correlate with true proportions via cor() on matched rows/columns of ref.based.estimates vs true.props

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable — LOOCV with only 6–8 paired individuals gives few folds
- A key assumption is that single-cell-based estimates of cell proportions accurately reflect the true proportions being estimated; accuracy may decrease if single-cell-captured proportions differ significantly from true physiological distributions
- Bisque provides estimates of cell proportions reported by the single-cell technology used to generate the reference data, so caution is advised when a cell type is severely underrepresented

## Evidence

- [methods] Reference-based decomposition call: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [methods] Validation by correlation with true proportions: "r <- cor(as.vector(ref.based.estimates), 
         as.vector(true.props[row.names(ref.based.estimates),colnames(ref.based.estimates)]))"
- [readme] Result object schema: [evidence span withheld — anchor status: external_doc]
- [discussion] Superiority claim underlying the benchmark: "Bisque effectively leverages single-cell information to decompose bulk expression samples, outperforming existing methods in datasets with snRNA-seq data available."
