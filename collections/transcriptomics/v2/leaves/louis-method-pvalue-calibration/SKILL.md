---
name: louis-method-pvalue-calibration
description: Use when testing whether a gene set is enriched in differentially expressed
  genes from scRNA-seq DE summary statistics (beta, beta_var) after fitting iDEA with
  iDEA.fit, and a properly calibrated enrichment p-value is required.
when_to_use_negative:
- Summary statistics are not gene-level DE effect sizes (beta, beta_var) from scRNA-seq
  DE tools, so the iDEA hierarchical model does not apply
- Gene set coverage rate is below the min_precent_annot threshold (default 0.0025)
  or fewer than min_degene = 5 DE genes are detected
- Only an uncalibrated nominal p-value is acceptable and the computational cost of
  MCMC (mcmc_iter = 1000) plus the Louis information correction is prohibitive
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3170
tools:
- name: iDEA
  role: Provides the EM-MCMC model fit (iDEA.fit) and the Louis-method corrected inference
    (iDEA.louis) that yields calibrated enrichment p-values
  repo: https://github.com/xzhoulab/iDEA
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:be411edb4ebf5fbc2150fd751c22b0405ec00ccc99b81956c2bb4988bcbe933a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# louis-method-pvalue-calibration

## Summary

Computes calibrated gene-set enrichment p-values in iDEA by applying the Louis missing-information correction to the observed Fisher information after EM-MCMC fitting, correcting the underestimation of variance that arises because latent gene DE indicators are not directly observed.

## When to use

Use when testing whether a gene set is enriched in differentially expressed genes from scRNA-seq DE summary statistics (beta, beta_var) after fitting iDEA with iDEA.fit, and a properly calibrated enrichment p-value is required.

## When NOT to use

- Summary statistics are not gene-level DE effect sizes (beta, beta_var) from scRNA-seq DE tools, so the iDEA hierarchical model does not apply
- Gene set coverage rate is below the min_precent_annot threshold (default 0.0025) or fewer than min_degene = 5 DE genes are detected
- Only an uncalibrated nominal p-value is acceptable and the computational cost of MCMC (mcmc_iter = 1000) plus the Louis information correction is prohibitive

## Inputs

- Gene-level DE summary statistics (beta, beta_var) from an existing DE tool (e.g. zingeR-DESeq2, MAST, SCDE), e.g. iDEA's summary_data
- Gene set annotation matrix (e.g. humanGeneSets or mouseGeneSets from MSigDB/GO/MGI), e.g. iDEA's annotation_data
- An iDEA object fitted via iDEA.fit (with em_iter = 15, mcmc_iter = 1000, fit.tol = 1e-5)

## Outputs

- annot_var_louis: Louis-method adjusted variance of the gene set annotation coefficient
- pvalue_louis: calibrated p-value testing enrichment of each gene set in DE genes
- idea@gsea data frame with columns annot_id, annot_coef, annot_var, annot_var_louis, sigma2_b, pvalue_louis, pvalue (e.g. exported to idea_gsea_results.csv)

## How to apply

First build the analysis container with CreateiDEAObject(summary_data, annotation_data, max_var_beta = 100, min_precent_annot = 0.0025) and fit the model with iDEA.fit(init_tau = c(-2, 0.5), min_degene = 5, em_iter = 15, mcmc_iter = 1000, fit.tol = 1e-5, modelVariant = FALSE); the EM alternates internal MCMC posterior computation of gene DE indicators with maximization of the annotation coefficients. Because the latent DE indicators are unobserved, the raw annotated variance underestimates uncertainty, so run iDEA.louis(idea) to obtain the Louis-method adjusted variance annot_var_louis and the calibrated enrichment p-value pvalue_louis, computed one gene set at a time. The resulting pvalue_louis entries lie in [0, 1] and are the values used downstream, e.g. for fixed-FDR 5% power comparisons against fGSEA, CAMERA, PAGE, and GSEA. Per-replicate pvalue_louis values at FDR 5% should reproduce the paper's baseline power simulation results (iDEA 98%, fGSEA 26%, CAMERA 0%, PAGE 8%, GSEA 26% at tau1 = 0.5, CR = 10%) within Monte Carlo error.

## Related tools

- **iDEA** (Provides the EM-MCMC model fit (iDEA.fit) and the Louis-method corrected inference (iDEA.louis) that yields calibrated enrichment p-values) — https://github.com/xzhoulab/iDEA

## Examples

```
idea <- iDEA.fit(CreateiDEAObject(summary_data, annotation_data, max_var_beta = 100, min_precent_annot = 0.0025), init_tau = c(-2, 0.5), em_iter = 15, mcmc_iter = 1000, fit.tol = 1e-5); idea <- iDEA.louis(idea)
```

## Evaluation signals

- All pvalue_louis entries fall within [0, 1] and annot_var_louis is present and finite for every annotated gene set
- The idea@gsea data frame contains the expected columns annot_id, annot_coef, annot_var, annot_var_louis, sigma2_b, pvalue_louis, pvalue
- In the baseline power simulation (tau0 = -2, tau1 = 0.5, CR = 10%), power at fixed FDR 5% computed from pvalue_louis reproduces iDEA's reported 98% (vs fGSEA 26%, CAMERA 0%, PAGE 8%, GSEA 26%) within Monte Carlo error
- The 9,000 null replicates in the power simulation verify false-positive control at FDR 5%

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets; FDR based on a permuted null via iDEA.fit.null and iDEA.FDR is used to mitigate this
- With modelVariant = FALSE (z-score modeling), the prior distribution of true effect sizes effectively depends on the standard errors and hence sample size
- DE analysis on inferred cell types may produce inflated test statistics with artificially smaller standard errors, degrading calibration

## Evidence

- [methods] Louis calibration in iDEA methods: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [discussion] Louis-corrected iDEA p-values are calibrated relative to competing GSE methods: "GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values"
- [methods] Fit precedes Louis correction; EM-MCMC model fitting step: "The function iDEA.fit fits the iDEA model on the input data."
