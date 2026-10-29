---
name: bayesian-hierarchical-model-fitting-scrnaseq
description: Use when you have per-gene summary statistics (beta and beta variance,
  or p-values convertible via qnorm) from a scRNA-seq DE tool such as zingeR-DESeq2
  and want joint DE + gene set enrichment analysis with permutation-based empirical
  FDR.
when_to_use_negative:
- When only individual-level scRNA-seq counts are available with no DE summary statistics
  and no DE tool can be run first — iDEA consumes summary statistics, not raw counts
- When gene set overlap non-independence must be explicitly modeled rather than handled
  by gene-label permutation — iDEA does not explicitly account for gene set overlap
- When DE was performed on inferred cell types that may have inflated test statistics
  with artificially small standard errors
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: iDEA
  role: 'Core hierarchical Bayesian package: CreateiDEAObject, iDEA.fit, iDEA.louis,
    iDEA.BMA, iDEA.fit.null, iDEA.FDR'
  repo: https://github.com/xzhoulab/iDEA
- name: zingeR
  role: Produces the input DE summary statistics via zingeR-DESeq2 differential expression
  repo: https://github.com/statOmics/zinbwaveZinger
- name: fGSEA
  role: Competing GSE method run on the same summary statistics for benchmarking
- name: CAMERA
  role: Competing competitive gene set test for benchmarking
- name: PAGE
  role: Competing competitive gene set test for benchmarking
- name: GSEA
  role: Competing gene set enrichment method for benchmarking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:76f441bd9256fc50d7e0e6eba21a76079e4f55b20d98114b4603948c3f6d1040
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# bayesian-hierarchical-model-fitting-scrnaseq

## Summary

Fits the iDEA hierarchical Bayesian model that jointly performs differential expression and gene set enrichment analysis from DE summary statistics (beta coefficients and standard errors) in scRNA-seq studies, improving GSE power and DE consistency over fGSEA, CAMERA, PAGE, GSEA, and Fisher's exact test.

## When to use

Use when you have per-gene summary statistics (beta and beta variance, or p-values convertible via qnorm) from a scRNA-seq DE tool such as zingeR-DESeq2 and want joint DE + gene set enrichment analysis with permutation-based empirical FDR.

## When NOT to use

- When only individual-level scRNA-seq counts are available with no DE summary statistics and no DE tool can be run first — iDEA consumes summary statistics, not raw counts
- When gene set overlap non-independence must be explicitly modeled rather than handled by gene-label permutation — iDEA does not explicitly account for gene set overlap
- When DE was performed on inferred cell types that may have inflated test statistics with artificially small standard errors

## Inputs

- Per-gene DE summary statistics (beta coefficient and beta variance) from zingeR-DESeq2; if only p-value and LogFC are available, convert via qnorm and set se = |beta/zscore|, beta_var = se^2
- Gene set annotations, e.g. humanGeneSets.RData or mouseGeneSets.RData from xzhoulab/iDEA (MSigDB GO/KEGG/Reactome/BioCarta/ImmuneSigDB/PID/PubChem on GRCh37)
- Reference summary_data.RData / annotation_data.RData from the iDEA GitHub data folder

## Outputs

- Calibrated gene set enrichment p-values (iDEA.louis) with empirical FDR at e.g. 5% (e.g. 2,106 enriched gene sets on human ESC DEC vs EC)
- Aggregated per-gene DE evidence via Bayesian model averaging (iDEA.BMA), e.g. 2,753 DE genes at FDR 1%
- lambda_gc genomic control factors and permutation-null FDR estimates for each compared method

## How to apply

Build the iDEA object with CreateiDEAObject, pairing zingeR-DESeq2 summary statistics with human gene set annotations (MSigDB-derived GO, KEGG, Reactome, BioCarta, PubChem Compound, ImmuneSigDB, PID on GRCh37), filtering with max_var_beta = 100, min_precent_annot = 0.0025, and num_core = 10. Fit the hierarchical Bayesian model with iDEA.fit using init_tau = c(-2, 0.5), min_degene = 5, em_iter = 15, mcmc_iter = 1000, fit.tol = 1e-5, modelVariant = FALSE, then compute Louis-method calibrated GSE p-values with iDEA.louis and aggregate per-gene DE evidence with iDEA.BMA. Because gene sets overlap and tests are non-independent, construct a permuted null by fitting iDEA.fit.null with permuted gene labels and estimate gene-set FDR with iDEA.FDR. Compute the lambda_gc genomic control factor to check calibration; iDEA's z-score modeling (beta/se) makes GSE less susceptible to gene-gene correlations and dropout events than methods using individual-level data.

## Related tools

- **iDEA** (Core hierarchical Bayesian package: CreateiDEAObject, iDEA.fit, iDEA.louis, iDEA.BMA, iDEA.fit.null, iDEA.FDR) — https://github.com/xzhoulab/iDEA
- **zingeR** (Produces the input DE summary statistics via zingeR-DESeq2 differential expression) — https://github.com/statOmics/zinbwaveZinger
- **fGSEA** (Competing GSE method run on the same summary statistics for benchmarking)
- **CAMERA** (Competing competitive gene set test for benchmarking)
- **PAGE** (Competing competitive gene set test for benchmarking)
- **GSEA** (Competing gene set enrichment method for benchmarking)

## Examples

```
library(iDEA); idea <- CreateiDEAObject(beta, beta_var, gene_name, num_core=10, gene_sets, max_var_beta=100, min_precent_annot=0.0025); idea <- iDEA.fit(idea, init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, modelVariant=FALSE); idea <- iDEA.louis(idea)
```

## Evaluation signals

- lambda_gc values reproduce the published calibration (e.g. 1.13, 1.02, 0.96, 1.01, 0.29 on the DEC vs EC comparison) within numerical tolerance
- Number of gene sets enriched at empirical FDR 5% matches or exceeds the reported counts (iDEA 2,106 vs fGSEA 1,742 on human ESC DEC vs EC)
- Number of DE genes at FDR 1% from iDEA.BMA (2,753) exceeds the best competing method (1,673) with consistent Jaccard overlap trajectory (0.10 -> 0.25)
- Permuted-null FDR estimates from iDEA.fit.null + iDEA.FDR are finite and monotone in observed enrichment p-value
- CreateiDEAObject filters satisfy max_var_beta = 100 and min_precent_annot = 0.0025 coverage-rate threshold

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets (handled only via gene-label permutation)
- Z-score modeling effectively assumes the prior distribution of true effect sizes depends on the standard errors and thus the sample size
- DE analysis on inferred cell types may lead to inflated DE test statistics with artificially smaller standard errors; future work is needed to account for cell type inference uncertainty

## Evidence

- [intro] iDEA makes use of summary statistics output from existing DE tools and does not make explicit modeling assumptions on the individual-level scRNA-seq data: "iDEA makes use of summary statistics output from existing DE tools and does not make explicit modeling assumptions on the individual-level scRNA-seq data"
- [methods] Louis method p-values: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [methods] Permuted null FDR: "The function iDEA.FDR calculate the estimated FDR for each gene set given the permuted null distribution of pvalues for gene sets."
- [abstract] Power gain: "iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods"
- [discussion] Gene set overlap limitation: "iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets"
- [readme] Install via GitHub: "devtools::install_github('xzhoulab/iDEA')"
