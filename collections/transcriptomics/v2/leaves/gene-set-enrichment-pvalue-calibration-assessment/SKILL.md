---
name: gene-set-enrichment-pvalue-calibration-assessment
description: Use when benchmarking GSE methods (e.g., iDEA, fGSEA, PAGE, GSEA, CAMERA)
  on scRNA-seq data and you must verify null p-value calibration rather than power,
  especially across varying gene set coverage rates.
when_to_use_negative:
- The simulation or dataset embeds true DE signal or gene-set enrichment (non-null
  tau1), since calibration assessment requires a pure null
- You need a power/FDR-TPR comparison of DE methods rather than GSE p-value calibration
  under the null
- Gene-level inputs are raw counts without per-gene effect-size beta and variance
  summary statistics required by iDEA and the competitive GSE methods
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: iDEA
  role: Fits the hierarchical Bayesian joint DE/GSE model on null summary statistics
    and produces calibrated gene-set p-values via iDEA.louis (Louis method)
  repo: https://github.com/xzhoulab/iDEA
- name: fGSEA
  role: Competitive GSE method run on the same null summary statistics as a calibration
    benchmark
- name: PAGE
  role: Competitive GSE method run on the same null summary statistics as a calibration
    benchmark
- name: GSEA
  role: Competitive GSE method run on the same null summary statistics as a calibration
    benchmark
- name: CAMERA
  role: Competitive GSE method benchmarked for calibration; expected to become increasingly
    conservative (deflated p-values) as coverage rate increases
- name: zingeR
  role: Simulation framework (zinbwaveZinger) used to estimate ZINB parameters from
    real scRNA-seq data and simulate null expression counts
  repo: https://github.com/statOmics/zinbwaveZinger
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:af9df921e6c853a98deb6ac143ea77e3adc191b13d5dafd0710f9520fcac5a3c
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# gene-set-enrichment-pvalue-calibration-assessment

## Summary

Evaluates whether competitive gene set enrichment (GSE) methods yield well-calibrated null p-values on zero-inflated scRNA-seq summary statistics, by simulating ZINB null data, sweeping gene set coverage rates, and checking genomic-control inflation factors and QQ plots per method.

## When to use

Use when benchmarking GSE methods (e.g., iDEA, fGSEA, PAGE, GSEA, CAMERA) on scRNA-seq data and you must verify null p-value calibration rather than power, especially across varying gene set coverage rates.

## When NOT to use

- The simulation or dataset embeds true DE signal or gene-set enrichment (non-null tau1), since calibration assessment requires a pure null
- You need a power/FDR-TPR comparison of DE methods rather than GSE p-value calibration under the null
- Gene-level inputs are raw counts without per-gene effect-size beta and variance summary statistics required by iDEA and the competitive GSE methods

## Inputs

- Zero-inflated negative-binomial scRNA-seq count matrix simulated under the null (10,000 genes x 174 cells, tau0=-2, tau1=0)
- Gene-level DE summary statistics (effect size beta and its variance) derived from simulated counts
- Synthetic gene set annotations at coverage rates of 1%, 2%, 5%, and 10%

## Outputs

- Per-method, per-coverage-rate gene-set null p-values (iDEA via iDEA.louis; fGSEA, PAGE, GSEA, CAMERA outputs)
- Genomic-control inflation factor lambda_gc per method and CR
- QQ plots of observed vs expected p-values per method, reproducing the Fig. 2 calibration pattern

## How to apply

