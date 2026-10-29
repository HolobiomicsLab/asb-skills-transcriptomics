---
name: hierarchical-bayesian-joint-de-gse-modeling
description: Use when you have gene-level summary statistics (beta coefficients and
  coefficient variances) from a scRNA-seq DE method and want joint DE + GSE analysis
  with calibrated gene set p-values.
when_to_use_negative:
- When only individual-level scRNA-seq counts are available and no DE tool has produced
  summary statistics — iDEA requires beta coefficients and their variances as input
- When gene set overlap/non-independence must be explicitly modeled in the joint fit
  — iDEA only handles it post hoc through the permuted null (iDEA.fit.null / iDEA.FDR)
- When DE is performed on inferred cell types whose test statistics may be inflated
  with artificially smaller standard errors — the z-score prior effectively assumes
  the true effect size distribution depends on the standard errors and sample size
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: iDEA
  role: 'Core method: CreateiDEAObject, iDEA.fit, iDEA.louis, iDEA.fit.null, iDEA.FDR,
    and iDEA.BMA implement the joint hierarchical Bayesian DE/GSE analysis on summary
    statistics'
  repo: https://github.com/xzhoulab/iDEA
- name: zingeR
  role: Upstream scRNA-seq DE method producing the beta coefficients and coefficient
    variances consumed by iDEA; also the DE competitor benchmarked on pbmc3k
  repo: https://github.com/statOmics/zinbwaveZinger
- name: MAST
  role: Alternative upstream DE method for computing gene-level summary statistics
    and a comparison DE approach using the CD8+ T-effector memory signature gene set
- name: fGSEA
  role: Competing GSE method run on the same summary statistics and gene sets for
    benchmarking enrichment rankings
- name: CAMERA
  role: Competing competitive-test GSE method benchmarked against iDEA
- name: PAGE
  role: Competing competitive-test GSE method benchmarked against iDEA
- name: GSEA
  role: Competing competitive-test GSE method benchmarked against iDEA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:226759b22fa19a6f829149991d1d9996a0d2301e468bd75710c9113dad83341b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CAMERA
  - GSEA
  - MAST
  - PAGE
  - SCDE
  - fGSEA
  - iDEA
  - zingeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# hierarchical-bayesian-joint-de-gse-modeling

## Summary

Fit iDEA, a hierarchical Bayesian model that jointly performs differential expression (DE) and gene set enrichment (GSE) analysis from scRNA-seq summary statistics, borrowing information across genes to boost power and consistency of both analyses.

## When to use

Use when you have gene-level summary statistics (beta coefficients and coefficient variances) from a scRNA-seq DE method and want joint DE + GSE analysis with calibrated gene set p-values.

## When NOT to use

- When only individual-level scRNA-seq counts are available and no DE tool has produced summary statistics — iDEA requires beta coefficients and their variances as input
- When gene set overlap/non-independence must be explicitly modeled in the joint fit — iDEA only handles it post hoc through the permuted null (iDEA.fit.null / iDEA.FDR)
- When DE is performed on inferred cell types whose test statistics may be inflated with artificially smaller standard errors — the z-score prior effectively assumes the true effect size distribution depends on the standard errors and sample size

## Inputs

- Gene-level DE summary statistics data.frame (beta coefficients and coefficient variances, gene names as row names) from zingeR-DESeq2, MAST, SCDE, or similar scRNA-seq DE tools
- scRNA-seq count matrix, e.g. pbmc3k from 10X Genomics (2,638 cells, 13,713 genes)
- Gene set annotations, e.g. the 144 Xcell-derived immune/stroma annotations from annotation_data.RData or humanGeneSets.RData

## Outputs

- Fitted iDEA object with joint DE/GSE model parameters
- Louis-calibrated gene set enrichment p-values and estimated FDR per gene set
- Significant DE gene counts at a chosen FDR threshold (e.g. 255 vs 221 at FDR 1% for iDEA vs zingeR on pbmc3k)
- Results table such as pbmc3k_idea_results.csv

## How to apply

