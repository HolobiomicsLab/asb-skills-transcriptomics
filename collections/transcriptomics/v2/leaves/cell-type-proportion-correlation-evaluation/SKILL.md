---
name: cell-type-proportion-correlation-evaluation
description: Use when a decomposition method's estimated cell-type proportions (e.g.,
  BisqueRNA's res$bulk.props) must be scored against known or reference proportions
  per individual and cell type using Pearson correlation and RMSD.
when_to_use_negative:
- No reference or matched single-cell-derived proportions exist to correlate against
  — use BisqueMarker's marker-based mode instead, whose relative abundances 'cannot
  be compared across cell types'
- Estimated proportions from different decomposition modes are being compared between
  cell types — BisqueMarker output is relative abundance, not proportions, so R/RMSD
  against proportions is invalid
- Single-cell-derived proportions are known to severely misrepresent true physiological
  composition (e.g., a cell type is severely underrepresented in the snRNA-seq reference),
  corrupting the ground-truth for correlation
tools:
- name: BisqueRNA
  role: Produces the estimated cell proportions (res$bulk.props) from ReferenceBasedDecomposition
    that are correlated with the snRNA-seq-derived reference proportions; supplies
    internal helpers (CountsToCPM, FilterUnexpressedGenes, FilterZeroVarianceGenes,
    SupervisedTransformBulk, CalculateSCCellProportions)
  repo: https://github.com/cozygene/bisque
- name: Biobase
  role: Provides the ExpressionSet container required for both bulk and single-cell
    inputs to BisqueRNA, whose phenoData (SubjectName) and sampleNames must match
    for paired evaluation
- name: CIBERSORTx
  role: Comparison method on the same LOOCV folds; its estimates are scored identically
    (R = 0.671 on DLPFC) to establish Bisque's superior global accuracy
- name: MuSiC
  role: Comparison method (weighted non-negative least-squares regression) evaluated
    on the same folds with the same R/RMSD metrics
- name: BSEQ-sc
  role: Comparison method (single-cell reference profile plugged into the CIBERSORT
    model) evaluated on the same folds with the same R/RMSD metrics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:37e3deeb324197d37d1d70b81ae002f15eaa02337a279d320313b1b290464d80
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# cell-type-proportion-correlation-evaluation

## Summary

Evaluates bulk RNA-seq cell-type decomposition accuracy by computing Pearson correlation (R) and RMSD between a method's estimated proportions and single-nucleus-derived reference proportions across individuals and cell types, as used to benchmark Bisque against CIBERSORTx, MuSiC, BSEQ-sc, and CIBERSORT on the DLPFC leave-one-out cross-validation.

## When to use

Use when a decomposition method's estimated cell-type proportions (e.g., BisqueRNA's res$bulk.props) must be scored against known or reference proportions per individual and cell type using Pearson correlation and RMSD.

## When NOT to use

- No reference or matched single-cell-derived proportions exist to correlate against — use BisqueMarker's marker-based mode instead, whose relative abundances 'cannot be compared across cell types'
- Estimated proportions from different decomposition modes are being compared between cell types — BisqueMarker output is relative abundance, not proportions, so R/RMSD against proportions is invalid
- Single-cell-derived proportions are known to severely misrepresent true physiological composition (e.g., a cell type is severely underrepresented in the snRNA-seq reference), corrupting the ground-truth for correlation

## Inputs

- Estimated cell-type proportions per individual and cell type (e.g., res$bulk.props from BisqueRNA::ReferenceBasedDecomposition)
- Reference ('true') cell-type proportions derived from snRNA-seq labels (cellType, SubjectName) via CalculateSCCellProportions
- Bulk and single-cell ExpressionSet objects (Biobase) with matched SubjectName identifiers across the 8 paired DLPFC individuals

## Outputs

- Pearson correlation (R) of estimated vs reference proportions across all individuals and cell types
- RMSD between estimated and reference proportions
- Per-method summary table (e.g., table3_dlpfc_decomposition_performance.csv) with mean correlation and RMSD

## How to apply

