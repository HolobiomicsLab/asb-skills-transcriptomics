---
name: permutation-based-fdr-estimation
description: Use when testing many overlapping gene sets from summary-statistics-based
  enrichment (e.g., iDEA on scRNA-seq DE z-scores) and you need calibrated FDR estimates
  rather than naive p-value thresholds that ignore gene set non-independence.
when_to_use_negative:
- Gene sets being tested are non-overlapping and small in number, so test non-independence
  from gene set overlap is not a concern and simpler p-value adjustment suffices
- Input is individual-level scRNA-seq counts rather than per-gene summary statistics
  — iDEA requires beta/beta_var from an upstream DE method such as zingeR-DESeq2,
  MAST, or SCDE
- Gene sets with coverage rate (annotated genes / tested genes) below the min_precent_annot
  threshold of 0.0025 — these are filtered out before fitting
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: iDEA
  role: Provides iDEA.fit.null to fit the model with permuted gene labels (constructing
    the null) and iDEA.FDR to estimate each gene set's FDR against that null
  repo: https://github.com/xzhoulab/iDEA
- name: zingeR
  role: Upstream zingeR-DESeq2 produces the per-gene LogFC and p-value summary statistics
    converted to beta/se_beta inputs for the iDEA object
  repo: https://github.com/statOmics/zinbwaveZinger
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:d0a7f27393e67c2143fc4104a2a2da821baee7baa1c1357537636930a7959a7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CAMERA
  - DAVID
  - GSEA
  - MAST
  - PAGE
  - R
  - SCDE
  - fGSEA
  - iDEA
  - zingeR
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# permutation-based-fdr-estimation

## Summary

Construct a permuted null distribution for gene set enrichment tests by permuting gene labels within each gene set using iDEA.fit.null, then estimate calibrated gene set FDR with iDEA.FDR. This accounts for test non-independence caused by gene set overlaps, which standard GSE p-value corrections do not.

## When to use

Use when testing many overlapping gene sets from summary-statistics-based enrichment (e.g., iDEA on scRNA-seq DE z-scores) and you need calibrated FDR estimates rather than naive p-value thresholds that ignore gene set non-independence.

## When NOT to use

- Gene sets being tested are non-overlapping and small in number, so test non-independence from gene set overlap is not a concern and simpler p-value adjustment suffices
- Input is individual-level scRNA-seq counts rather than per-gene summary statistics — iDEA requires beta/beta_var from an upstream DE method such as zingeR-DESeq2, MAST, or SCDE
- Gene sets with coverage rate (annotated genes / tested genes) below the min_precent_annot threshold of 0.0025 — these are filtered out before fitting

## Inputs

- iDEA object created by CreateiDEAObject from DE summary statistics (beta and se_beta per gene, e.g., z-scores via qnorm(pvalue/2, lower.tail=FALSE))
- Gene set annotation matrix (e.g., mouseGeneSets from MGI GO annotations in GAF 2.0 format, or humanGeneSets compiled from MSigDB databases on GRCh37)
- Fitted iDEA model from iDEA.fit with em_iter=15, mcmc_iter=1000, fit.tol=1e-5, min_degene=5

## Outputs

- Permuted null distribution of gene set p-values from iDEA.fit.null
- Estimated FDR per gene set from iDEA.FDR (e.g., counts of gene sets called enriched at FDR 5%)
- Calibrated enrichment p-values via iDEA.louis (Louis method)

## How to apply

After fitting the joint DE/GSE model with iDEA.fit (em_iter = 15, mcmc_iter = 1000, fit.tol = 1e-5, min_degene = 5) on the iDEA object built by CreateiDEAObject (max_var_beta = 100, min_precent_annot = 0.0025), call iDEA.fit.null to fit the model while permuting gene labels in gene sets, constructing the permuted null distribution of gene set p-values. Use num_core > 1 to parallelize the permutation fits. Then compute calibrated enrichment p-values with iDEA.louis (Louis method) and pass them to iDEA.FDR, which calculates the estimated FDR for each gene set given the permuted null. The rationale is that permuted-label nulls absorb the non-independence among gene sets caused by gene overlap, yielding FDR estimates that remain calibrated even when many gene sets share genes.

## Related tools

- **iDEA** (Provides iDEA.fit.null to fit the model with permuted gene labels (constructing the null) and iDEA.FDR to estimate each gene set's FDR against that null) — https://github.com/xzhoulab/iDEA
- **zingeR** (Upstream zingeR-DESeq2 produces the per-gene LogFC and p-value summary statistics converted to beta/se_beta inputs for the iDEA object) — https://github.com/statOmics/zinbwaveZinger

## Examples

```
library(iDEA); idea <- iDEA.fit(idea, min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, num_core=4); idea <- iDEA.fit.null(idea, num_core=4); res <- iDEA.FDR(idea)
```

## Evaluation signals

- Confirm the null was built by permuting gene labels within gene sets (iDEA.fit.null), not by resampling p-values — this is what absorbs gene set overlap
- Enrichment p-values must be Louis-method calibrated (iDEA.louis) before FDR estimation; uncalibrated p-values from iDEA.fit alone should not feed iDEA.FDR
- Reproduced counts of enriched gene sets at FDR 5% should be comparable to reported values, e.g., iDEA identifying 1,268 gene sets on the Usoskin NP1 vs other-cell-type comparison versus 246 for GSEA
- Check genomic-control lambda_gc values: calibrated methods (iDEA, PAGE, fGSEA, GSEA) should show inflation factors near 1 rather than systematically inflated test statistics
- FDR estimates should be reported per gene set against the permuted null p-value distribution, not as simple Bonferroni/BH-adjusted raw p-values

## Limitations

- The permutation null accounts for, but iDEA still does not explicitly model, non-independence among gene sets caused by gene set overlap
- z-score modeling effectively assumes the prior distribution of true effect sizes depends on the standard errors and, subsequently, the sample size
- DE analysis on inferred cell types may yield inflated test statistics with artificially smaller standard errors, which can distort the null and FDR calibration; the authors note future methods must account for cell type inference uncertainty
- Permutation fitting is computationally heavier than a single model fit; use num_core > 1 to parallelize

## Evidence

- [methods] The function iDEA.fit.null fits iDEA model by permuting gene labels in gene set, thus constructing tbe permuted null distribution for gene sets.: "The function iDEA.fit.null fits iDEA model by permuting gene labels in gene set, thus constructing tbe permuted null distribution for gene sets."
- [methods] The function iDEA.FDR calculate the estimated FDR for each gene set given the permuted null distribution of pvalues for gene sets.: "The function iDEA.FDR calculate the estimated FDR for each gene set given the permuted null distribution of pvalues for gene sets."
- [discussion] By estimating FDR based on such permuted null, we can account for test non-independence due to gene set overlaps: "By estimating FDR based on such permuted null, we can account for test non-independence due to gene set overlaps"
- [readme] devtools::install_github('xzhoulab/iDEA'): "devtools::install_github('xzhoulab/iDEA')"
