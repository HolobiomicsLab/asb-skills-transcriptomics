---
name: gene-set-enrichment-summary-statistics-testing
description: Use when you have per-gene DE summary statistics (e.g., z-scores from
  MAST/SCDE/zingeR) plus gene set annotations and need a calibrated p-value testing
  whether each gene set is enriched in differentially expressed genes.
when_to_use_negative:
- You only have raw scRNA-seq count matrices and no DE summary statistics — iDEA requires
  summary statistics output from existing DE tools and does not model individual-level
  data.
- You need p-values that explicitly account for gene set overlap/non-independence
  — iDEA does not account for this; use the permutation-based iDEA.fit.null / iDEA.FDR
  workflow instead for FDR estimation.
- A gene set is below the coverage threshold (min_precent_annot default 0.0025) or
  too few DE genes exist (min_degene = 5), making enrichment testing unstable.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: iDEA
  role: R package implementing the hierarchical Bayesian EM-MCMC enrichment model
    (CreateiDEAObject, iDEA.fit) and the Louis-method p-value correction (iDEA.louis)
  repo: https://github.com/xzhoulab/iDEA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:99783e09efcb2c9a1f0e49cd62b0a4860b82189bbb218f9a320bd6a013738d7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - iDEA
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gene-set-enrichment-summary-statistics-testing

## Summary

Calibrated gene set enrichment testing from DE summary statistics using iDEA: an EM-MCMC fit of a hierarchical Bayesian model per gene set, with the Louis missing-information correction to obtain valid enrichment p-values.

## When to use

Use when you have per-gene DE summary statistics (e.g., z-scores from MAST/SCDE/zingeR) plus gene set annotations and need a calibrated p-value testing whether each gene set is enriched in differentially expressed genes.

## When NOT to use

- You only have raw scRNA-seq count matrices and no DE summary statistics — iDEA requires summary statistics output from existing DE tools and does not model individual-level data.
- You need p-values that explicitly account for gene set overlap/non-independence — iDEA does not account for this; use the permutation-based iDEA.fit.null / iDEA.FDR workflow instead for FDR estimation.
- A gene set is below the coverage threshold (min_precent_annot default 0.0025) or too few DE genes exist (min_degene = 5), making enrichment testing unstable.

## Inputs

- DE summary statistics table (data(summary_data) from the iDEA package; e.g., z-scores from MAST, SCDE, or zingeR)
- Gene set annotation matrix (data(annotation_data) from the iDEA package; GRCh37 sets from MSigDB-derived databases such as KEGG, Reactome, BioCarta, GO)

## Outputs

- idea@gsea data frame with columns annot_id, annot_coef, annot_var, annot_var_louis, sigma2_b, pvalue_louis, pvalue
- Calibrated enrichment p-value per gene set (pvalue_louis, within [0, 1])

## How to apply

Load DE summary statistics (data(summary_data)) and gene set annotations (data(annotation_data)) into iDEA, then build the analysis container with CreateiDEAObject(summary_data, annotation_data, max_var_beta = 100, min_precent_annot = 0.0025, num_core = 10). Run iDEA.fit with init_tau = c(-2, 0.5), min_degene = 5, em_iter = 15, mcmc_iter = 1000, fit.tol = 1e-5, modelVariant = FALSE; this alternates internal MCMC posterior computation of gene DE indicators with maximization of the annotation coefficients. Apply iDEA.louis(idea) to compute the Louis-method adjusted variance (annot_var_louis) and the calibrated enrichment p-value (pvalue_louis), since iDEA uses the Louis method to correct for missing information in the EM fit. Export the idea@gsea data frame (annot_id, annot_coef, annot_var, annot_var_louis, sigma2_b, pvalue_louis, pvalue) for downstream inspection.

## Related tools

- **iDEA** (R package implementing the hierarchical Bayesian EM-MCMC enrichment model (CreateiDEAObject, iDEA.fit) and the Louis-method p-value correction (iDEA.louis)) — https://github.com/xzhoulab/iDEA

## Examples

```
idea <- CreateiDEAObject(summary_data, annotation_data, max_var_beta = 100, min_precent_annot = 0.0025, num_core = 10); idea <- iDEA.fit(idea, init_tau = c(-2, 0.5), min_degene = 5, em_iter = 15, mcmc_iter = 1000, fit.tol = 1e-5, modelVariant = FALSE); idea <- iDEA.louis(idea)
```

## Evaluation signals

- Output file idea_gsea_results.csv exists and contains the annot_var_louis and pvalue_louis fields.
- All pvalue_louis entries fall within [0, 1].
- The idea@gsea data frame contains the expected columns: annot_id, annot_coef, annot_var, annot_var_louis, sigma2_b, pvalue_louis, pvalue.
- Gene sets failing the min_precent_annot (0.0025) or min_degene (5) thresholds are excluded from results.
- EM convergence reached fit.tol = 1e-5 within em_iter = 15 iterations with mcmc_iter = 1000 MCMC samples per fit.

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets.
- Z-score modeling effectively assumes the prior distribution of true effect sizes depends on the standard errors and sample size.
- DE analysis on inferred cell types may lead to inflated DE test statistics with artificially smaller standard errors.

## Evidence

- [methods] Louis method p-value correction: [evidence span withheld — anchor status: not_found]
- [methods] CreateiDEAObject and iDEA.fit workflow: [evidence span withheld — anchor status: not_found]
- [methods] filter parameter min_percent_annot: "min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the number of tested genes. Default value is 0.0025."
- [intro] summary-statistics-only input: "iDEA makes use of summary statistics output from existing DE tools and does not make explicit modeling assumptions on the individual-level scRNA-seq data"
- [readme] package installation from GitHub: "devtools::install_github('xzhoulab/iDEA')"
