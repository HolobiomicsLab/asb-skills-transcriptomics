---
name: simulated-bulk-expression-benchmarking
description: Use when benchmarking cell-type decomposition robustness under simulated
  gene-specific bias between bulk RNA-seq and a snRNA-seq reference, e.g. to quantify
  how R and RMSD degrade as bias variance sigma increases.
when_to_use_negative:
- The goal is decomposition of real bulk data rather than controlled benchmarking
  of decomposition robustness under simulated bias
- No single-cell/snRNA-seq reference with cell-type and individual labels is available,
  so reference-based methods cannot be compared fairly
- Single-cell measurements have known severe bias, e.g. a cell type of interest is
  severely underrepresented, which violates Bisque's key assumption
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: Bisque/BisqueRNA
  role: Reference-based decomposition with gene-specific transformations (use.overlap=TRUE)
    being benchmarked for robustness
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Provides ExpressionSet format required by Bisque for bulk and single-nucleus
    expression data
- name: CIBERSORT/BSEQ-sc
  role: Comparator method using SVM regression on a single-cell-derived reference
- name: MuSiC
  role: Comparator method using weighted non-negative least-squares (NNLS) regression
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:fb60d388083c5b3c6bae105a8003e0a1e32e8aa6b9a8236b39eb449b3ca74c6b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSEQ-sc
  - Biobase
  - Bisque
  - Bisque/BisqueRNA
  - BisqueRNA
  - CIBERSORT
  - CIBERSORT/BSEQ-sc
  - MuSiC
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# simulated-bulk-expression-benchmarking

## Summary

Reproduces the gene-specific bias simulation from Jew et al. (2020) in which bulk expression is synthesized from adipose snRNA-seq counts and perturbed with gene-specific linear biases of increasing variance, to benchmark Bisque's robustness (R ≈ 0.85, RMSD ≈ 0.07 at sigma = 20) against CIBERSORT/BSEQ-sc and MuSiC.

## When to use

Use when benchmarking cell-type decomposition robustness under simulated gene-specific bias between bulk RNA-seq and a snRNA-seq reference, e.g. to quantify how R and RMSD degrade as bias variance sigma increases.

## When NOT to use

- The goal is decomposition of real bulk data rather than controlled benchmarking of decomposition robustness under simulated bias
- No single-cell/snRNA-seq reference with cell-type and individual labels is available, so reference-based methods cannot be compared fairly
- Single-cell measurements have known severe bias, e.g. a cell type of interest is severely underrepresented, which violates Bisque's key assumption

## Inputs

- adipose snRNA-seq counts matrix with cell-type and individual labels
- true cell-type proportions for overlapping bulk samples (true.props)
- bias levels sigma in {0, 5, 10, 20}
- 10 replicate bulk expression matrices per sigma level

## Outputs

- bisque_bias_simulation_results.csv with per-method, per-sigma, per-replicate estimates and R/RMSD metrics
- bulk.props cell-type proportion estimates from each decomposition method

## How to apply

Load adipose snRNA-seq counts with cell-type and individual labels plus true cell-type proportions for overlapping bulk samples, converting both to Biobase ExpressionSet objects (sc.eset, bulk.eset). For each bias level sigma in {0, 5, 10, 20}, draw gene-specific linear coefficient and intercept perturbations from a half-normal distribution, add the resulting linear bias to the bulk expression, and generate 10 replicate bulk matrices per sigma (analogous to BisqueRNA::SimulateData with fixed cell.types and avg.props). Run BisqueRNA::ReferenceBasedDecomposition with use.overlap=TRUE so gene-specific transformations are learned from overlapping samples, collecting res$bulk.props, and run CIBERSORT/BSEQ-sc (SVM regression on a single-cell-derived reference) and MuSiC (weighted NNLS) on the identical biased bulk inputs. For each method, sigma, and replicate, correlate estimates with true.props and compute RMSD, writing results to bisque_bias_simulation_results.csv.

## Related tools

- **Bisque/BisqueRNA** (Reference-based decomposition with gene-specific transformations (use.overlap=TRUE) being benchmarked for robustness) — github.com/cozygene/bisque
- **Biobase** (Provides ExpressionSet format required by Bisque for bulk and single-nucleus expression data)
- **CIBERSORT/BSEQ-sc** (Comparator method using SVM regression on a single-cell-derived reference)
- **MuSiC** (Comparator method using weighted non-negative least-squares (NNLS) regression)

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)
```

## Evaluation signals

- At the highest bias level (sigma = 20), Bisque attains R ≈ 0.85 and RMSD ≈ 0.07
- Comparator methods (CIBERSORT/BSEQ-sc, MuSiC) degrade relative to Bisque as sigma increases
- R is computed by correlating each method's bulk.props estimates against true.props row/column-matched to the estimates
- Exactly 10 replicate bulk matrices are generated per sigma level {0, 5, 10, 20}
- Results CSV contains estimates and R/RMSD metrics for every method, sigma, and replicate combination

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable
- A key assumption of the transformation framework is that single-cell-based estimates of cell proportions accurately reflect the true proportions to be estimated
- Bisque estimates reflect the proportions reported by the single-cell technology used for the reference data, so accuracy may decrease if single-cell-captured proportions differ significantly from true physiological distributions
- Take caution if there is known significant bias in single-cell measurements, such as severe underrepresentation of a cell type of interest

## Evidence

- [methods] Simulate single-cell and bulk RNA-seq count data: "sim.data <- SimulateData(n.ind=10, n.genes=100, n.cells=500, cell.types=cell.types, avg.props=avg.props)"
- [methods] Reference-based decomposition call: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [methods] Validation by correlation with true proportions: "r <- cor(as.vector(ref.based.estimates), as.vector(true.props[row.names(ref.based.estimates),colnames(ref.based.estimates)]))"
- [abstract] Bisque robustness under technical variation: "These transformations significantly improve decomposition performance compared to existing methods when there is significant technical variation in the generation of the reference profile and"
- [methods] Bisque expression set format requirement: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [readme] Result access from decomposition output: [evidence span withheld — anchor status: external_doc]