Run the decomposition (e.g., BisqueRNA::ReferenceBasedDecomposition with markers=NULL and use.overlap=TRUE, which learns gene-specific bulk expression transformations via SupervisedTransformBulk from overlapping individuals) on the same folds as each comparison method, then flatten estimated and true proportions to matched row/column order. Compute Pearson correlation of the estimates against the reference (single-nucleus-derived) proportions across all individuals and cell types, plus RMSD. Under the DLPFC LOOCV protocol (8 paired individuals, 25% nuclei subsampled per the CIBERSORTx constraint, CPM normalization after filtering unexpressed and zero-variance genes), correct application yields Bisque R = 0.924 and RMSD = 0.029, the highest global accuracy among benchmarked methods. Before scoring, confirm the 8 individual IDs appear in both sc.eset$SubjectName and sampleNames(bulk.eset) so estimated and true proportions are paired correctly.

## Related tools

- **BisqueRNA** (Produces the estimated cell proportions (res$bulk.props) from ReferenceBasedDecomposition that are correlated with the snRNA-seq-derived reference proportions; supplies internal helpers (CountsToCPM, FilterUnexpressedGenes, FilterZeroVarianceGenes, SupervisedTransformBulk, CalculateSCCellProportions)) — https://github.com/cozygene/bisque
- **Biobase** (Provides the ExpressionSet container required for both bulk and single-cell inputs to BisqueRNA, whose phenoData (SubjectName) and sampleNames must match for paired evaluation)
- **CIBERSORTx** (Comparison method on the same LOOCV folds; its estimates are scored identically (R = 0.671 on DLPFC) to establish Bisque's superior global accuracy)
- **MuSiC** (Comparison method (weighted non-negative least-squares regression) evaluated on the same folds with the same R/RMSD metrics)
- **BSEQ-sc** (Comparison method (single-cell reference profile plugged into the CIBERSORT model) evaluated on the same folds with the same R/RMSD metrics)

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE); r <- cor(as.vector(res$bulk.props), as.vector(true.props[row.names(res$bulk.props),colnames(res$bulk.props)]))
```

## Evaluation signals

- Pearson r computed via cor(as.vector(estimates), as.vector(true.props)) with matched row (cell type) and column (individual) ordering
- On the DLPFC 25%-nuclei LOOCV benchmark, Bisque attains R = 0.924 and RMSD = 0.029, exceeding CIBERSORTx (R = 0.671), MuSiC, BSEQ-sc, and CIBERSORT
- Pre-check invariant: the 8 individual IDs appear in both sc.eset$SubjectName and sampleNames(bulk.eset) before pairing estimates to truth
- Output schema check: summary table contains one mean-correlation and one RMSD value per method (table3_dlpfc_decomposition_performance.csv)
- Every cell population identified from the snRNA-seq data is detected in the Bisque estimates (no dropped cell types in res$bulk.props)

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable, degrading evaluation validity
- A key assumption of the transformation framework is that single-cell-based estimates of cell proportions accurately reflect the true proportions being estimated
- Bisque provides estimates of cell proportions reported by the single-cell technology used to generate the reference data, so accuracy may decrease if captured proportions differ significantly from true physiological distributions
- Take caution if there is a known significant bias in the single-cell measurements of a tissue, such as severe underrepresentation of a cell type of interest

## Evidence

- [methods] Validation of decomposition estimates by correlation with true proportions: "r <- cor(as.vector(ref.based.estimates), 
         as.vector(true.props[row.names(ref.based.estimates),colnames(ref.based.estimates)]))"
- [discussion] Bisque outperforms existing methods on snRNA-seq datasets: "Bisque effectively leverages single-cell information to decompose bulk expression samples, outperforming existing methods in datasets with snRNA-seq data available."
- [readme] Proportions are returned in res$bulk.props: [evidence span withheld — anchor status: external_doc]
- [abstract] Learned transformations drive improved decomposition accuracy: "These transformations significantly improve decomposition performance compared to existing methods when there is significant technical variation"
- [methods] Reference-based decomposition call used in evaluation: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
