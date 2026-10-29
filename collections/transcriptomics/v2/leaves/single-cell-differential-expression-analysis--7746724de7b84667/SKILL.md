---
name: single-cell-differential-expression-analysis
description: Use when scRNA-seq count data or DE summary statistics (per-gene beta
  coefficient and variance) are available and gene-level differential expression calls
  between cell types or conditions are needed, especially when power or cross-method
  consistency is a concern.
when_to_use_negative:
- When the analysis goal is gene set enrichment rather than gene-level DE calls —
  use iDEA's GSE branch (iDEA.louis / iDEA.FDR) or fGSEA/GSEA/CAMERA instead
- When only individual-level scRNA-seq counts with no DE tool output exist and you
  need an off-the-shelf per-gene test without summarizing to beta/beta_var statistics
  for joint modeling
- When DE is performed on inferred cell types whose uncertainty must be propagated
  — the article notes this can yield inflated DE test statistics with artificially
  smaller standard errors
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: iDEA
  role: Integrative framework that consumes DE summary statistics (beta, beta_var)
    and augments gene-level DE calls via iDEA.fit and iDEA.BMA Bayesian model averaging
  repo: https://github.com/xzhoulab/iDEA
- name: MAST
  role: Standalone scRNA-seq DE method whose per-gene summary statistics feed CreateiDEAObject
- name: zingeR
  role: Zero-inflation-aware DE method; zingeR-DESeq2 produces LogFC and p-values
    for the Usoskin NP1 comparison, and its statistics yield 81% iDEA-augmented power
    vs 65% standalone
  repo: https://github.com/statOmics/zinbwaveZinger
- name: edgeR
  role: Bulk-tuned DE method applied with ZINB-WaVE observation weights; its summary
    statistics give 83% iDEA-augmented power vs 67% standalone
- name: SCDE
  role: Standalone scRNA-seq DE method used on the Usoskin dataset to generate per-gene
    LogFC and p-values
- name: zinbwave
  role: Provides observation weights used to adapt edgeR to single-cell zero inflation
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:088c181c7e65cd8cc2e95e4c47ca50c755102f6835355956f59a4675574f16f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CAMERA
  - CreateiDEAObject
  - DAVID
  - GSEA
  - MAST
  - PAGE
  - R
  - SCDE
  - edgeR
  - iDEA
  - iDEA.BMA
  - iDEA.fit
  - zinbwave
  - zingeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# single-cell-differential-expression-analysis

## Summary

Perform differential expression (DE) analysis on scRNA-seq data with MAST, SCDE, zingeR/zingeR-DESeq2, or edgeR with ZINB-WaVE weights, then optionally augment the per-gene evidence with iDEA's BMA aggregation to gain power by borrowing information across genes. This skill covers running the standalone DE tools, converting their outputs (beta and beta_var summary statistics) to iDEA input, and evaluating power/consistency of the resulting DE gene lists.

## When to use

Use when scRNA-seq count data or DE summary statistics (per-gene beta coefficient and variance) are available and gene-level differential expression calls between cell types or conditions are needed, especially when power or cross-method consistency is a concern.

## When NOT to use

- When the analysis goal is gene set enrichment rather than gene-level DE calls — use iDEA's GSE branch (iDEA.louis / iDEA.FDR) or fGSEA/GSEA/CAMERA instead
- When only individual-level scRNA-seq counts with no DE tool output exist and you need an off-the-shelf per-gene test without summarizing to beta/beta_var statistics for joint modeling
- When DE is performed on inferred cell types whose uncertainty must be propagated — the article notes this can yield inflated DE test statistics with artificially smaller standard errors

## Inputs

- scRNA-seq count matrix with cell-type or condition labels (e.g. GEO GSE59739 mouse sensory neuron data, NP1 vs others)
- Per-gene DE summary statistics: beta coefficient and beta variance (or p-values convertible via qnorm(pvalue/2, lower.tail=FALSE)) from MAST, SCDE, zingeR-DESeq2, or edgeR+ZINB-WaVE
- Gene annotation/gene-set object (e.g. mouseGeneSets from MGI GO annotations in GAF 2.0 format)

## Outputs

- Per-gene DE results: log fold-changes, p-values, and FDR-adjusted significance calls (e.g. DE genes at FDR 1% or 5%)
- iDEA-augmented gene-level posterior inclusion probabilities from iDEA.BMA
- Power, Jaccard-consistency, and genomic-control (lambda_gc) metrics across DE methods

## How to apply

