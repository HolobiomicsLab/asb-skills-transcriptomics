---
name: gene-specific-linear-bias-simulation
description: Use when evaluating or benchmarking bulk RNA-seq cell-type decomposition
  methods under simulated single-cell-vs-bulk technical bias, especially with snRNA-seq
  reference data such as adipose nuclei.
when_to_use_negative:
- The goal is decomposing real bulk data rather than stress-testing method robustness
  under synthetic gene-specific bias
- No overlapping individuals between single-cell and bulk data exist, since use.overlap=TRUE
  gene-specific transformation learning requires matched samples
- The reference is marker-gene-only (no single-cell reference profile available);
  BisqueMarker rather than this reference-based simulation applies
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BisqueRNA
  role: Runs ReferenceBasedDecomposition on biased bulk ExpressionSets with the snRNA-seq
    reference (use.overlap=TRUE) to collect bulk.props estimates
  repo: github.com/cozygene/bisque
- name: Biobase
  role: Wraps bulk and single-nucleus count matrices as ExpressionSet objects (bulk.eset,
    sc.eset) required by Bisque
- name: CIBERSORT
  role: 'Comparator method: SVM-regression decomposition on a single-cell-derived
    reference applied to the identical biased bulk inputs'
- name: BSEQ-sc
  role: 'Comparator method: generates a reference profile from single-cell expression
    data used in the CIBERSORT model'
- name: MuSiC
  role: 'Comparator method: weighted non-negative least-squares (NNLS) decomposition
    applied to the same biased bulk inputs'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15816-6
    title: Accurate estimation of cell composition in bulk expression through robust
      integration of single-cell information
  source_content_hash: sha256:58cc92cffefa68af5c9f70fb52579eb16e440f20b02169349586fdd2d153df99
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gene-specific-linear-bias-simulation

## Summary

A simulation skill that stresses reference-based decomposition methods by adding gene-specific linear biases (coefficient + intercept perturbations drawn from a half-normal distribution) to bulk expression synthesized from adipose snRNA-seq counts, quantifying robustness of Bisque vs. CIBERSORT/BSEQ-sc and MuSiC via correlation (R) and RMSD against true proportions.

## When to use

Use when evaluating or benchmarking bulk RNA-seq cell-type decomposition methods under simulated single-cell-vs-bulk technical bias, especially with snRNA-seq reference data such as adipose nuclei.

## When NOT to use

- The goal is decomposing real bulk data rather than stress-testing method robustness under synthetic gene-specific bias
- No overlapping individuals between single-cell and bulk data exist, since use.overlap=TRUE gene-specific transformation learning requires matched samples
- The reference is marker-gene-only (no single-cell reference profile available); BisqueMarker rather than this reference-based simulation applies

## Inputs

- Adipose snRNA-seq count matrix with cell-type and individual labels
- True cell-type proportions for bulk samples overlapping the single-cell data
- Bulk expression matrices to be perturbed (converted to Biobase ExpressionSet)
- Bias level grid sigma in {0, 5, 10, 20} with 10 replicates per level

## Outputs

- Per-method, per-sigma, per-replicate cell-type proportion estimates (bulk.props)
- Correlation (R) and RMSD metrics against true.props per method/sigma/replicate
- bisque_bias_simulation_results.csv containing all estimates and R/RMSD metrics

## How to apply

Load adipose snRNA-seq counts with cell-type and individual labels plus true cell-type proportions for overlapping bulk samples, converting both to Biobase ExpressionSet objects (sc.eset and bulk.eset). For each bias level sigma in {0, 5, 10, 20}, draw gene-specific linear coefficient and intercept perturbations from a half-normal distribution and add the resulting linear bias to the bulk expression, generating 10 replicate bulk matrices per sigma level (analogous to SimulateData with fixed cell.types and avg.props). Run BisqueRNA::ReferenceBasedDecomposition on each biased bulk ExpressionSet with the single-nucleus reference using use.overlap=TRUE so gene-specific transformations are learned from overlapping samples, and run comparators CIBERSORT/BSEQ-sc (SVM regression on a single-cell-derived reference) and MuSiC (weighted NNLS) on the identical biased bulk inputs. For each method, sigma, and replicate, correlate bulk.props estimates with true proportions and compute RMSD against true.props, writing all estimates and metrics to bisque_bias_simulation_results.csv. Rationale: differences between single-cell and bulk technologies (e.g., nuclear pre-mRNA in snRNA-seq) may introduce gene-specific biases breaking direct proportionality, and Bisque's learned gene-specific transformations should absorb such bias.

## Related tools

- **BisqueRNA** (Runs ReferenceBasedDecomposition on biased bulk ExpressionSets with the snRNA-seq reference (use.overlap=TRUE) to collect bulk.props estimates) — github.com/cozygene/bisque
- **Biobase** (Wraps bulk and single-nucleus count matrices as ExpressionSet objects (bulk.eset, sc.eset) required by Bisque)
- **CIBERSORT** (Comparator method: SVM-regression decomposition on a single-cell-derived reference applied to the identical biased bulk inputs)
- **BSEQ-sc** (Comparator method: generates a reference profile from single-cell expression data used in the CIBERSORT model)
- **MuSiC** (Comparator method: weighted non-negative least-squares (NNLS) decomposition applied to the same biased bulk inputs)

## Examples

```
res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE); estimates <- res$bulk.props
```

## Evaluation signals

- At the highest bias level (sigma = 20), Bisque attains R ≈ 0.85 and RMSD ≈ 0.07 against true proportions
- Comparator methods (CIBERSORT/BSEQ-sc, MuSiC) degrade relative to Bisque as sigma increases, so Bisque's R/RMSD curves should stay flat or improve relative to comparators
- Each sigma level yields exactly 10 replicate bulk matrices with half-normal-distributed gene-specific coefficient and intercept perturbations
- Results file bisque_bias_simulation_results.csv contains estimates plus R/RMSD for every method × sigma × replicate combination
- At sigma = 0 the perturbation is a no-op, so all methods should recover baseline (unbiased) performance

## Limitations

- If the number of individuals with single-cell data available is small, the reference profile and gene-specific transformations may become unreliable
- A key assumption is that single-cell-based estimates of cell proportions accurately reflect the true proportions; accuracy may decrease if single-cell-captured proportions differ significantly from true physiological distributions
- Caution is advised when there is known significant bias in single-cell measurements, such as severe underrepresentation of a cell type of interest

## Evidence

- [results] Gene-specific bias introduced between simulated bulk and snRNA-seq reference; Bisque robust at R ≈ 0.85 and RMSD ≈ 0.07 at highest bias: [evidence span withheld — anchor status: not_found]
- [intro] Differences between single-cell and bulk technologies introduce gene-specific biases breaking proportionality: "these differences may introduce gene-specific biases that break down the correlation between cell-type-specific and bulk tissue measurements"
- [methods] Reference-based decomposition call with use.overlap: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers=NULL, use.overlap=TRUE)"
- [methods] ExpressionSet conversion required by Bisque: "Bisque requires expression data in the ExpressionSet format from the Biobase package."
- [abstract] Bisque learns gene-specific transformations improving performance under technical variation: "These transformations significantly improve decomposition performance compared to existing methods when there is significant technical variation in the generation of the reference profile and"
- [readme] BisqueRNA installs from CRAN and the two decomposition entry points: "res <- BisqueRNA::ReferenceBasedDecomposition(bulk.eset, sc.eset, markers)"
