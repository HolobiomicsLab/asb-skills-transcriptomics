---
name: empirical-fdr-estimation-by-gene-label-permutation
description: Use when running gene set enrichment on scRNA-seq DE summary statistics
  (e.g. zingeR-DESeq2 beta/beta-variance) and a calibrated, overlap-aware empirical
  FDR (e.g. 5% threshold) is needed instead of nominal p-value cutoffs.
when_to_use_negative:
- When the input is raw scRNA-seq count data rather than per-gene DE summary statistics
  — iDEA consumes beta/beta-variance summaries from tools like zingeR-DESeq2, MAST,
  or SCDE
- When gene sets are strictly non-overlapping and independent, the permutation-based
  FDR machinery is unnecessary overhead
- When performing DE analysis on inferred cell types whose standard errors may be
  artificially small — iDEA's z-score-based prior can then be mis-calibrated
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: iDEA
  role: Provides iDEA.fit.null to build the permuted null by permuting gene labels
    in gene sets and iDEA.FDR to calculate estimated FDR per gene set from that null
  repo: https://github.com/xzhoulab/iDEA
- name: zingeR
  role: Produces the per-gene beta coefficient and beta variance summary statistics
    (via zingeR-DESeq2) on which the enrichment fit and permuted null are computed
  repo: https://github.com/statOmics/zinbwaveZinger
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:4b2e53afd2efca036afc12907e65ef88a99e370897f521d73ad8b4aadd60a68d
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# empirical-fdr-estimation-by-gene-label-permutation

## Summary

Estimate empirical gene-set FDR for iDEA by permuting gene labels in gene sets with iDEA.fit.null to build a permuted null distribution of GSE p-values, then computing FDR with iDEA.FDR; this accounts for test non-independence caused by gene set overlap.

## When to use

Use when running gene set enrichment on scRNA-seq DE summary statistics (e.g. zingeR-DESeq2 beta/beta-variance) and a calibrated, overlap-aware empirical FDR (e.g. 5% threshold) is needed instead of nominal p-value cutoffs.

## When NOT to use

- When the input is raw scRNA-seq count data rather than per-gene DE summary statistics — iDEA consumes beta/beta-variance summaries from tools like zingeR-DESeq2, MAST, or SCDE
- When gene sets are strictly non-overlapping and independent, the permutation-based FDR machinery is unnecessary overhead
- When performing DE analysis on inferred cell types whose standard errors may be artificially small — iDEA's z-score-based prior can then be mis-calibrated

## Inputs

- Fitted iDEA object from CreateiDEAObject + iDEA.fit (per-gene beta and beta variance from zingeR-DESeq2 summary statistics paired with MSigDB-derived human gene sets, e.g. humanGeneSets.RData)
- Human gene set annotations (BioCarta, KEGG, GO, PubChem Compound, ImmuneSigDB, PID, Reactome from MSigDB, GRCh37)

## Outputs

- Permuted null distribution of gene-set p-values (from iDEA.fit.null)
- Per-gene-set empirical FDR estimates (from iDEA.FDR)
- Set of significantly enriched gene sets at empirical FDR 5%
- Genomic control factor lambda_gc for calibration assessment

## How to apply

After fitting the iDEA hierarchical Bayesian model with iDEA.fit (init_tau = c(-2, 0.5), min_degene = 5, em_iter = 15, mcmc_iter = 1000, fit.tol = 1e-5, modelVariant = FALSE) on an iDEA object built by CreateiDEAObject (max_var_beta = 100, min_precent_annot = 0.0025), call iDEA.fit.null to fit the model repeatedly while permuting gene labels within gene sets, constructing the permuted null distribution of gene-set p-values. Then apply iDEA.FDR to estimate, for each gene set, the FDR given that permuted null, and declare gene sets enriched at empirical FDR 5% (e.g. 2,106 sets for iDEA vs 1,742 for fGSEA in the DEC vs EC human ESC comparison). Optionally compute calibrated p-values via iDEA.louis (Louis method) and the lambda_gc genomic control factor (reported values 1.13, 1.02, 0.96, 1.01, 0.29 for iDEA, fGSEA, CAMERA, PAGE, GSEA) to check calibration. The rationale is that permutation of gene labels accounts for GSE test non-independence due to gene set overlap, which ordinary competitive tests ignore.

## Related tools

- **iDEA** (Provides iDEA.fit.null to build the permuted null by permuting gene labels in gene sets and iDEA.FDR to calculate estimated FDR per gene set from that null) — https://github.com/xzhoulab/iDEA
- **zingeR** (Produces the per-gene beta coefficient and beta variance summary statistics (via zingeR-DESeq2) on which the enrichment fit and permuted null are computed) — https://github.com/statOmics/zinbwaveZinger

## Examples

```
library(iDEA); idea <- CreateiDEAObject(gene_name, beta, beta_var, annotation); idea <- iDEA.fit(idea, init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5); idea <- iDEA.fit.null(idea); idea <- iDEA.FDR(idea)
```

## Evaluation signals

- At empirical FDR 5% on the human ESC DEC vs EC data, iDEA should identify ~2,106 enriched gene sets, ~20.9% more than fGSEA's 1,742, within reasonable numerical tolerance
- Computed lambda_gc values should reproduce the published Fig. 4 values (1.13, 1.02, 0.96, 1.01, 0.29 for iDEA, fGSEA, CAMERA, PAGE, GSEA) within tolerance
- Permuted null p-values from iDEA.fit.null should be roughly uniform, since gene labels are randomly permuted
- All quantities (FDR 5% set counts, FDR 1% DE gene counts, Jaccard overlaps 0.10 -> 0.25) should be written to idea_dec_ec_results.csv matching the published validation targets

## Limitations

- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets; the permutation of gene labels is an empirical workaround
- The permuted null requires repeated model fits and can be computationally expensive (multi-core setup with num_core = 10 is recommended)
- The FDR estimate depends on the chosen filter thresholds (min_precent_annot = 0.0025, min_degene = 5) and may not transfer to other gene set databases or annotation coverage rates

## Evidence

- [discussion] accounted for GSE test non-independence due to gene set overlap through permutation of gene labels: "accounted for GSE test non-independence due to gene set overlap through permutation of gene labels"
- [methods] iDEA.fit.null fits model with permuted gene labels: "The function iDEA.fit.null fits iDEA model by permuting gene labels in gene set, thus constructing tbe permuted null distribution for gene sets."
- [methods] iDEA.FDR calculates estimated FDR from permuted null: "The function iDEA.FDR calculate the estimated FDR for each gene set given the permuted null distribution of pvalues for gene sets."
- [methods] Louis method computes calibrated p-values: "iDEA utilizes [Louis method](https://www.jstor.org/stable/2345828) to compute calibrated p-values for testing gene set enrichment"
- [readme] README install command for iDEA package: "devtools::install_github('xzhoulab/iDEA')"
