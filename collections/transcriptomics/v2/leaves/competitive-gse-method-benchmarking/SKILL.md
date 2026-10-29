---
name: competitive-gse-method-benchmarking
description: Use when comparing the statistical power of gene set enrichment methods
  on identical simulated DE summary statistics (beta, beta_var) at a fixed FDR threshold,
  e.g. to quantify a new method's gain over fGSEA, CAMERA, PAGE, GSEA.
when_to_use_negative:
- The goal is to compare DE (per-gene) methods rather than competitive gene set enrichment
  methods — iDEA's DE-power benchmarking uses a different design (up to 64% gain over
  existing DE methods).
- Input is individual-level scRNA-seq count data rather than DE summary statistics
  — iDEA's framework consumes summary statistics output from existing DE tools and
  makes no explicit modeling of raw counts.
- There is no null-replicate set to verify false-positive control, so a fixed-FDR
  power comparison cannot be validated.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: iDEA
  role: Integrative DE+GSE method under evaluation; fitted with iDEA.fit and assigned
    Louis-method-calibrated gene set p-values via iDEA.louis on each simulated replicate
  repo: https://github.com/xzhoulab/iDEA
- name: fGSEA
  role: Competing GSE method applied to the identical simulated summary statistics
    for power comparison
- name: CAMERA
  role: Competing competitive-test GSE method applied to the same replicates
- name: PAGE
  role: Competing competitive-test GSE method applied to the same replicates
- name: GSEA
  role: Competing competitive-test GSE method applied to the same replicates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:315a10c9d8b3427abf0de84757561eae128c99f741f42a1b68b6b6633856dc43
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# competitive-gse-method-benchmarking

## Summary

Benchmark gene set enrichment (GSE) methods by simulating gene-level DE summary statistics under an enrichment alternative (tau0=-2, tau1=0.5, CR=10%) and comparing empirical power at a fixed FDR of 5% across iDEA, fGSEA, CAMERA, PAGE, and GSEA.

## When to use

Use when comparing the statistical power of gene set enrichment methods on identical simulated DE summary statistics (beta, beta_var) at a fixed FDR threshold, e.g. to quantify a new method's gain over fGSEA, CAMERA, PAGE, GSEA.

## When NOT to use

- The goal is to compare DE (per-gene) methods rather than competitive gene set enrichment methods — iDEA's DE-power benchmarking uses a different design (up to 64% gain over existing DE methods).
- Input is individual-level scRNA-seq count data rather than DE summary statistics — iDEA's framework consumes summary statistics output from existing DE tools and makes no explicit modeling of raw counts.
- There is no null-replicate set to verify false-positive control, so a fixed-FDR power comparison cannot be validated.

## Inputs

- 10,000 simulated gene-level DE summary statistics replicates (beta, beta_var) with tau0=-2, tau1=0.5 and coverage rate CR=10% (1,000 enriched + 9,000 null replicates)
- Gene set annotations (e.g. humanGeneSets from the iDEA package / MSigDB-derived databases)

## Outputs

- Empirical power estimates at FDR 5% for each of the five methods (expected: iDEA 98%, fGSEA 26%, CAMERA 0%, PAGE 8%, GSEA 26%)
- power_comparison_fdr5.csv tabulating the five power estimates
- Per-replicate gene set significance calls and false-positive rates from the 9,000 null replicates

## How to apply

Generate 10,000 replicates of gene-level DE summary statistics under the baseline enrichment setting tau0=-2, tau1=0.5 with gene set coverage rate CR=10%, comprising 1,000 enriched and 9,000 null replicates. For each replicate, fit iDEA via iDEA.fit with init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, modelVariant=FALSE (z-score modeling), then compute Louis-method-calibrated gene set p-values with iDEA.louis. Apply fGSEA, CAMERA, PAGE, and GSEA to the identical simulated summary statistics of every replicate so that all five methods consume exactly the same inputs. Call gene sets significant at fixed FDR 5% per method and per replicate; empirical power is the detection rate among the 1,000 enriched replicates, while the 9,000 null replicates verify false-positive control. In the baseline scenario the expected powers are iDEA 98%, fGSEA 26%, CAMERA 0%, PAGE 8%, and GSEA 26% (Fig. 3a), i.e. up to roughly a five-fold power gain for iDEA.

## Related tools

- **iDEA** (Integrative DE+GSE method under evaluation; fitted with iDEA.fit and assigned Louis-method-calibrated gene set p-values via iDEA.louis on each simulated replicate) — https://github.com/xzhoulab/iDEA
- **fGSEA** (Competing GSE method applied to the identical simulated summary statistics for power comparison)
- **CAMERA** (Competing competitive-test GSE method applied to the same replicates)
- **PAGE** (Competing competitive-test GSE method applied to the same replicates)
- **GSEA** (Competing competitive-test GSE method applied to the same replicates)

## Examples

```
iDEA.fit(idea_obj, init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, modelVariant=FALSE)
```

## Evaluation signals

- Computed power at FDR 5% for all five methods must reproduce the reported baseline-scenario values (iDEA 98%, fGSEA 26%, CAMERA 0%, PAGE 8%, GSEA 26%) within Monte Carlo error, matching Fig. 3a.
- Empirical power is computed only as the detection rate among the 1,000 enriched replicates, never pooled with null replicates.
- False-positive control must be verified using the 9,000 null replicates for every method before trusting power estimates.
- All five methods must be applied to the identical simulated summary statistics per replicate — differing inputs invalidate the comparison.
- Output file power_comparison_fdr5.csv contains exactly five method-level power values at the fixed FDR=5% threshold.

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets; non-independence is handled only via permutation of gene labels when estimating FDR from a permuted null (iDEA.fit.null / iDEA.FDR).
- Z-score modeling (modelVariant=FALSE) effectively assumes the prior distribution of true effect sizes depends on the standard errors, and subsequently the sample size.
- DE analysis on inferred cell types may lead to inflated DE test statistics with artificially smaller standard errors, which could bias simulation-based benchmarking conclusions.

## Evidence

- [abstract] iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods: "iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods"
- [methods] iDEA Louis p-values: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [intro] GSE method families compared: "competitive test methods such as PAGE13, GSEA14, and CAMERA15"
- [discussion] Calibration of compared GSE methods: "GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values"
- [readme] iDEA installation: "devtools::install_github('xzhoulab/iDEA')"
