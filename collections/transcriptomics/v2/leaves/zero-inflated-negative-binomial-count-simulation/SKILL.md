---
name: zero-inflated-negative-binomial-count-simulation
description: Use when benchmarking DE or gene-set enrichment methods on scRNA-seq
  data and you need null count matrices with realistic zero inflation, or when calibration
  of null p-values across coverage rates must be tested.
when_to_use_negative:
- The goal is power/TPR benchmarking with embedded differential expression or enrichment
  signal, not null calibration (set non-null tau parameters instead).
- The input data is already gene-level summary statistics — simulation of counts is
  unnecessary; proceed directly to CreateiDEAObject and iDEA.fit.
- Benchmarking bulk RNA-seq rather than single-cell data where zero inflation may
  not need ZINB parameterization (the zinbwaveZinger repo also provides a bulk simulation
  script, rnaseqSim.R, for that setting).
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: iDEA
  role: Fits the iDEA model (iDEA.fit) on summary statistics derived from simulated
    null counts and computes calibrated gene-set p-values (iDEA.louis); CreateiDEAObject
    constructs gene sets with coverage-rate filters
  repo: https://github.com/xzhoulab/iDEA
- name: zingeR
  role: zingeRsimulationFunctions in the zinbwaveZinger repo provide the ZINB parameter
    estimation and count-simulation framework used to generate null scRNA-seq datasets
  repo: https://github.com/statOmics/zinbwaveZinger
- name: fGSEA
  role: Competitive GSE method run on the same null summary statistics to assess calibration
    relative to iDEA
- name: CAMERA
  role: Competitive GSE method expected to become increasingly deflated (conservative)
    as coverage rate increases under the null
- name: PAGE
  role: Competitive GSE method run on null summary statistics; expected to yield well-calibrated
    null p-values
- name: GSEA
  role: Competitive GSE method run on null summary statistics; expected to yield well-calibrated
    null p-values
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:c72db3f0c786e9ab7dd7c3ff49ef0298f7b80436e1d778967ca0ce8a6d9c3f22
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
  - zingeR
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# zero-inflated-negative-binomial-count-simulation

## Summary

Simulate null single-cell RNA-seq count matrices under a zero-inflated negative-binomial model with parameters inferred from reference scRNA-seq data, to generate benchmarking scenarios with no embedded DE or gene-set enrichment signal for evaluating p-value calibration of GSE methods.

## When to use

Use when benchmarking DE or gene-set enrichment methods on scRNA-seq data and you need null count matrices with realistic zero inflation, or when calibration of null p-values across coverage rates must be tested.

## When NOT to use

- The goal is power/TPR benchmarking with embedded differential expression or enrichment signal, not null calibration (set non-null tau parameters instead).
- The input data is already gene-level summary statistics — simulation of counts is unnecessary; proceed directly to CreateiDEAObject and iDEA.fit.
- Benchmarking bulk RNA-seq rather than single-cell data where zero inflation may not need ZINB parameterization (the zinbwaveZinger repo also provides a bulk simulation script, rnaseqSim.R, for that setting).

## Inputs

- Reference scRNA-seq count matrix used to infer zero-inflated negative-binomial (ZINB) simulation parameters
- Simulation settings: number of genes and cells (e.g. 10,000 genes x 174 cells), null parameters tau0=-2, tau1=0
- Gene set coverage rates to sweep (e.g. CR = 1%, 2%, 5%, 10%)
- CreateiDEAObject filter thresholds: min_percent_annot (default 0.0025), max_var_beta (default 100)

## Outputs

- Zero-inflated negative-binomial simulated scRNA-seq count matrices with no embedded DE or gene-set enrichment signal
- Synthetic gene sets at specified coverage rates
- Gene-level summary statistics (effect size beta and its variance) per simulation
- Null gene-set p-values per GSE method and coverage rate, with lambda_gc inflation factors and QQ plots of observed vs expected p-values

## How to apply

