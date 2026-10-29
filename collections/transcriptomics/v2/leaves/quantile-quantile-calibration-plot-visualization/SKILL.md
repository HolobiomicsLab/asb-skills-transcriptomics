---
name: quantile-quantile-calibration-plot-visualization
description: 'Use when validating p-value calibration of gene-set enrichment or DE
  methods under a permuted/simulated null: pooled null p-values per method must be
  inspected via lambda_gc and QQ plots before trusting real-data calls.'
when_to_use_negative:
- The p-values come from simulations with embedded DE signal or gene-set enrichment
  (non-null tau1) — QQ calibration plots only assess null behavior, not power
- The goal is FDR control assessment across overlapping gene sets, where the permuted-null
  pipeline (iDEA.fit.null + iDEA.FDR) is the appropriate mechanism instead
- Input is per-gene DE p-value calibration rather than aggregated gene-set p-values;
  the QQ stratification here is built around gene-set-level tests at defined coverage
  rates
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: iDEA
  role: Source of calibrated gene-set p-values via iDEA.louis (Louis method) that
    are then aggregated and plotted for calibration
  repo: https://github.com/xzhoulab/iDEA
- name: CAMERA
  role: Competitive GSE method whose null p-values are expected to become increasingly
    deflated as coverage rate increases
- name: fGSEA
  role: Competitive GSE method whose null p-values are expected to be well-calibrated
    in the QQ comparison
- name: PAGE
  role: Competitive GSE method whose null p-values are expected to be well-calibrated
    in the QQ comparison
- name: GSEA
  role: Competitive GSE method whose null p-values are expected to be well-calibrated
    in the QQ comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:738def420a84bf0b85506567de5679340e131e2046762221ef2692fa1b464346
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
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# quantile-quantile-calibration-plot-visualization

## Summary

A calibration-check skill that aggregates gene-set p-values from null simulations, computes the genomic-control inflation factor lambda_gc, and plots observed vs expected p-value quantiles to verify whether a GSE method (iDEA, fGSEA, PAGE, GSEA, CAMERA) produces well-calibrated p-values.

## When to use

Use when validating p-value calibration of gene-set enrichment or DE methods under a permuted/simulated null: pooled null p-values per method must be inspected via lambda_gc and QQ plots before trusting real-data calls.

## When NOT to use

- The p-values come from simulations with embedded DE signal or gene-set enrichment (non-null tau1) — QQ calibration plots only assess null behavior, not power
- The goal is FDR control assessment across overlapping gene sets, where the permuted-null pipeline (iDEA.fit.null + iDEA.FDR) is the appropriate mechanism instead
- Input is per-gene DE p-value calibration rather than aggregated gene-set p-values; the QQ stratification here is built around gene-set-level tests at defined coverage rates

## Inputs

- Per-method gene-set p-value vectors from null simulations (iDEA via iDEA.louis; fGSEA, PAGE, GSEA, CAMERA on the same null summary statistics)
- Coverage-rate condition labels (CR = 1%, 2%, 5%, 10%) to stratify the plots
- Gene-level summary statistics (effect size beta and its variance) derived from simulated zero-inflated scRNA-seq counts

## Outputs

- Quantile-quantile plots of observed vs expected p-values, one per method per coverage rate
- Genomic-control inflation factor lambda_gc per method and per CR
- Calibration verdict: lambda_gc near 1 / QQ points on the diagonal vs deflated (over-conservative) p-values

## How to apply

Run each competitive GSE method on null summary statistics (e.g. zero-inflated negative-binomial scRNA-seq counts with 10,000 genes and 174 cells, null tau0=-2, tau1=0) at several coverage rates such as CR = 1%, 2%, 5%, 10%. For iDEA specifically, compute gene-set p-values with iDEA.louis, which applies the Louis method to the iDEA.fit model fit (em_iter=15, mcmc_iter=1000, fit.tol=1e-5, init_tau=c(-2,0.5)). Aggregate the null gene-set p-values per method and per CR, then compute the genomic-control inflation factor lambda_gc (ratio of observed to expected median chi-square statistic). Draw quantile-quantile plots of observed vs expected -log10(p) with the diagonal as reference; calibrated methods should have lambda_gc near 1 with points on the diagonal. This reproduces the Fig. 2 pattern: CAMERA p-values become increasingly overly conservative (deflated) as CR increases, while iDEA, fGSEA, PAGE, and GSEA stay calibrated because they do not explicitly model gene-gene correlation in this null setting.

## Related tools

- **iDEA** (Source of calibrated gene-set p-values via iDEA.louis (Louis method) that are then aggregated and plotted for calibration) — https://github.com/xzhoulab/iDEA
- **CAMERA** (Competitive GSE method whose null p-values are expected to become increasingly deflated as coverage rate increases)
- **fGSEA** (Competitive GSE method whose null p-values are expected to be well-calibrated in the QQ comparison)
- **PAGE** (Competitive GSE method whose null p-values are expected to be well-calibrated in the QQ comparison)
- **GSEA** (Competitive GSE method whose null p-values are expected to be well-calibrated in the QQ comparison)

## Examples

```
qqplot(-log10(ppoints(length(p))), -log10(p)); abline(0,1); lambda_gc <- median(qchisq(1-p, 1)) / qchisq(0.5, 1)
```

## Evaluation signals

- lambda_gc for iDEA, fGSEA, PAGE, and GSEA should be near 1 across all coverage rates (1%, 2%, 5%, 10%)
- QQ points should lie on the diagonal for well-calibrated methods; systematic bow above/below the diagonal indicates deflation/inflation
- CAMERA lambda_gc should show a monotone trend toward greater over-conservatism (deflated p-values) as CR rises from 1% to 10%, with calibration only at very low CR (1%)
- The pattern across methods must match the Fig. 2 calibration layout from the paper
- Identical null summary statistics must be fed to every compared method within each CR condition, so QQ differences reflect method calibration only

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets, which is handled separately via permutation of gene labels
- A QQ plot from a null run only checks calibration; it does not measure power or FDR behavior under signal
- iDEA's z-score modeling effectively assumes the prior distribution of true effect sizes depends on standard errors and sample size, which could bias calibration in some settings
- Compared tools lack documented version numbers, so exact calibration values may not be bit-reproducible

## Evidence

- [discussion] GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values: "GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values"
- [methods] iDEA utilizes Louis method to compute calibrated p-values for testing gene set enrichment: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [results] In null simulations, iDEA yields well-calibrated p-values, while CAMERA p-values are calibrated only at very low coverage rate (1%): [evidence span withheld — anchor status: not_found]
- [readme] This framework has been used to simulate all scRNA-seq datasets.: [evidence span withheld — anchor status: not_found]
