---
name: proportion-estimate-correlation-accuracy-evaluation
description: Use when comparing decomposition methods (e.g., Bisque vs MuSiC/CIBERSORTx)
  on samples with paired bulk RNA-seq and snRNA/scRNA-seq data, where single-cell-derived
  proportions serve as ground truth.
when_to_use_negative:
- No paired single-cell ground-truth proportions exist for the individuals, so correlation/RMSD
  against true props cannot be computed
- Estimates come from marker-based decomposition (BisqueMarker), which yields relative
  abundances that cannot be compared across cell types or treated as proportions
- The single-cell reference comes from very few individuals, making the reference
  profile and gene-specific transformations unreliable as a truth standard
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BisqueRNA
  role: Performs reference-based decomposition (ReferenceBasedDecomposition with markers=NULL,
    use.overlap=TRUE) whose bulk.props estimates are scored against true proportions
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet container holding bulk and single-cell matrices
    with SubjectName/cellType phenoData used for overlap and truth extraction
- name: MuSiC
  role: Baseline NNLS-based decomposition method benchmarked in the LOOCV accuracy
    comparison
- name: CIBERSORT
  role: SVM-regression baseline decomposition method included in the per-method accuracy
    comparison
- name: BSEQ-sc
  role: Single-cell-reference baseline method benchmarked against Bisque in the accuracy
    evaluation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:8ebb5a02da1509e1d919297fa5b257c9336c1513054e38433d116783c85cbdf8
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
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# proportion-estimate-correlation-accuracy-evaluation

## Summary

Evaluates cell-type decomposition accuracy by correlating estimated bulk cell proportions (from Bisque, CIBERSORTx, MuSiC, BSEQ-sc, CIBERSORT) against single-cell-derived true proportions across leave-one-out cross-validation rounds, reporting global Pearson R and RMSD per method.

## When to use

Use when comparing decomposition methods (e.g., Bisque vs MuSiC/CIBERSORTx) on samples with paired bulk RNA-seq and snRNA/scRNA-seq data, where single-cell-derived proportions serve as ground truth.

## When NOT to use

- No paired single-cell ground-truth proportions exist for the individuals, so correlation/RMSD against true props cannot be computed
- Estimates come from marker-based decomposition (BisqueMarker), which yields relative abundances that cannot be compared across cell types or treated as proportions
- The single-cell reference comes from very few individuals, making the reference profile and gene-specific transformations unreliable as a truth standard

## Inputs

- Bulk RNA-seq expression matrix as a Biobase ExpressionSet (bulk.eset)
- scRNA-seq/snRNA-seq counts matrix as a Biobase ExpressionSet with phenoData containing SubjectName and cellType columns (sc.eset)
- Single-cell-derived true cell-type proportions per individual (e.g., via CalculateSCCellProportions)
- Per-method bulk.props proportion estimates for held-out individuals

## Outputs

- Per-method global Pearson correlation (R) between estimates and true proportions
- Per-method RMSD between estimates and true proportions
- Summary accuracy table (e.g., table2_decomposition_metrics.csv reproducing Table 2)

## How to apply

Format paired bulk and single-cell data as Biobase ExpressionSet objects with phenoData carrying SubjectName and cellType, then identify overlapping samples/genes (GetOverlappingSamples, GetOverlappingGenes) and apply FilterUnexpressedGenes/FilterZeroVarianceGenes. Generate the single-cell reference via CalculateSCCellProportions and GenerateSCReference, then run each method's decomposition (e.g., BisqueRNA::ReferenceBasedDecomposition with markers=NULL, use.overlap=TRUE) in leave-one-out rounds over the paired individuals, holding out one individual's single-cell-derived proportions as truth. Collect res$bulk.props estimates for each held-out individual and cell type, then compute Pearson correlation and RMSD against the true proportions using cor(as.vector(estimates), as.vector(true.props[...])) as in the article's validation step. Assemble per-method R and RMSD into a summary table (e.g., Table 2 in the adipose analysis where Bisque reached R = 0.923, RMSD = 0.074).

## Related tools

- **BisqueRNA** (Performs reference-based decomposition (ReferenceBasedDecomposition with markers=NULL, use.overlap=TRUE) whose bulk.props estimates are scored against true proportions) — github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet container holding bulk and single-cell matrices with SubjectName/cellType phenoData used for overlap and truth extraction)
- **MuSiC** (Baseline NNLS-based decomposition method benchmarked in the LOOCV accuracy comparison)
- **CIBERSORT** (SVM-regression baseline decomposition method included in the per-method accuracy comparison)
- **BSEQ-sc** (Single-cell-reference baseline method benchmarked against Bisque in the accuracy evaluation)

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE); r <- cor(as.vector(res$bulk.props), as.vector(true.props[row.names(res$bulk.props),colnames(res$bulk.props)]))
```

## Evaluation signals

- Bisque global accuracy reproduces the reported Table 2 values: Pearson R = 0.923 and RMSD = 0.074 on the 6 adipose individuals
- cor() computed between estimate vectors and true.props aligned by row.names and colnames of the estimates
- Each of the 6 individuals is held out exactly once per method in the leave-one-out rounds, with all cell types scored
- All methods compared on identical overlapping samples and genes after FilterUnexpressedGenes/FilterZeroVarianceGenes
- Output table contains one R and one RMSD row per method (Bisque, CIBERSORTx, MuSiC, BSEQ-sc, CIBERSORT, semi-supervised Bisque variant)

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable
- A key assumption is that single-cell-based estimates of cell proportions accurately reflect the true proportions; accuracy may decrease if single-cell-captured proportions differ significantly from true physiological distributions
- Advise caution if there is known significant bias in single-cell measurements, such as severe underrepresentation of a cell type of interest

## Evidence

- [methods] Validate decomposition estimates by correlation with true proportions: "r <- cor(as.vector(ref.based.estimates), 
         as.vector(true.props[row.names(ref.based.estimates),colnames(ref.based.estimates)]))"
- [methods] Reference-based decomposition call and overlap flags: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [readme] README on where estimates are stored: [evidence span withheld — anchor status: external_doc]
- [discussion] Truth assumption and small-n limitation: "if the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable."
- [readme] Marker-based abundances are not proportions: [evidence span withheld — anchor status: external_doc]