Simulate zero-inflated negative-binomial single-cell count matrices (10,000 genes, 174 cells) with ZINB parameters inferred from reference scRNA-seq data, embedding no DE signal or enrichment (null tau0=-2, tau1=0). Build synthetic gene sets at coverage rates (CR = gene set size / tested genes) of 1%, 2%, 5%, and 10%, using CreateiDEAObject's annotation framework with min_percent_annot (default 0.0025) and max_var_beta (default 100) as filtering thresholds. Derive gene-level effect sizes (beta) and variances from the simulated counts, fit the iDEA model with iDEA.fit (em_iter=15, mcmc_iter=1000, fit.tol=1e-5, init_tau=c(-2,0.5)), and obtain calibrated gene-set p-values via iDEA.louis (Louis method). Run fGSEA, PAGE, GSEA, and CAMERA on the same null summary statistics at each CR. Aggregate gene-set p-values per method and CR, compute the genomic-control inflation factor lambda_gc, and draw QQ plots of observed vs expected p-values; the rationale is that summary-statistic-based GSE is less susceptible to gene-gene correlations and dropout, so methods ignoring correlation (CAMERA) may mis-calibrate as CR grows.

## Related tools

- **iDEA** (Fits the hierarchical Bayesian joint DE/GSE model on null summary statistics and produces calibrated gene-set p-values via iDEA.louis (Louis method)) — https://github.com/xzhoulab/iDEA
- **fGSEA** (Competitive GSE method run on the same null summary statistics as a calibration benchmark)
- **PAGE** (Competitive GSE method run on the same null summary statistics as a calibration benchmark)
- **GSEA** (Competitive GSE method run on the same null summary statistics as a calibration benchmark)
- **CAMERA** (Competitive GSE method benchmarked for calibration; expected to become increasingly conservative (deflated p-values) as coverage rate increases)
- **zingeR** (Simulation framework (zinbwaveZinger) used to estimate ZINB parameters from real scRNA-seq data and simulate null expression counts) — https://github.com/statOmics/zinbwaveZinger

## Examples

```
install.packages('devtools'); devtools::install_github('xzhoulab/iDEA'); library(iDEA)
```

## Evaluation signals

- iDEA, fGSEA, PAGE, and GSEA null p-values show lambda_gc near 1 with QQ points on the diagonal at every coverage rate (1%, 2%, 5%, 10%)
- CAMERA p-values become increasingly deflated (conservative) as coverage rate rises from 1% to 10%, reproducing the Fig. 2 pattern
- Simulated count matrix matches the null ZINB specification: 10,000 genes, 174 cells, tau0=-2, tau1=0, with no embedded DE or enrichment signal
- Gene set construction respects the CreateiDEAObject filters (min_percent_annot 0.0025, max_var_beta 100) and yields the intended coverage rates
- iDEA model converges within the specified settings (em_iter=15, mcmc_iter=1000, fit.tol=1e-5) and iDEA.louis returns calibrated p-values for each gene set

## Limitations

- CAMERA's calibration depends on coverage rate: it is well calibrated only at very low CR (~1%) and becomes overly conservative as CR increases, so a single-CR check can mask mis-calibration
- iDEA does not explicitly account for gene set overlap, which may cause non-independence among gene sets; FDR is instead estimated from a permuted null (iDEA.fit.null / iDEA.FDR)
- Z-score modeling effectively assumes the prior distribution of true effect sizes depends on the standard errors and sample size, which can affect calibration when DE is run on inferred cell types with artificially small standard errors

## Evidence

- [methods] iDEA utilizes Louis method to compute calibrated p-values for testing gene set enrichment: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [methods] min_percent_annot defines the coverage-rate threshold used in gene set filtering: "min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the  number of tested genes. Default value is 0.0025."
- [discussion] Discussion states that GSE methods ignoring gene-gene correlation give more calibrated p-values, contrasting CAMERA's behavior: "GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values"
- [readme] zinbwaveZinger README locates the simulation functions used to estimate parameters and simulate null counts: [evidence span withheld — anchor status: external_doc]
- [intro] Summary-statistic-based GSE framing motivates the null-calibration design on dropout-prone scRNA-seq data: "incorporating summary statistics from scRNA-seq DE analysis into GSE analysis under the framework of iDEA makes GSE analysis less susceptible to gene-gene correlations"
