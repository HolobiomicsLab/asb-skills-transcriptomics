---
name: simulation-based-de-power-evaluation
description: Use when evaluating or comparing DE method power on single-cell RNA-seq
  data where simulated ground truth (known DE genes, coverage rate, tau1/tau0 parameters)
  is available, or when reproducing iDEA power-gain results.
when_to_use_negative:
- Only real scRNA-seq data with no known DE ground truth is available — empirical
  power cannot be computed without simulated or validated DE genes.
- The goal is gene set enrichment (GSE) power or FDR calibration rather than gene-level
  differential expression power.
- Input data is bulk RNA-seq or non-count data incompatible with the MAST/edgeR/zingeR
  summary-statistic pipeline.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: iDEA
  role: Aggregates gene-level DE summary statistics via hierarchical Bayesian model
    and BMA to boost DE power over standalone methods
  repo: https://github.com/xzhoulab/iDEA
- name: MAST
  role: Standalone DE method producing beta-coefficient summary statistics fed into
    iDEA
- name: edgeR
  role: Standalone DE method used with ZINB-WaVE observation weights to produce summary
    statistics
- name: zingeR
  role: Standalone DE method producing gene-level summary statistics; simulation framework
    sourced from statOmics/zinbwaveZinger
  repo: https://github.com/statOmics/zinbwaveZinger
- name: CreateiDEAObject
  role: Constructs iDEA input objects with max_var_beta=100 and min_precent_annot=0.0025
    filters
  repo: https://github.com/xzhoulab/iDEA
- name: iDEA.fit
  role: Fits the iDEA hierarchical Bayesian model (em_iter=15, mcmc_iter=1000, fit.tol=1e-5)
  repo: https://github.com/xzhoulab/iDEA
- name: iDEA.BMA
  role: Bayesian model averaging across gene sets to aggregate DE evidence per gene
  repo: https://github.com/xzhoulab/iDEA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:f93f0694df51e7724f763310bc37e62f3e5c26b3b23c641780c84bc975156b1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CreateiDEAObject
  - MAST
  - edgeR
  - iDEA
  - iDEA.BMA
  - iDEA.fit
  - zingeR
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# simulation-based-de-power-evaluation

## Summary

Benchmark differential expression power on scRNA-seq simulations with known DE genes, comparing standalone DE methods (MAST, edgeR, zingeR) against iDEA-augmented aggregation of their summary statistics, at a fixed true FDR of 5%.

## When to use

Use when evaluating or comparing DE method power on single-cell RNA-seq data where simulated ground truth (known DE genes, coverage rate, tau1/tau0 parameters) is available, or when reproducing iDEA power-gain results.

## When NOT to use

- Only real scRNA-seq data with no known DE ground truth is available — empirical power cannot be computed without simulated or validated DE genes.
- The goal is gene set enrichment (GSE) power or FDR calibration rather than gene-level differential expression power.
- Input data is bulk RNA-seq or non-count data incompatible with the MAST/edgeR/zingeR summary-statistic pipeline.

## Inputs

- Simulated single-cell expression count matrix with known DE gene labels (gene set enrichment parameters tau0=-2, tau1=5, CR=0.10)
- Gene-level summary statistics (beta coefficient and variance) from MAST, edgeR (ZINB-WaVE weighted), and zingeR
- Gene set annotations (e.g., humanGeneSets.RData / mouseGeneSets.RData from the iDEA repository)

## Outputs

- Empirical DE power per method (standalone vs iDEA-augmented) at true FDR 5%
- Jaccard index of top-1500 DE gene lists between methods
- FDR-controlled DE gene lists per method

## How to apply

Simulate gene sets and DE signals at coverage rate CR=0.10 with gene set enrichment parameters tau0=-2 and tau1=5, generating single-cell expression data with known DE genes (the zinbwaveZinger simulationHelpFunctions framework supports such parameterized simulations). Run standalone DE analyses with MAST, edgeR with ZINB-WaVE weighting, and zingeR to produce gene-level summary statistics (beta coefficient and its variance) per method. Build iDEA objects with CreateiDEAObject (max_var_beta=100, min_precent_annot=0.0025), then fit with iDEA.fit (fit_noGS=FALSE, init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, modelVariant=FALSE) and aggregate evidence with iDEA.BMA. Compute empirical DE power of each method standalone and iDEA-augmented at true FDR 5%, and compute the Jaccard index of top-1500 DE gene lists across methods. Rationale: analyzing one gene at a time loses power by failing to exploit consistent DE evidence across genes, which iDEA's hierarchical Bayesian joint model recovers.

## Related tools

- **iDEA** (Aggregates gene-level DE summary statistics via hierarchical Bayesian model and BMA to boost DE power over standalone methods) — https://github.com/xzhoulab/iDEA
- **MAST** (Standalone DE method producing beta-coefficient summary statistics fed into iDEA)
- **edgeR** (Standalone DE method used with ZINB-WaVE observation weights to produce summary statistics)
- **zingeR** (Standalone DE method producing gene-level summary statistics; simulation framework sourced from statOmics/zinbwaveZinger) — https://github.com/statOmics/zinbwaveZinger
- **CreateiDEAObject** (Constructs iDEA input objects with max_var_beta=100 and min_precent_annot=0.0025 filters) — https://github.com/xzhoulab/iDEA
- **iDEA.fit** (Fits the iDEA hierarchical Bayesian model (em_iter=15, mcmc_iter=1000, fit.tol=1e-5)) — https://github.com/xzhoulab/iDEA
- **iDEA.BMA** (Bayesian model averaging across gene sets to aggregate DE evidence per gene) — https://github.com/xzhoulab/iDEA

## Examples

```
idea <- CreateiDEAObject(beta=res_mast$coef, var_beta=res_mast$var, gene=pvals$gene, annotation=humanGeneSets, max_var_beta=100, min_precent_annot=0.0025)
```

## Evaluation signals

- Reproduced standalone power should match ~65% (zingeR), ~52% (MAST), ~67% (edgeR) at tau1=5, CR=10%, true FDR 5%.
- iDEA-augmented power should rise to ~81% (zingeR), ~61% (MAST), ~83% (edgeR) at true FDR 5%.
- Top-1500 DE gene list Jaccard index should increase from 0.59 to 0.77 when iDEA is applied.
- Observed FDR at the nominal 5% threshold should be approximately calibrated (near 5%) for each method.
- CreateiDEAObject pre-filters should retain only genes with variance of the coefficient < max_var_beta=100 and gene sets meeting min_precent_annot=0.0025.

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets.
- z-score modeling effectively assumes the prior distribution of true effect sizes depends on the standard errors and sample size.
- DE analysis performed on inferred (rather than known) cell types may lead to inflated DE test statistics with artificially smaller standard errors, biasing simulated-power conclusions.

## Evidence

- [intro] Gene-level power gain rationale: "Analyzing one gene at a time can lead to potential power loss, as this approach fails to exploit consistent DE evidence across similar genes"
- [abstract] Headline power gain claim: "iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods"
- [methods] Filter parameter definition: "min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the number of tested genes."
- [readme] iDEA installation / availability: "devtools::install_github('xzhoulab/iDEA')"
- [readme] Simulation framework availability: [evidence span withheld — anchor status: not_found]
