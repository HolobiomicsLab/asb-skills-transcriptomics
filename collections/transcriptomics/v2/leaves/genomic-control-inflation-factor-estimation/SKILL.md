---
name: genomic-control-inflation-factor-estimation
description: Use when benchmarking whether a competitive GSE method (iDEA, fGSEA,
  PAGE, GSEA, CAMERA) yields well-calibrated p-values under null simulations on zero-inflated
  scRNA-seq data.
when_to_use_negative:
- When simulations embed true DE signal or gene-set enrichment (non-null tau1) — lambda_gc
  then reflects real signal, not miscalibration
- When only a handful of gene-set p-values are available; lambda_gc and QQ plots need
  many p-values aggregated across replicates to be informative
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: iDEA
  role: Fits the null iDEA model (iDEA.fit, em_iter=15, mcmc_iter=1000, fit.tol=1e-5,
    init_tau=c(-2,0.5)) and computes calibrated gene-set p-values via iDEA.louis (Louis
    method) for the calibration benchmark
  repo: https://github.com/xzhoulab/iDEA
- name: fGSEA
  role: Competitive GSE method run on the same null summary statistics to compare
    p-value calibration
- name: PAGE
  role: Competitive GSE method run on the same null summary statistics to compare
    p-value calibration
- name: GSEA
  role: Competitive GSE method run on the same null summary statistics to compare
    p-value calibration
- name: CAMERA
  role: Gene-gene correlation-aware GSE method whose p-values become increasingly
    overly conservative as coverage rate increases
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:cad2418e29db0f81ae9ea978be7427a7c25aa8addec5523d307b7680221a3d96
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# genomic-control-inflation-factor-estimation

## Summary

Assess null p-value calibration of gene set enrichment methods by aggregating gene-set p-values per method and coverage rate, computing the genomic-control inflation factor lambda_gc, and drawing observed-vs-expected quantile-quantile plots to detect inflation or over-conservatism.

## When to use

Use when benchmarking whether a competitive GSE method (iDEA, fGSEA, PAGE, GSEA, CAMERA) yields well-calibrated p-values under null simulations on zero-inflated scRNA-seq data.

## When NOT to use

- When simulations embed true DE signal or gene-set enrichment (non-null tau1) — lambda_gc then reflects real signal, not miscalibration
- When only a handful of gene-set p-values are available; lambda_gc and QQ plots need many p-values aggregated across replicates to be informative

## Inputs

- Vectors of gene-set p-values from each competitive GSE method (iDEA via iDEA.louis, fGSEA, PAGE, GSEA, CAMERA) per simulated null replicate
- Coverage rate labels (CR = 1%, 2%, 5%, 10%) for each gene set size configuration
- Gene-level summary statistics (beta and var(beta)) from simulated zero-inflated scRNA-seq counts

## Outputs

- Per-method, per-CR genomic-control inflation factor lambda_gc
- Quantile-quantile plots of observed vs expected gene-set p-values
- Calibration verdict per method (well-calibrated vs inflated/conservative)

## How to apply

Run each GSE method on gene-level summary statistics (effect size beta and its variance) derived from null zero-inflated negative-binomial simulations (e.g. 10,000 genes x 174 cells, no embedded enrichment, tau0=-2, tau1=0), repeated across gene set coverage rates (CR = gene set size / tested genes) of 1%, 2%, 5%, and 10%. Aggregate all gene-set p-values per method and per CR, then compute the genomic-control inflation factor lambda_gc as a scalar summary of p-value dispersion relative to the expected uniform null. Draw quantile-quantile plots of observed versus expected p-values for each method-CR combination. A well-calibrated method shows lambda_gc near 1 and QQ points lying on the diagonal; systematic deflation (points below the diagonal) indicates over-conservative p-values, the failure mode CAMERA exhibits as CR increases.

## Related tools

- **iDEA** (Fits the null iDEA model (iDEA.fit, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, init_tau=c(-2,0.5)) and computes calibrated gene-set p-values via iDEA.louis (Louis method) for the calibration benchmark) — https://github.com/xzhoulab/iDEA
- **fGSEA** (Competitive GSE method run on the same null summary statistics to compare p-value calibration)
- **PAGE** (Competitive GSE method run on the same null summary statistics to compare p-value calibration)
- **GSEA** (Competitive GSE method run on the same null summary statistics to compare p-value calibration)
- **CAMERA** (Gene-gene correlation-aware GSE method whose p-values become increasingly overly conservative as coverage rate increases)

## Examples

```
library(iDEA); idea_obj <- CreateiDEAObject(beta, var_beta, gene_names, num_nonsig); idea_obj <- iDEA.fit(idea_obj, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, init_tau=c(-2,0.5)); idea_obj <- iDEA.louis(idea_obj)
```

## Evaluation signals

- lambda_gc close to 1 for iDEA, fGSEA, PAGE, and GSEA under null simulations
- QQ points of observed vs expected p-values lie on the diagonal for well-calibrated methods
- CAMERA p-values show increasing deflation (sub-diagonal QQ points, lambda_gc < 1) as CR rises from 1% to 10%, reproducing the Fig. 2 calibration pattern
- iDEA p-values computed via iDEA.louis (Louis method) remain calibrated even at CR = 10%
- Null simulations must contain no embedded signal (tau0=-2, tau1=0) so any inflation is attributable to miscalibration

## Limitations

- The simulation setting (10,000 genes, 174 cells, ZINB parameters inferred from reference scRNA-seq data) may not generalize to other scRNA-seq platforms or coverage regimes
- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets

## Evidence

- [results] CAMERA p-values calibrated only at very low coverage rate: [evidence span withheld — anchor status: not_found]
- [methods] Louis method calibrated p-values: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [methods] coverage rate threshold: "min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the  number of tested genes. Default value is 0.0025."
- [discussion] calibrated p-values for non-correlation-aware methods: "GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values"
- [readme] README install command: "devtools::install_github('xzhoulab/iDEA')"
