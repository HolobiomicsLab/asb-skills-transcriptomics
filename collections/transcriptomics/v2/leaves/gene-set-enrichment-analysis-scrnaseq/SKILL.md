---
name: gene-set-enrichment-analysis-scrnaseq
description: Use when you have gene-level differential expression summary statistics
  (beta, beta_var or p-values) from scRNA-seq data (e.g. MAST, SCDE, zingeR-DESeq2)
  and need calibrated gene set enrichment p-values and FDR estimates.
when_to_use_negative:
- You only have a scored gene list without DE summary statistics or per-gene variances,
  so summary-statistic-based joint modeling is not applicable
- You need explicit modeling of non-independence among gene sets caused by gene set
  overlap — iDEA only addresses this indirectly via the permuted null, and the article
  lists this as a limitation
- Input is individual-level scRNA-seq counts and the goal is DE alone; prefer MAST,
  SCDE, or zingeR-DESeq2 directly, then feed their summary statistics into this skill
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0659
tools:
- name: iDEA
  role: 'Primary method: fits the joint DE/GSE hierarchical Bayesian model on summary
    statistics and computes calibrated p-values and permuted-null FDR'
  repo: https://github.com/xzhoulab/iDEA
- name: zingeR
  role: Produces DE summary statistics via zingeR-DESeq2 as input to iDEA; also serves
    as a baseline DE method
  repo: https://github.com/statOmics/zinbwaveZinger
- name: MAST
  role: Baseline DE method whose per-gene LogFC and p-values are converted to iDEA
    input summary statistics
- name: SCDE
  role: Baseline DE method providing per-gene summary statistics for the iDEA workflow
- name: fGSEA
  role: Competitive-test GSE baseline run on the same summary statistics for power
    comparison
- name: GSEA
  role: Competitive-test GSE baseline for benchmarking iDEA's power gain
- name: CAMERA
  role: Competitive-test GSE baseline that accounts for gene-gene correlation, used
    in the method comparison
- name: PAGE
  role: Competitive-test GSE baseline run on identical summary statistics
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:61134a16173a65412d215a6dc263bdfaf977e93e8643b5dcc7a3f8a192b73feb
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# gene-set-enrichment-analysis-scRNAseq

## Summary

Perform gene set enrichment analysis (GSE) on single-cell RNA-seq data using DE summary statistics (beta coefficients and variances) with iDEA's hierarchical Bayesian framework, which jointly models DE and GSE and achieves up to five-fold power gains over fGSEA, CAMERA, PAGE, and GSEA.

## When to use

Use when you have gene-level differential expression summary statistics (beta, beta_var or p-values) from scRNA-seq data (e.g. MAST, SCDE, zingeR-DESeq2) and need calibrated gene set enrichment p-values and FDR estimates.

## When NOT to use

- You only have a scored gene list without DE summary statistics or per-gene variances, so summary-statistic-based joint modeling is not applicable
- You need explicit modeling of non-independence among gene sets caused by gene set overlap — iDEA only addresses this indirectly via the permuted null, and the article lists this as a limitation
- Input is individual-level scRNA-seq counts and the goal is DE alone; prefer MAST, SCDE, or zingeR-DESeq2 directly, then feed their summary statistics into this skill

## Inputs

- Gene-level DE summary statistics (beta coefficient and beta variance per gene) from MAST, SCDE, or zingeR-DESeq2, as a data.frame with gene names as row names
- Gene set collections (e.g. mouseGeneSets from MGI GO GAF 2.0, humanGeneSets from MSigDB databases BioCarta/KEGG/GO/PubChem Compound/ImmuneSigDB/PID/Reactome, or Xcell-derived annotations)
- Raw scRNA-seq count matrix (e.g. pbmc3k: 2,638 cells x 13,713 genes; Usoskin GSE59739) if DE statistics must be computed first

## Outputs

- Calibrated per-gene-set enrichment p-values (iDEA.louis, Louis method)
- Per-gene-set FDR estimates from the permuted null (iDEA.FDR)
- Gene-level DE posterior inclusion probabilities via iDEA.BMA
- Counts of enriched gene sets at a given FDR threshold (e.g. FDR 5%) for comparison with fGSEA/CAMERA/PAGE/GSEA

## How to apply

