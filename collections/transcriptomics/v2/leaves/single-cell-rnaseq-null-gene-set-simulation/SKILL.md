---
name: single-cell-rnaseq-null-gene-set-simulation
description: Use when benchmarking whether gene set enrichment methods (iDEA, fGSEA,
  PAGE, GSEA, CAMERA) produce well-calibrated null p-values on zero-inflated scRNA-seq-like
  data, or when studying how gene set size / coverage rate affects calibration.
when_to_use_negative:
- The goal is to detect real enrichment or DE signal — this skill simulates explicit
  null data with no embedded signal and cannot measure power or ranking of true positives.
- Data are bulk RNA-seq without zero inflation — the zero-inflated negative-binomial
  simulation framework is designed for scRNA-seq dropout characteristics.
- Assessment of sensitivity to gene set overlap/non-independence among tested gene
  sets is the target; this null simulation varies coverage rate for synthetic sets
  instead.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: iDEA
  role: Fits the null hierarchical Bayesian model on simulated summary statistics
    (iDEA.fit with init_tau=c(-2,0.5)) and produces calibrated gene-set p-values via
    iDEA.louis (Louis method); CreateiDEAObject framework defines gene-set coverage-rate
    filtering
  repo: https://github.com/xzhoulab/iDEA
- name: fGSEA
  role: Competitive GSE method run on the same null summary statistics per coverage
    rate to compare p-value calibration
- name: PAGE
  role: Competitive GSE method run on the same null summary statistics per coverage
    rate to compare p-value calibration
- name: GSEA
  role: Competitive GSE method run on the same null summary statistics per coverage
    rate to compare p-value calibration
- name: CAMERA
  role: Competitive GSE method run on the same null summary statistics; expected to
    become increasingly overly conservative as coverage rate increases
- name: zingeR
  role: Underlying zero-inflated scRNA-seq simulation framework (zinbwaveZinger) used
    to estimate parameters from real data and simulate null counts
  repo: https://github.com/statOmics/zinbwaveZinger
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:864991113f501da572b871c109689f57965a3e0d05a9bb9b4a928d5a39111f5d
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-rnaseq-null-gene-set-simulation

## Summary

Simulate zero-inflated negative-binomial scRNA-seq count matrices under an explicit null (no DE signal, null tau0=-2, tau1=0), construct synthetic gene sets at controlled coverage rates, and use the resulting null gene-set p-values to assess p-value calibration (via lambda_gc and QQ plots) of competitive gene set enrichment methods.

## When to use

Use when benchmarking whether gene set enrichment methods (iDEA, fGSEA, PAGE, GSEA, CAMERA) produce well-calibrated null p-values on zero-inflated scRNA-seq-like data, or when studying how gene set size / coverage rate affects calibration.

## When NOT to use

- The goal is to detect real enrichment or DE signal — this skill simulates explicit null data with no embedded signal and cannot measure power or ranking of true positives.
- Data are bulk RNA-seq without zero inflation — the zero-inflated negative-binomial simulation framework is designed for scRNA-seq dropout characteristics.
- Assessment of sensitivity to gene set overlap/non-independence among tested gene sets is the target; this null simulation varies coverage rate for synthetic sets instead.

## Inputs

- Reference scRNA-seq count matrix (e.g. 174 cells) for ZINB parameter inference
- Null simulation settings: no DE signal (tau0=-2, tau1=0)
- Gene annotation mapping for constructing synthetic gene sets at CR = 1%, 2%, 5%, 10%
- Gene-level summary statistics (effect size beta and variance) derived from simulated counts

## Outputs

- Null gene-set p-value distributions per method (iDEA, fGSEA, PAGE, GSEA, CAMERA) and per coverage rate
- Genomic-control inflation factor lambda_gc per method per CR
- Quantile-quantile plots of observed vs expected p-values (Fig. 2 calibration pattern)

## How to apply