Run standalone DE with MAST, edgeR with ZINB-WaVE weighting, or zingeR to produce gene-level beta coefficients and variances; for zingeR-DESeq2 on the Usoskin mouse sensory neuron data (GSE59739, NP1 vs all other cell types), compute z-scores with qnorm(pvalue/2, lower.tail=FALSE) and se_beta = abs(beta/zscore) to obtain iDEA summary statistics. Build the iDEA object with CreateiDEAObject (max_var_beta=100 to drop genes with coefficient variance above the cutoff, min_precent_annot=0.0025 coverage-rate threshold) and fit the model with iDEA.fit (min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, init_tau=c(-2,0.5)). Aggregate gene-level DE evidence across gene sets with iDEA.BMA (Bayesian model averaging) to boost power. Validate by computing empirical power at true FDR 5% (expected standalone ~65%/52%/67% vs iDEA-augmented ~81%/61%/83% for zingeR/MAST/edgeR at tau1=5, CR=10%) and the Jaccard index of top-1500 DE gene lists (expected to rise from ~0.59 to ~0.77 with iDEA).

## Related tools

- **iDEA** (Integrative framework that consumes DE summary statistics (beta, beta_var) and augments gene-level DE calls via iDEA.fit and iDEA.BMA Bayesian model averaging) — https://github.com/xzhoulab/iDEA
- **MAST** (Standalone scRNA-seq DE method whose per-gene summary statistics feed CreateiDEAObject)
- **zingeR** (Zero-inflation-aware DE method; zingeR-DESeq2 produces LogFC and p-values for the Usoskin NP1 comparison, and its statistics yield 81% iDEA-augmented power vs 65% standalone) — https://github.com/statOmics/zinbwaveZinger
- **edgeR** (Bulk-tuned DE method applied with ZINB-WaVE observation weights; its summary statistics give 83% iDEA-augmented power vs 67% standalone)
- **SCDE** (Standalone scRNA-seq DE method used on the Usoskin dataset to generate per-gene LogFC and p-values)
- **zinbwave** (Provides observation weights used to adapt edgeR to single-cell zero inflation)

## Examples

```
devtools::install_github('xzhoulab/iDEA'); idea_obj <- CreateiDEAObject(beta, beta_var, gene_ids, annot, max_var_beta=100, min_precent_annot=0.0025); idea_obj <- iDEA.fit(idea_obj, min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5); de_pp <- iDEA.BMA(idea_obj)
```

## Evaluation signals

- At tau1=5 and CR=10%, iDEA-augmented power at true FDR 5% should approximate 81% (zingeR), 61% (MAST), 83% (edgeR) versus standalone 65%, 52%, 67%
- Jaccard index of top-1500 DE gene lists between methods should increase from ~0.59 to ~0.77 after iDEA augmentation
- Summary-statistic conversion check: se_beta = abs(beta / qnorm(pvalue/2, lower.tail=FALSE)) reproduces the input beta variance
- CreateiDEAObject filters: no gene with coefficient variance > max_var_beta (default 100) and gene sets below min_precent_annot (default 0.0025 coverage) are excluded
- On Usoskin NP1 comparison, iDEA.BMA-anchored DE calls at FDR 1% (e.g. anchoring on GO:0097458) should show overlap with the top-100 known NP1 DE genes

## Limitations

- iDEA's z-score modeling effectively assumes the prior distribution of true effect sizes depends on the standard errors, and subsequently the sample size
- DE analysis performed on inferred cell types may lead to inflated DE test statistics with artificially smaller standard errors; future work is needed to account for cell type inference uncertainty
- Computational cost: iDEA.fit requires MCMC (mcmc_iter=1000) per fit, and permuted nulls for FDR calibration require additional iDEA.fit.null runs with num_core > 1 recommended

## Evidence

- [intro] iDEA makes use of summary statistics output from existing DE tools and does not make explicit modeling assumptions on the individual-level scRNA-seq data: "iDEA makes use of summary statistics output from existing DE tools and does not make explicit modeling assumptions on the individual-level scRNA-seq data"
- [intro] Analyzing one gene at a time can lead to potential power loss: "Analyzing one gene at a time can lead to potential power loss, as this approach fails to exploit consistent DE evidence across similar genes"
- [methods] The iDEA object is created by the function CreateiDEAObject.: "The iDEA object is created by the function CreateiDEAObject."
- [methods] max_var_beta cutoff and min_percent_annot default: "max_var_beta: the cutoff of the variance of the coefficient of genes. Genes with variance smaller than 'max_var_beta' are maintained. Default is 100"
- [methods] BMA aggregation of DE evidence: "we developed a Bayesian model averaging (BMA) approach to aggregate DE evidence for any given genes across all available gene sets"
- [intro] Commonly applied DE methods in scRNA-seq: "Commonly applied DE methods in scRNA-seq include MAST3, SCDE4, and zingeR5, to name a few."
- [readme] R installation from GitHub: "devtools::install_github('xzhoulab/iDEA')"