Compute gene-level summary statistics (beta coefficients and coefficient variances) with a scRNA-seq DE method such as zingeR-DESeq2 or MAST, keeping a data.frame with gene names as row names, then build the iDEA object with CreateiDEAObject (defaults max_var_beta = 100, min_precent_annot = 0.0025). Fit the joint DE/GSE hierarchical Bayesian model with iDEA.fit (em_iter = 15, mcmc_iter = 1000, fit.tol = 1e-5) and calibrate enrichment p-values with iDEA.louis (Louis method). On the 10X Genomics PBMC data (pbmc3k, 13,713 genes, 2,638 cells, CD4+ vs CD8+ T cells) with the 144 Xcell-derived immune/stroma annotations, this detects 255 significant DE genes at FDR 1% versus 221 for zingeR (15.3% higher) and ranks 22/25 top gene sets as CD4/CD8-relevant versus 13/13/14/13 for fGSEA/CAMERA/PAGE/GSEA. For FDR, fit a permuted null with iDEA.fit.null and estimate gene set FDR with iDEA.FDR, which accounts for test non-independence from gene set overlap. Because iDEA consumes only summary statistics, it makes no explicit modeling assumptions on individual-level counts, enabling scalable computation.

## Related tools

- **iDEA** (Core method: CreateiDEAObject, iDEA.fit, iDEA.louis, iDEA.fit.null, iDEA.FDR, and iDEA.BMA implement the joint hierarchical Bayesian DE/GSE analysis on summary statistics) — https://github.com/xzhoulab/iDEA
- **zingeR** (Upstream scRNA-seq DE method producing the beta coefficients and coefficient variances consumed by iDEA; also the DE competitor benchmarked on pbmc3k) — https://github.com/statOmics/zinbwaveZinger
- **MAST** (Alternative upstream DE method for computing gene-level summary statistics and a comparison DE approach using the CD8+ T-effector memory signature gene set)
- **fGSEA** (Competing GSE method run on the same summary statistics and gene sets for benchmarking enrichment rankings)
- **CAMERA** (Competing competitive-test GSE method benchmarked against iDEA)
- **PAGE** (Competing competitive-test GSE method benchmarked against iDEA)
- **GSEA** (Competing competitive-test GSE method benchmarked against iDEA)

## Examples

```
idea <- CreateiDEAObject(beta, se_beta, gene_name, annotation); idea <- iDEA.fit(idea, em_iter=15, mcmc_iter=1000, fit.tol=1e-5); idea <- iDEA.louis(idea)
```

## Evaluation signals

- On pbmc3k CD4+ vs CD8+ T cells, iDEA should detect 255 significant DE genes at FDR 1% versus 221 for zingeR (15.3% more) using the CD8+ T-effector memory gene set as annotation
- Among the top 25 gene sets per method, 22 should be CD4/CD8-relevant for iDEA versus 13/13/14/13 for fGSEA/CAMERA/PAGE/GSEA
- Jaccard similarity of enrichment rankings between iDEA and competitors should improve from 0.06 to 0.15 (Fig. 6), matching the published reproduction target
- Verify filter steps were applied: genes with coefficient variance below max_var_beta = 100 retained and gene sets with coverage rate above min_precent_annot = 0.0025 kept
- Enrichment p-values must come from the Louis-method calibration step (iDEA.louis), and gene set FDR from the permuted null (iDEA.fit.null + iDEA.FDR), not raw iDEA.fit output

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets; this is only addressed via permutation of gene labels in the null
- Z-score modeling effectively assumes the prior distribution of true effect sizes depends on the standard errors, and subsequently the sample size
- DE analysis performed on inferred cell types may lead to inflated DE test statistics with artificially smaller standard errors; future work is needed to account for uncertainty in cell type inference

## Evidence

- [abstract] we develop an integrative and scalable computational method, iDEA, to perform joint DE and GSE analysis through a hierarchical Bayesian framework: "we develop an integrative and scalable computational method, iDEA, to perform joint DE and GSE analysis through a hierarchical Bayesian framework"
- [methods] The iDEA object is created by the function CreateiDEAObject.: "The iDEA object is created by the function CreateiDEAObject."
- [methods] iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [methods] The function iDEA.fit.null fits iDEA model by permuting gene labels in gene set, thus constructing tbe permuted null distribution for gene sets.: "The function iDEA.fit.null fits iDEA model by permuting gene labels in gene set, thus constructing tbe permuted null distribution for gene sets."
- [discussion] iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets: "iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets"
- [readme] devtools::install_github('xzhoulab/iDEA'): "devtools::install_github('xzhoulab/iDEA')"
