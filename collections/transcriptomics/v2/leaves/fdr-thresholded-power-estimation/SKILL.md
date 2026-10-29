---
name: fdr-thresholded-power-estimation
description: Use when simulations provide a known set of true DE genes and you must
  quantify detection power at a controlled FDR level (e.g., true FDR 5%) and compare
  methods.
when_to_use_negative:
- Applied to real datasets where the true DE gene set is unknown — power cannot be
  computed without simulated ground truth
- When FDR calibration itself is the object of study rather than power at a fixed
  FDR threshold
- When gene sets or annotations are absent, since iDEA.fit requires gene set annotations
  to borrow information across genes
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: iDEA
  role: Fits the hierarchical model on DE summary statistics and augments gene calls
    via iDEA.fit and iDEA.BMA to raise power at FDR 5%
  repo: https://github.com/xzhoulab/iDEA
- name: MAST
  role: Standalone DE method producing beta/variance summary statistics used as the
    power baseline
- name: edgeR
  role: Standalone DE method (with ZINB-WaVE weighting) producing summary statistics
    used as the power baseline
- name: zingeR
  role: Standalone DE method producing summary statistics used as the power baseline
  repo: https://github.com/statOmics/zinbwaveZinger
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:5db9dc2d575af346c6e854f90e363f7953f58df055fe28d308bfaf8b7659a461
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CreateiDEAObject
  - MAST
  - edgeR
  - iDEA
  - iDEA.BMA
  - iDEA.fit
  - zingeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# fdr-thresholded-power-estimation

## Summary

Empirical DE power estimation protocol that compares the fraction of known simulated true DE genes recovered at a fixed 5% FDR threshold, used in the iDEA paper to quantify power gains of iDEA over standalone MAST, edgeR, and zingeR.

## When to use

Use when simulations provide a known set of true DE genes and you must quantify detection power at a controlled FDR level (e.g., true FDR 5%) and compare methods.

## When NOT to use

- Applied to real datasets where the true DE gene set is unknown — power cannot be computed without simulated ground truth
- When FDR calibration itself is the object of study rather than power at a fixed FDR threshold
- When gene sets or annotations are absent, since iDEA.fit requires gene set annotations to borrow information across genes

## Inputs

- Simulated single-cell expression counts with a known ground-truth set of DE genes (e.g., tau1=5, CR=10%)
- Gene-level summary statistics (beta coefficient and variance) from standalone DE methods MAST, edgeR/ZINB-WaVE, and zingeR
- Gene set annotation objects (e.g., humanGeneSets.RData) for iDEA fitting

## Outputs

- Empirical DE power estimates at true FDR 5% for each standalone and iDEA-augmented method
- Jaccard index of top-1500 DE gene lists between methods (before and after iDEA augmentation)

## How to apply

Simulate single-cell expression data with known DE signals (e.g., coverage rate CR=0.10, gene set enrichment parameters tau0=-2 and tau1=5), then run each standalone DE method (MAST, edgeR with ZINB-WaVE weighting, zingeR) to produce gene-level beta coefficients and variances. Build iDEA objects via CreateiDEAObject with max_var_beta=100 and min_precent_annot=0.0025, fit the model with iDEA.fit (fit_noGS=FALSE, init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, modelVariant=FALSE), and aggregate DE evidence with iDEA.BMA. Call genes significant at the estimated FDR threshold of 5% and compute power as the fraction of simulated true DE genes recovered, comparing standalone versus iDEA-augmented calls. Complement the power estimate with the Jaccard index of top-1500 DE gene lists across methods to measure consistency gains.

## Related tools

- **iDEA** (Fits the hierarchical model on DE summary statistics and augments gene calls via iDEA.fit and iDEA.BMA to raise power at FDR 5%) — https://github.com/xzhoulab/iDEA
- **MAST** (Standalone DE method producing beta/variance summary statistics used as the power baseline)
- **edgeR** (Standalone DE method (with ZINB-WaVE weighting) producing summary statistics used as the power baseline)
- **zingeR** (Standalone DE method producing summary statistics used as the power baseline) — https://github.com/statOmics/zinbwaveZinger

## Examples

```
idea_obj <- CreateiDEAObject(summary_stats, gene_sets, max_var_beta=100, min_precent_annot=0.0025); idea_obj <- iDEA.fit(idea_obj, fit_noGS=FALSE, init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, modelVariant=FALSE); idea_obj <- iDEA.BMA(idea_obj)
```

## Evaluation signals

- Standalone power at true FDR 5% should reproduce ~65% (zingeR), ~52% (MAST), ~67% (edgeR) at tau1=5, CR=10%
- iDEA-augmented power should reproduce ~81% (zingeR), ~61% (MAST), ~83% (edgeR) at true FDR 5%
- Top-1500 DE gene list Jaccard index should increase from ~0.59 to ~0.77 after applying iDEA
- Estimated FDR from significant calls should be controlled at approximately 5%

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets
- Z-score modeling effectively assumes the prior distribution of true effect sizes depends on the standard errors and sample size
- DE analysis on inferred cell types may yield inflated test statistics with artificially smaller standard errors

## Evidence

- [abstract] power values: "iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods"
- [methods] CreateiDEAObject parameters: "max_var_beta: the cutoff of the variance of the coefficient of genes. Genes with variance smaller than 'max_var_beta' are maintained. Default is 100"
- [methods] iDEA.BMA aggregation: "we developed a Bayesian model averaging (BMA) approach to aggregate DE evidence for any given genes across all available gene sets"
- [readme] zingeR simulation framework: [evidence span withheld — anchor status: external_doc]
