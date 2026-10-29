---
name: empirical-power-estimation-at-fixed-fdr
description: Use when comparing detection rates of gene set enrichment methods on
  simulated summary statistics where the truth (which replicates are enriched) is
  known and power must be computed at a fixed FDR threshold (e.g. 5%).
when_to_use_negative:
- Input is real scRNA-seq data with no known ground truth about which gene sets are
  enriched — power cannot be computed and only calibration/replication checks apply.
- The analysis goal is calibration of gene set p-values or FDR control rather than
  detection-rate comparison at a fixed FDR.
- Summary statistics were produced under a different enrichment regime (e.g. different
  tau1 or coverage rate), so baseline-scenario expected powers (98/26/0/8/26%) do
  not apply.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: iDEA
  role: Fits the integrative DE+GSE model per replicate via iDEA.fit and computes
    Louis-calibrated gene set p-values via iDEA.louis under the baseline alternative
    (init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, modelVariant=FALSE)
  repo: https://github.com/xzhoulab/iDEA
- name: fGSEA
  role: Competing GSE method applied to the identical simulated summary statistics
    of every replicate for power comparison at FDR 5%
- name: CAMERA
  role: Competing GSE method applied to the identical simulated summary statistics
    of every replicate for power comparison at FDR 5%
- name: PAGE
  role: Competing GSE method applied to the identical simulated summary statistics
    of every replicate for power comparison at FDR 5%
- name: GSEA
  role: Competing GSE method applied to the identical simulated summary statistics
    of every replicate for power comparison at FDR 5%
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:0b3a70d56181fe88a429cfc5e1f8a6d691b79496ec3d22548272df2243ad1d0b
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# empirical-power-estimation-at-fixed-fdr

## Summary

Benchmark empirical gene-set-enrichment power by simulating gene-level DE summary statistics under a known enrichment alternative (tau0=-2, tau1=0.5, CR=10%), running competing GSE methods (iDEA, fGSEA, CAMERA, PAGE, GSEA) on identical replicates, and calling significance at a fixed 5% FDR.

## When to use

Use when comparing detection rates of gene set enrichment methods on simulated summary statistics where the truth (which replicates are enriched) is known and power must be computed at a fixed FDR threshold (e.g. 5%).

## When NOT to use

- Input is real scRNA-seq data with no known ground truth about which gene sets are enriched — power cannot be computed and only calibration/replication checks apply.
- The analysis goal is calibration of gene set p-values or FDR control rather than detection-rate comparison at a fixed FDR.
- Summary statistics were produced under a different enrichment regime (e.g. different tau1 or coverage rate), so baseline-scenario expected powers (98/26/0/8/26%) do not apply.

## Inputs

- 10,000 simulated replicates of gene-level DE summary statistics (beta, beta_var) under tau0=-2, tau1=0.5, coverage rate CR=10% (1,000 enriched + 9,000 null replicates)
- Gene set annotations (e.g. humanGeneSets.RData from the iDEA package)

## Outputs

- Empirical power estimates at fixed FDR 5% for each method (iDEA, fGSEA, CAMERA, PAGE, GSEA)
- Tabulated comparison file (e.g. power_comparison_fdr5.csv)
- Per-replicate significant gene set calls and false-positive rates over the 9,000 null replicates

## How to apply

Generate 10,000 simulated replicates of gene-level DE summary statistics (beta, beta_var) under the baseline alternative tau0=-2, tau1=0.5 with a 10% gene set coverage rate, yielding 1,000 enriched (truth-positive) and 9,000 null replicates. Fit iDEA per replicate with iDEA.fit using init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, modelVariant=FALSE (z-score modeling), then compute Louis-method-calibrated gene set p-values with iDEA.louis. Apply fGSEA, CAMERA, PAGE, and GSEA to the identical simulated summary statistics for every replicate. Call gene sets significant at fixed FDR 5% per method and per replicate, and compute empirical power as the detection rate among the 1,000 enriched replicates while using the 9,000 null replicates to verify false-positive control. Tabulate the five power estimates (expected: iDEA 98%, fGSEA 26%, CAMERA 0%, PAGE 8%, GSEA 26%, matching Fig. 3a) to quantify iDEA's power gain over existing GSE approaches.

## Related tools

- **iDEA** (Fits the integrative DE+GSE model per replicate via iDEA.fit and computes Louis-calibrated gene set p-values via iDEA.louis under the baseline alternative (init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, modelVariant=FALSE)) — https://github.com/xzhoulab/iDEA
- **fGSEA** (Competing GSE method applied to the identical simulated summary statistics of every replicate for power comparison at FDR 5%)
- **CAMERA** (Competing GSE method applied to the identical simulated summary statistics of every replicate for power comparison at FDR 5%)
- **PAGE** (Competing GSE method applied to the identical simulated summary statistics of every replicate for power comparison at FDR 5%)
- **GSEA** (Competing GSE method applied to the identical simulated summary statistics of every replicate for power comparison at FDR 5%)

## Examples

```
idea <- iDEA.fit(idea_obj, gene.set = geneSet, init.tau = c(-2, 0.5), min.degene = 5, em.iter = 15, mcmc.iter = 1000, fit.tol = 1e-5, modelVariant = FALSE)
```

## Evaluation signals

- Computed powers at FDR 5% reproduce the baseline-scenario values for all five methods within Monte Carlo error: iDEA 98%, fGSEA 26%, CAMERA 0%, PAGE 8%, GSEA 26% (Fig. 3a).
- False-positive rate on the 9,000 null replicates is controlled at the nominal 5% FDR level for each method.
- Power is computed as detection rate among exactly 1,000 enriched replicates out of 10,000 total (CR = 10%).
- All five methods are run on the identical simulated summary statistics per replicate, ensuring a paired, fair comparison.
- iDEA's gene set p-values are Louis-method-calibrated (via iDEA.louis) before thresholding at FDR 5%.

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets; the paper accounts for GSE test non-independence through permutation of gene labels to estimate FDR.
- Z-score modeling (modelVariant=FALSE) effectively assumes the prior distribution of true effect sizes depends on the standard errors and sample size.
- Results are scenario-specific: expected powers (iDEA 98%, fGSEA 26%, CAMERA 0%, PAGE 8%, GSEA 26%) hold for tau1=0.5, CR=10% at FDR 5% and will differ under other enrichment regimes or thresholds.

## Evidence

- [abstract] iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods: "iDEA achieves up to five-fold power gain over existing GSE methods and up to 64% power gain over existing DE methods"
- [methods] Louis calibrated p-values: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [methods] FDR estimation via permuted null: "The function iDEA.FDR calculate the estimated FDR for each gene set given the permuted null distribution of pvalues for gene sets."
- [readme] Package installation: "devtools::install_github('xzhoulab/iDEA')"
- [discussion] Accounting for gene set overlap: "accounted for GSE test non-independence due to gene set overlap through permutation of gene labels"
