---
name: hierarchical-bayesian-em-mcmc-fitting
description: Use when gene-level DE summary statistics (beta, beta_var) plus gene
  set annotations are available and you need calibrated per-gene-set enrichment p-values
  that borrow information across genes rather than analyzing one gene at a time.
when_to_use_negative:
- Only individual-level scRNA-seq count matrices are available with no DE summary
  statistics yet — iDEA consumes summary statistics from existing DE tools and does
  not model raw counts
- Gene sets are so small that coverage rate falls below the min_precent_annot=0.0025
  threshold, or the summary statistics contain fewer than min_degene=5 detected DE
  genes
- Gene coefficients have variance above max_var_beta=100 and are filtered out, leaving
  too few informative genes per set
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: iDEA
  role: Implements the hierarchical Bayesian model fitted via EM with internal MCMC,
    plus Louis-method calibration (CreateiDEAObject, iDEA.fit, iDEA.louis, iDEA.fit.null,
    iDEA.FDR)
  repo: https://github.com/xzhoulab/iDEA
- name: fGSEA
  role: Competing GSE method used as a power benchmark against the iDEA EM-MCMC enrichment
    p-values
- name: CAMERA
  role: Competing competitive-test GSE method used as a power benchmark
- name: PAGE
  role: Competing competitive-test GSE method used as a power benchmark
- name: GSEA
  role: Competing competitive-test GSE method used as a power benchmark
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:b2fea64ce144efbeb9930fed008627697b50155a292f5219e7fb0a552df829f7
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Hierarchical Bayesian EM-MCMC fitting with Louis-method calibrated p-values

## Summary

Fits the iDEA hierarchical Bayesian model that jointly performs DE and GSE analysis from DE summary statistics, alternating MCMC posterior computation of gene DE indicators with EM maximization of annotation coefficients, and applies the Louis missing-information correction to obtain calibrated gene set enrichment p-values.

## When to use

Use when gene-level DE summary statistics (beta, beta_var) plus gene set annotations are available and you need calibrated per-gene-set enrichment p-values that borrow information across genes rather than analyzing one gene at a time.

## When NOT to use

- Only individual-level scRNA-seq count matrices are available with no DE summary statistics yet — iDEA consumes summary statistics from existing DE tools and does not model raw counts
- Gene sets are so small that coverage rate falls below the min_precent_annot=0.0025 threshold, or the summary statistics contain fewer than min_degene=5 detected DE genes
- Gene coefficients have variance above max_var_beta=100 and are filtered out, leaving too few informative genes per set

## Inputs

- Gene-level DE summary statistics (beta, beta_var), e.g. summary_data.RData from xzhoulab/iDEA
- Gene set / pathway annotation matrix mapping genes to sets, e.g. annotation_data.RData
- Optional human or mouse gene set collections (humanGeneSets.RData, mouseGeneSets.RData from MSigDB-derived databases: BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID, Reactome)

## Outputs

- idea@gsea data frame with columns annot_id, annot_coef, annot_var, annot_var_louis, sigma2_b, pvalue_louis, pvalue
- Louis-corrected variance (annot_var_louis) and calibrated enrichment p-value (pvalue_louis) per gene set
- Estimated FDR per gene set when iDEA.fit.null and iDEA.FDR are run on the permuted null distribution

## How to apply

Load DE summary statistics and gene set annotations (e.g. data(summary_data), data(annotation_data) from the iDEA package) and build the analysis container with CreateiDEAObject, using max_var_beta=100 to keep genes whose coefficient variance is below the cutoff and min_precent_annot=0.0025 as the coverage-rate threshold (gene set size / tested genes). Run iDEA.fit with init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, modelVariant=FALSE, which alternates internal MCMC posterior computation of gene DE indicators with maximization of the annotation coefficients (z-score modeling when modelVariant=FALSE). Because the observed-data information is incomplete after the EM, apply iDEA.louis to obtain the Louis-corrected variance annot_var_louis and the calibrated enrichment p-value pvalue_louis for each gene set, one gene set at a time. Extract the results from the idea@gsea data frame (annot_id, annot_coef, annot_var, annot_var_louis, sigma2_b, pvalue_louis, pvalue) and, if FDR estimation against gene-set overlap non-independence is needed, construct a permuted null with iDEA.fit.null (permuting gene labels within gene sets) and call iDEA.FDR.

## Related tools

- **iDEA** (Implements the hierarchical Bayesian model fitted via EM with internal MCMC, plus Louis-method calibration (CreateiDEAObject, iDEA.fit, iDEA.louis, iDEA.fit.null, iDEA.FDR)) — https://github.com/xzhoulab/iDEA
- **fGSEA** (Competing GSE method used as a power benchmark against the iDEA EM-MCMC enrichment p-values)
- **CAMERA** (Competing competitive-test GSE method used as a power benchmark)
- **PAGE** (Competing competitive-test GSE method used as a power benchmark)
- **GSEA** (Competing competitive-test GSE method used as a power benchmark)

## Examples

```
idea <- CreateiDEAObject(summary_data, annotation_data, max_var_beta = 100, min_precent_annot = 0.0025, num_core = 10); idea <- iDEA.fit(idea, init_tau = c(-2, 0.5), min_degene = 5, em_iter = 15, mcmc_iter = 1000, fit.tol = 1e-5, modelVariant = FALSE); idea <- iDEA.louis(idea)
```

## Evaluation signals

- idea@gsea output contains both annot_var_louis and pvalue_louis columns, and every pvalue_louis lies within [0, 1]
- At fixed FDR of 5% under the baseline simulation (tau0=-2, tau1=0.5, CR=10%), power should approximate iDEA 98% vs fGSEA 26%, CAMERA 0%, PAGE 8%, GSEA 26% within Monte Carlo error
- The 9,000 null replicates in the baseline power simulation show no inflation, confirming false-positive control
- EM convergence is reached within fit.tol=1e-5 over em_iter=15 iterations with mcmc_iter=1000 MCMC samples
- Gene sets and genes failing filter steps (min_precent_annot=0.0025, max_var_beta=100, min_degene=5) are excluded from the fitted object

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets; FDR must instead be estimated against a permuted null (iDEA.fit.null permuting gene labels)
- With modelVariant=FALSE (z-score modeling), the prior distribution of true effect sizes effectively depends on standard errors and hence sample size
- DE analysis on inferred cell types may produce inflated DE test statistics with artificially smaller standard errors, degrading the summary-statistic input

## Evidence

- [methods] The iDEA object is created by the function CreateiDEAObject.: "The iDEA object is created by the function CreateiDEAObject."
- [methods] The function iDEA.fit fits the iDEA model on the input data.: "The function iDEA.fit fits the iDEA model on the input data."
- [methods] iDEA utilizes Louis method to compute calibrated p-values for testing gene set enrichment: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [methods] The function iDEA.fit.null fits iDEA model by permuting gene labels in gene set, constructing the permuted null.: "The function iDEA.fit.null fits iDEA model by permuting gene labels in gene set, thus constructing tbe permuted null distribution for gene sets."
- [abstract] perform joint DE and GSE analysis through a hierarchical Bayesian framework: "we develop an integrative and scalable computational method, iDEA, to perform joint DE and GSE analysis through a hierarchical Bayesian framework"
- [readme] devtools::install_github('xzhoulab/iDEA'): "devtools::install_github('xzhoulab/iDEA')"