Estimate ZINB parameters from a reference scRNA-seq dataset using the simulation framework in the statOmics/zinbwaveZinger repository (simulationHelpFunctions_v7_diffInZero.R), which 'has been used to simulate all scRNA-seq datasets' in the zingeR/ZINB-WaVE workflow. Simulate count matrices at the desired dimensions (e.g. 10,000 genes x 174 cells as in the iDEA null-calibration task), with no embedded DE signal by setting the null effect parameters (null tau0=-2, tau1=0) so gene-level summary statistics are pure noise. Construct synthetic gene sets at controlled coverage rates (CR = gene set size / tested genes; e.g. 1%, 2%, 5%, 10%), respecting iDEA's CreateiDEAObject filter thresholds (min_percent_annot default 0.0025, max_var_beta default 100) so gene sets are retained downstream. Derive gene-level effect sizes (beta) and their variances from the simulated counts, fit iDEA with iDEA.fit (em_iter=15, mcmc_iter=1000, fit.tol=1e-5, init_tau=c(-2,0.5)), obtain calibrated p-values via iDEA.louis (Louis method), and compare against competitive GSE methods (fGSEA, PAGE, GSEA, CAMERA). Assess calibration by aggregating null p-values per method and coverage rate, computing the genomic-control inflation factor lambda_gc, and drawing quantile-quantile plots of observed vs expected p-values.

## Related tools

- **iDEA** (Fits the iDEA model (iDEA.fit) on summary statistics derived from simulated null counts and computes calibrated gene-set p-values (iDEA.louis); CreateiDEAObject constructs gene sets with coverage-rate filters) — https://github.com/xzhoulab/iDEA
- **zingeR** (zingeRsimulationFunctions in the zinbwaveZinger repo provide the ZINB parameter estimation and count-simulation framework used to generate null scRNA-seq datasets) — https://github.com/statOmics/zinbwaveZinger
- **fGSEA** (Competitive GSE method run on the same null summary statistics to assess calibration relative to iDEA)
- **CAMERA** (Competitive GSE method expected to become increasingly deflated (conservative) as coverage rate increases under the null)
- **PAGE** (Competitive GSE method run on null summary statistics; expected to yield well-calibrated null p-values)
- **GSEA** (Competitive GSE method run on null summary statistics; expected to yield well-calibrated null p-values)

## Examples

```
devtools::install_github('xzhoulab/iDEA'); library(iDEA); idea <- CreateiDEAObject(summary_data, annotation_data, min_percent_annot=0.0025, max_var_beta=100); idea <- iDEA.fit(idea, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, init_tau=c(-2,0.5)); idea <- iDEA.louis(idea)
```

## Evaluation signals

- Null QQ plots of observed vs expected gene-set p-values fall on the diagonal for well-calibrated methods (iDEA, fGSEA, PAGE, GSEA), reproducing the Fig. 2 pattern.
- Genomic-control inflation factor lambda_gc is near 1 for calibrated methods under the null.
- CAMERA p-values show increasing deflation (conservatism) as coverage rate increases from 1% to 10%; CAMERA is calibrated only at very low coverage rate (1%).
- Simulated counts match reference data quality — the zinbwaveZinger workflow evaluates simulated datasets with the countsimQC package.
- All synthetic gene sets pass CreateiDEAObject filters: coverage rate >= min_percent_annot (0.0025) and gene coefficient variance <= max_var_beta (100).

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets; permutation of gene labels (iDEA.fit.null) is used to build the permuted null.
- The z-score modeling in iDEA effectively assumes the prior distribution of true effect sizes depends on the standard errors and sample size.
- Simulation parameter inference is tied to the zinbwaveZinger framework and reference datasets (e.g. Islam, Trapnell, 10X); simulated-data quality should be verified with countsimQC before benchmarking.

## Evidence

- [readme] The functions used to estimate parameters based on real scRNA-seq data, and to simulate the expression counts can be found in the simulationHelpFunctions_v7_diffInZero.R: [evidence span withheld — anchor status: not_found]
- [readme] This framework has been used to simulate all scRNA-seq datasets.: [evidence span withheld — anchor status: external_doc]
- [methods] min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the number of tested genes. Default value is 0.0025.: "min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the number of tested genes. Default value is 0.0025."
- [methods] iDEA utilizes Louis method to compute calibrated p-values for testing gene set enrichment: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [discussion] GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values: "GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values"
- [intro] incorporating summary statistics from scRNA-seq DE analysis into GSE analysis under the framework of iDEA makes GSE analysis less susceptible to gene-gene correlations and other technical: "incorporating summary statistics from scRNA-seq DE analysis into GSE analysis under the framework of iDEA makes GSE analysis less susceptible to gene-gene correlations and other technical"
