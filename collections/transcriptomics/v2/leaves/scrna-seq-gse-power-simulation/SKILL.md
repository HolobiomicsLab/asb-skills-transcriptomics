---
name: scrna-seq-gse-power-simulation
description: Use when benchmarking GSE method power on simulated DE summary statistics
  (beta, beta_var), e.g. to quantify how much more iDEA is than fGSEA, CAMERA, PAGE
  or GSEA at fixed FDR in scRNA-seq settings.
when_to_use_negative:
- Input is real single-cell count data requiring DE calling first — this skill operates
  on pre-computed DE summary statistics from simulated or real replicates
- Goal is to evaluate DE-gene detection power (iDEA vs MAST/SCDE/zingeR) rather than
  gene set enrichment power
- The enrichment scenario differs from the baseline (e.g. different tau1 or coverage
  rate) and published baseline power values are used as validation targets without
  adjusting expectations
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: iDEA
  role: Fits the integrative DE+GSE model on simulated summary statistics (iDEA.fit)
    and computes calibrated gene set p-values (iDEA.louis); the method whose power
    is being benchmarked
  repo: https://github.com/xzhoulab/iDEA
- name: fGSEA
  role: Competing GSE method applied to the identical simulated summary statistics
    for power comparison
- name: CAMERA
  role: Competing competitive-test GSE method run per replicate for the power benchmark
- name: PAGE
  role: Competing competitive-test GSE method run per replicate for the power benchmark
- name: GSEA
  role: Competing GSE method run per replicate for the power benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:b6aa10f82d5c3bb3a6b43ffe160f771d97eee8908940ace8e449ad284378f895
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CAMERA
  - GSEA
  - PAGE
  - fGSEA
  - iDEA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# scRNA-seq-gse-power-simulation

## Summary

Simulation-based power benchmarking of gene set enrichment (GSE) methods for scRNA-seq: generate replicates of gene-level DE summary statistics under an enrichment alternative (tau0=-2, tau1=0.5, CR=10%) and compare iDEA against fGSEA, CAMERA, PAGE and GSEA at a fixed 5% FDR.

## When to use

Use when benchmarking GSE method power on simulated DE summary statistics (beta, beta_var), e.g. to quantify how much more powerful iDEA is than fGSEA, CAMERA, PAGE or GSEA at fixed FDR in scRNA-seq settings.

## When NOT to use

- Input is real single-cell count data requiring DE calling first — this skill operates on pre-computed DE summary statistics from simulated or real replicates
- Goal is to evaluate DE-gene detection power (iDEA vs MAST/SCDE/zingeR) rather than gene set enrichment power
- The enrichment scenario differs from the baseline (e.g. different tau1 or coverage rate) and published baseline power values are used as validation targets without adjusting expectations

## Inputs

- 10,000 simulated replicates of gene-level DE summary statistics (beta, beta_var) with tau0=-2, tau1=0.5, CR=10% (1,000 enriched / 9,000 null replicates)
- Gene set annotations (e.g. humanGeneSets.RData / MSigDB-derived sets from the iDEA package)

## Outputs

- power_comparison_fdr5.csv: empirical power at fixed FDR 5% per method (iDEA, fGSEA, CAMERA, PAGE, GSEA)
- Empirical false-positive rates from the 9,000 null replicates per method

## How to apply

Simulate 10,000 replicates of gene-level DE summary statistics under the baseline enrichment setting tau0=-2, tau1=0.5 with 10% gene set coverage rate (CR), yielding 1,000 enriched and 9,000 null replicates. For each replicate, fit iDEA via iDEA.fit with init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, modelVariant=FALSE (z-score modeling), then compute Louis-method-calibrated gene set p-values with iDEA.louis. Apply fGSEA, CAMERA, PAGE and GSEA to the identical simulated summary statistics of every replicate. Call gene sets significant at fixed FDR 5% per method and replicate; empirical power is the detection rate among the 1,000 enriched replicates, while the 9,000 null replicates verify false-positive control. Tabulate the five powers and expect iDEA 98%, fGSEA 26%, CAMERA 0%, PAGE 8%, GSEA 26% within Monte Carlo error, matching the paper's baseline scenario (Fig. 3a).

## Related tools

- **iDEA** (Fits the integrative DE+GSE model on simulated summary statistics (iDEA.fit) and computes calibrated gene set p-values (iDEA.louis); the method whose power is being benchmarked) — https://github.com/xzhoulab/iDEA
- **fGSEA** (Competing GSE method applied to the identical simulated summary statistics for power comparison)
- **CAMERA** (Competing competitive-test GSE method run per replicate for the power benchmark)
- **PAGE** (Competing competitive-test GSE method run per replicate for the power benchmark)
- **GSEA** (Competing GSE method run per replicate for the power benchmark)

## Examples

```
fit0 <- iDEA.fit(idea, init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, modelVariant=FALSE); fit1 <- iDEA.louis(fit0)
```

## Evaluation signals

- Powers at FDR 5% reproduce the reported baseline values within Monte Carlo error: iDEA 98%, fGSEA 26%, CAMERA 0%, PAGE 8%, GSEA 26% (Fig. 3a)
- Empirical false-positive rate across the 9,000 null replicates is controlled near the nominal 5% FDR for each method
- power_comparison_fdr5.csv contains exactly five power estimates, one per method
- iDEA fits converge (fit.tol=1e-5 within em_iter=15 / mcmc_iter=1000) and each replicate has at least min_degene=5 detected DE genes
- Power ratio iDEA vs best competitor is close to the reported up to five-fold gain

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets; permutation-based null (iDEA.fit.null / iDEA.FDR) is used to address this
- Z-score modeling effectively assumes the prior distribution of true effect sizes depends on the standard errors and hence sample size
- Power values are scenario-specific; they must be recomputed (not reused) for other tau1/CR settings, and Monte Carlo error bounds apply

## Evidence

- [abstract] iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods: "iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods"
- [methods] iDEA Louis p-values: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [methods] iDEA.fit function: "The function iDEA.fit fits the iDEA model on the input data."
- [readme] iDEA installation: "devtools::install_github('xzhoulab/iDEA')"