Convert DE results (e.g. from zingeR-DESeq2, MAST, or SCDE) to iDEA summary statistics by computing z-scores with qnorm(pvalue/2, lower.tail=FALSE) and se_beta = abs(beta/zscore). Build the iDEA object with CreateiDEAObject using gene set collections (e.g. mouseGeneSets from MGI GO annotations, humanGeneSets, or Xcell-derived immune/stroma annotations) with max_var_beta = 100 and min_precent_annot = 0.0025. Fit the joint model with iDEA.fit (min_degene = 5, em_iter = 15, mcmc_iter = 1000, fit.tol = 1e-5), then calibrate enrichment p-values with iDEA.louis via the Louis method. Because iDEA does not explicitly account for gene set overlap, estimate FDR from a permuted null using iDEA.fit.null (permuting gene labels) and iDEA.FDR. Benchmark against fGSEA, CAMERA, PAGE, and GSEA run on the same summary statistics, e.g. counting enriched gene sets at FDR 5%.

## Related tools

- **iDEA** (Primary method: fits the joint DE/GSE hierarchical Bayesian model on summary statistics and computes calibrated p-values and permuted-null FDR) — https://github.com/xzhoulab/iDEA
- **zingeR** (Produces DE summary statistics via zingeR-DESeq2 as input to iDEA; also serves as a baseline DE method) — https://github.com/statOmics/zinbwaveZinger
- **MAST** (Baseline DE method whose per-gene LogFC and p-values are converted to iDEA input summary statistics)
- **SCDE** (Baseline DE method providing per-gene summary statistics for the iDEA workflow)
- **fGSEA** (Competitive-test GSE baseline run on the same summary statistics for power comparison)
- **GSEA** (Competitive-test GSE baseline for benchmarking iDEA's power gain)
- **CAMERA** (Competitive-test GSE baseline that accounts for gene-gene correlation, used in the method comparison)
- **PAGE** (Competitive-test GSE baseline run on identical summary statistics)

## Examples

```
library(iDEA); idea <- CreateiDEAObject(gene_name, beta, beta_var, gene_sets, max_var_beta=100, min_precent_annot=0.0025); idea <- iDEA.fit(idea, min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5); idea <- iDEA.louis(idea)
```

## Evaluation signals

- On Usoskin NP1 vs other cell types, iDEA should identify ~1,268 enriched gene sets at FDR 5%, roughly five times GSEA (246), with fGSEA/CAMERA/PAGE at 236/134/205
- On the 10X PBMC CD4+ vs CD8+ T cell comparison, iDEA should detect 255 DE genes at FDR 1% using the CD8+ T-effector memory gene set annotation, vs 221 for zingeR (15.3% gain)
- Among the top 25 gene sets, iDEA should recover ~22 CD4/CD8-relevant sets versus 13/13/14/13 for fGSEA/CAMERA/PAGE/GSEA, with Jaccard similarity of enrichment rankings around 0.06 -> 0.15
- Genomic-control lambda_gc should be computed per method to confirm calibrated p-values; iDEA, PAGE, fGSEA and GSEA (which do not explicitly model gene-gene correlation) should show more calibrated p-values
- CreateiDEAObject filter checks should hold: genes with beta variance below max_var_beta = 100 are retained and gene sets below min_precent_annot = 0.0025 coverage are dropped

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets; FDR calibration relies on permuting gene labels (iDEA.fit.null + iDEA.FDR)
- z-score modeling effectively assumes the prior distribution of true effect sizes depends on the standard errors and hence the sample size
- DE analysis performed on inferred cell types may lead to inflated DE test statistics with artificially smaller standard errors
- Future methodological work is still needed to account for uncertainty associated with cell type inference in DE analysis between cell types

## Evidence

- [abstract] iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods on three scRNA-seq data sets: "iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods"
- [abstract] iDEA uses only DE summary statistics as input: "iDEA uses only DE summary statistics as input, enabling effective data modeling through complementing and pairing with various existing DE methods"
- [methods] min_percent_annot default: "min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the number of tested genes. Default value is 0.0025."
- [methods] iDEA.fit.null permuted null: "The function iDEA.fit.null fits iDEA model by permuting gene labels in gene set, thus constructing tbe permuted null distribution for gene sets."
- [methods] Louis method p-value calibration: [evidence span withheld — anchor status: not_found]
- [discussion] Calibrated p-values for GSE methods without gene-gene correlation modeling: "GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values"
- [readme] Installation command from README: "devtools::install_github('xzhoulab/iDEA')"
