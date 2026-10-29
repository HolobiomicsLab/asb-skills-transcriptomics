---
name: correlation-based-decomposition-validation
description: Use when a bulk expression decomposition run has produced estimated cell
  proportions (e.g. res$bulk.props) and ground-truth proportions exist (from simulated
  data with known avg.props) so accuracy can be quantified.
when_to_use_negative:
- No ground-truth (or simulated) proportions exist, so correlation with true.props
  is impossible — use biological/phenotype replication instead (e.g. previously reported
  cell-proportion–phenotype associations)
- 'Evaluating BisqueMarker cross-cell-type accuracy: its estimates are relative abundance
  differences that cannot be compared across cell types, so raw proportion-level metrics
  are inappropriate without first scaling the true proportions'
- The single-cell reference proportions are known to differ strongly from true physiological
  distributions — correlation-based validation would certify a systematically biased
  estimate
tools:
- name: BisqueRNA
  role: Produces the decomposition estimates (bulk.props) that are correlated against
    true simulated proportions
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Supplies the ExpressionSet format required for bulk.eset and sc.eset inputs
    to the decomposition being validated
- name: stats
  role: Provides cor() for computing the R validation metric and scale() for matching
    BisqueMarker's relative abundances to true proportions
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:95bd052be5cf6e2eb540f54e7e561cc6e6b095d250531fc65db10518e8068957
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# correlation-based-decomposition-validation

## Summary

Validates cell-type proportion estimates produced by Bisque (or comparator decomposition methods such as CIBERSORT/BSEQ-sc and MuSiC) by correlating estimated proportions against true simulated proportions and reporting R and RMSD metrics.

## When to use

Use when a bulk expression decomposition run has produced estimated cell proportions (e.g. res$bulk.props) and ground-truth proportions exist (from simulated data with known avg.props) so accuracy can be quantified.

## When NOT to use

- No ground-truth (or simulated) proportions exist, so correlation with true.props is impossible — use biological/phenotype replication instead (e.g. previously reported cell-proportion–phenotype associations)
- Evaluating BisqueMarker cross-cell-type accuracy: its estimates are relative abundance differences that cannot be compared across cell types, so raw proportion-level metrics are inappropriate without first scaling the true proportions
- The single-cell reference proportions are known to differ strongly from true physiological distributions — correlation-based validation would certify a systematically biased estimate

## Inputs

- Estimated cell-type proportion matrix (res$bulk.props from ReferenceBasedDecomposition or MarkerBasedDecomposition), samples x cell types
- True/simulated cell-type proportion matrix (true.props) with overlapping sample and cell-type labels
- For bias simulations: per-replicate bulk.props estimates across sigma levels and comparator methods (CIBERSORT/BSEQ-sc, MuSiC)

## Outputs

- Correlation coefficient R between estimated and true proportions (vectorized over all samples and cell types)
- RMSD of estimates against true.props per method and replicate
- Validation table written to CSV, e.g. decomposition_estimates.csv or bisque_bias_simulation_results.csv

## How to apply

After running decomposition — e.g. BisqueRNA::ReferenceBasedDecomposition on a bulk.eset/sc.eset pair with use.overlap=TRUE — collect the estimated proportions from res$bulk.props and subset the true proportions matrix to matching rows (samples/individuals) and columns (cell types, e.g. Neurons, Astrocytes, Oligodendrocytes, Microglia, Endothelial Cells from SimulateData with n.ind=10, n.genes=100, n.cells=500). Compute the correlation between all estimate-true pairs as vectors using cor(as.vector(estimates), as.vector(true.props[matched, matched])). For robustness simulations (gene-specific bias levels sigma in {0, 5, 10, 20}), additionally compute RMSD against true.props per method and replicate, and compare across methods (Bisque vs CIBERSORT/BSEQ-sc vs MuSiC). For marker-based BisqueMarker estimates, which are relative abundances, first scale the true proportions (t(scale(t(true.props)))) before correlating, and require a positive correlation r consistent with the paper's reported R. Write all estimates and R/RMSD metrics to a results CSV (e.g. decomposition_estimates.csv or bisque_bias_simulation_results.csv).

## Related tools

- **BisqueRNA** (Produces the decomposition estimates (bulk.props) that are correlated against true simulated proportions) — github.com/cozygene/bisque
- **Biobase** (Supplies the ExpressionSet format required for bulk.eset and sc.eset inputs to the decomposition being validated)
- **stats** (Provides cor() for computing the R validation metric and scale() for matching BisqueMarker's relative abundances to true proportions)

## Examples

```
r <- cor(as.vector(ref.based.estimates), as.vector(true.props[row.names(ref.based.estimates), colnames(ref.based.estimates)]))
```

## Evaluation signals

- All estimated proportions from a reference-based run positively correlate with true simulated proportions; correlation is computed on the subset of true.props matching the estimate row names and column names
- At the highest simulated gene-specific bias level (sigma = 20), Bisque attains R ≈ 0.85 and RMSD ≈ 0.07 while comparator methods (CIBERSORT/BSEQ-sc, MuSiC) degrade relative to Bisque
- Marker-based estimates correlate positively with scaled true proportions t(scale(t(true.props))), consistent with the paper's reported R value
- A from-scratch implementation (e.g. NNLS via limSolve with non-negative, sum-to-one coefficients) matches BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE) as a sanity reference
- All estimates and R/RMSD metrics are written to the results CSV with rows/columns aligned to samples and cell types

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable
- Bisque estimates reflect the proportions reported by the single-cell technology used to generate the reference; accuracy may decrease if single-cell-captured proportions differ significantly from true physiological distributions
- BisqueMarker estimates relative differences in abundances that cannot be compared across cell types

## Evidence

- [methods] validation correlation call: "r <- cor(as.vector(ref.based.estimates), as.vector(true.props[row.names(ref.based.estimates),colnames(ref.based.estimates)]))"
- [abstract] transformation robustness claim: "These transformations significantly improve decomposition performance compared to existing methods when there is significant technical variation"
- [discussion] small-sample limitation: "if the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable."
- [readme] result container for validation: [evidence span withheld — anchor status: external_doc]
- [intro] accuracy claim: "We demonstrate using simulated and real datasets from brain and adipose tissue that our method is significantly more accurate than existing methods"