Simulate zero-inflated negative-binomial single-cell count matrices (e.g. 10,000 genes x 174 cells) by inferring ZINB parameters from a reference scRNA-seq dataset with no embedded DE signal or enrichment (null tau0=-2, tau1=0), reusing the zinbwaveZinger simulation framework. Construct synthetic gene sets at coverage rates of 1%, 2%, 5%, and 10% (CR = gene set size / tested genes) using the CreateiDEAObject annotation framework (min_percent_annot default 0.0025, max_var_beta 100). Derive gene-level summary statistics (effect size beta and its variance) from the simulated counts, fit the iDEA model with iDEA.fit (em_iter=15, mcmc_iter=1000, fit.tol=1e-5, init_tau=c(-2,0.5)) and obtain calibrated p-values via iDEA.louis (Louis method); run fGSEA, PAGE, GSEA, and CAMERA on the same null summary statistics per CR. Aggregate gene-set p-values per method and CR, compute the genomic-control inflation factor lambda_gc, and draw quantile-quantile plots of observed vs expected p-values: well-calibrated methods show lambda_gc near 1 with QQ points on the diagonal, whereas CAMERA p-values become increasingly deflated (overly conservative) as coverage rate increases.

## Related tools

- **iDEA** (Fits the null hierarchical Bayesian model on simulated summary statistics (iDEA.fit with init_tau=c(-2,0.5)) and produces calibrated gene-set p-values via iDEA.louis (Louis method); CreateiDEAObject framework defines gene-set coverage-rate filtering) — https://github.com/xzhoulab/iDEA
- **fGSEA** (Competitive GSE method run on the same null summary statistics per coverage rate to compare p-value calibration)
- **PAGE** (Competitive GSE method run on the same null summary statistics per coverage rate to compare p-value calibration)
- **GSEA** (Competitive GSE method run on the same null summary statistics per coverage rate to compare p-value calibration)
- **CAMERA** (Competitive GSE method run on the same null summary statistics; expected to become increasingly overly conservative as coverage rate increases)
- **zingeR** (Underlying zero-inflated scRNA-seq simulation framework (zinbwaveZinger) used to estimate parameters from real data and simulate null counts) — https://github.com/statOmics/zinbwaveZinger

## Examples

```
library(iDEA); idea.obj <- CreateiDEAObject(beta, se, gene_name, annot, min_percent_annot=0.0025, max_var_beta=100); idea.obj <- iDEA.fit(idea.obj, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, init_tau=c(-2,0.5)); idea.obj <- iDEA.louis(idea.obj)
```

## Evaluation signals

- iDEA, fGSEA, PAGE, and GSEA null p-values should be well calibrated: lambda_gc near 1 with QQ points on the diagonal at all coverage rates (1%, 2%, 5%, 10%).
- CAMERA null p-values should show increasing deflation (overly conservative) as coverage rate increases from 1% to 10%, reproducing the Fig. 2 pattern.
- Simulated counts must reproduce reference scRNA-seq characteristics (zero inflation, 10,000 genes, 174 cells) with no embedded DE or enrichment signal (tau1=0).
- Synthetic gene sets must satisfy the CreateiDEAObject filters: min_percent_annot (default 0.0025) and max_var_beta (default 100).
- Aggregated null p-value distributions should be uniform under the null for calibrated methods.

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets; permutation-based nulls (iDEA.fit.null) and FDR estimation address this but the calibration simulation itself uses synthetic sets.
- The z-score modeling effectively assumes the prior distribution of true effect sizes depends on standard errors and sample size, which can affect transferability of calibration conclusions.
- No software version numbers for the compared tools are given in the discussion, complicating exact reproduction of the calibration benchmark.

## Evidence

- [methods] iDEA utilizes Louis method to compute calibrated p-values for testing gene set enrichment: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [discussion] GSE methods not accounting for gene-gene correlation provide more calibrated p-values: "GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values"
- [methods] min_percent_annot is the coverage rate threshold in CreateiDEAObject: "min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the  number of tested genes. Default value is 0.0025."
- [methods] max_var_beta cutoff filters genes by variance of the coefficient: "max_var_beta: the cutoff of the variance of the coefficient of genes. Genes with variance smaller than 'max_var_beta' are maintained. Default is 100"
- [readme] zingeR simulation framework used for simulating scRNA-seq datasets: [evidence span withheld — anchor status: external_doc]
- [readme] iDEA install command from the package README: "devtools::install_github('xzhoulab/iDEA')"
