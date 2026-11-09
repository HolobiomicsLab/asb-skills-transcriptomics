---
name: gene-specific-length-scale-gaussian-process-modeling
description: Use when identifying spatially variable genes in spatially-resolved transcriptomics
  data (e.g., Visium DLPFC, ST mouse OB) where genes may have different spatial ranges
  (length scales) and scalable fitting over thousands of spots is required.
when_to_use_negative:
- The goal is identifying spatial domain marker genes (mean-level changes between
  domains) rather than genes varying continuously across the tissue or within a priori
  defined spatial domains
- Multiple-sample datasets requiring a principled statistical model — nnSVG is developed
  for a single sample at a time and handles multiple samples only via averaging per-sample
  ranks
- Effect sizes must separate technical from biological variance — the prop_sv definition
  does not make this distinction
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: nnSVG
  role: 'Primary method: NNGP-based SVG identification with gene-specific length scale
    estimation and likelihood-ratio test; results stored in rowData of the SpatialExperiment'
  repo: https://bioconductor.org/packages/nnSVG
- name: BRISC
  role: Underlying algorithm for NNGP model fitting and parameter estimation invoked
    by nnSVG
- name: SpatialExperiment
  role: Bioconductor container for input SRT data; nnSVG writes results into its rowData
  repo: https://bioconductor.org/packages/SpatialExperiment
- name: scran
  role: computeLibraryFactors and logNormCounts to produce the logcounts assay used
    as nnSVG input
- name: STexampleData
  role: Source of example datasets such as Visium_humanDLPFC used in the workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:916c7119b52e6c2e05832b9061b5bff85384ab5a33991eabff2e7d5fdf080881
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BRISC
  - Bioconductor
  - STexampleData
  - SpatialExperiment
  - nnSVG
  - scran
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# gene-specific-length-scale-gaussian-process-modeling

## Summary

Fit a nearest-neighbor Gaussian process (NNGP) model per gene with a gene-specific length scale parameter to identify and rank spatially variable genes (SVGs) via a likelihood-ratio test, using nnSVG's BRISC-based fitting that scales linearly with the number of spatial locations.

## When to use

Use when identifying spatially variable genes in spatially-resolved transcriptomics data (e.g., Visium DLPFC, ST mouse OB) where genes may have different spatial ranges (length scales) and scalable fitting over thousands of spots is required.

## When NOT to use

- The goal is identifying spatial domain marker genes (mean-level changes between domains) rather than genes varying continuously across the tissue or within a priori defined spatial domains
- Multiple-sample datasets requiring a principled statistical model — nnSVG is developed for a single sample at a time and handles multiple samples only via averaging per-sample ranks
- Effect sizes must separate technical from biological variance — the prop_sv definition does not make this distinction

## Inputs

- SpatialExperiment Bioconductor object with 'counts' assay and spatial coordinates (e.g., Visium_humanDLPFC from STexampleData)
- Alternatively, a numeric matrix of normalized and transformed counts (logcounts) plus a numeric matrix of spatial coordinates

## Outputs

- rowData columns: LR_stat (likelihood ratio statistics used to rank SVGs), rank, pval, padj, prop_sv (proportion of spatial variance effect size)
- Per-gene model parameters: sigma.sq, tau.sq, phi (length scale), loglik, loglik_lm, spcov, mean, var, runtime

## How to apply

Load a SpatialExperiment object (e.g., Visium_humanDLPFC from STexampleData), keep spots over tissue, apply nnSVG's filter_genes defaults (remove mitochondrial genes; retain genes with at least 3 counts in at least 0.5% of spatial locations), then compute logcounts via scran's computeLibraryFactors and logNormCounts. Run nnSVG on the filtered object: it fits, per gene, an NNGP spatial covariance model through the BRISC algorithm, estimating gene-specific length scale (phi) and variance (sigma.sq, tau.sq) parameters, and compares against a non-spatial linear model (loglik_lm) to compute LR_stat. Rank genes by LR_stat; call significance using pval/padj from the approximate LR test at cutoffs such as 0.05. Read results from rowData(Spe), including prop_sv (proportion of spatial variance) as the effect size. Runtime depends on the number of genes, so more stringent gene filtering reduces runtime, and transcriptome-wide runs with >=10,000 locations need several hours on 10 cores.

## Related tools

- **nnSVG** (Primary method: NNGP-based SVG identification with gene-specific length scale estimation and likelihood-ratio test; results stored in rowData of the SpatialExperiment) — https://bioconductor.org/packages/nnSVG
- **BRISC** (Underlying algorithm for NNGP model fitting and parameter estimation invoked by nnSVG)
- **SpatialExperiment** (Bioconductor container for input SRT data; nnSVG writes results into its rowData) — https://bioconductor.org/packages/SpatialExperiment
- **scran** (computeLibraryFactors and logNormCounts to produce the logcounts assay used as nnSVG input)
- **STexampleData** (Source of example datasets such as Visium_humanDLPFC used in the workflow)

## Examples

```
library(nnSVG); spe <- Visium_humanDLPFC(); spe <- spe[, colData(spe)$in_tissue == 1]; spe <- filter_genes(spe); spe <- computeLibraryFactors(spe); spe <- logNormCounts(spe); set.seed(123); spe <- nnSVG(spe, n_threads = 1)
```

## Evaluation signals

- rowData contains all expected columns: LR_stat, rank, pval, padj, prop_sv, sigma.sq, tau.sq, phi, loglik, loglik_lm
- Known SVGs (e.g., MOBP, PCP4, SNAP25, HBB, IGKC, NPY) rank at the top; in the README example 9 of 16 genes have padj <= 0.05
- Simulation benchmark reproduces false positive rates of 0.003, 0.016, 0.031 at nominal p-value thresholds 0.01, 0.05, 0.1 (conservative behavior)
- Coordinate-permuted nulls yield FPR of ~1.5% (Visium DLPFC) and ~1.0% (ST mouse OB) at cutoff 0.05
- Estimated phi values differ per gene (e.g., MOBP phi=1.10 vs IGKC phi=20.11 in the README example), confirming gene-specific length scales rather than a fixed combination

## Limitations

- Transcriptome-wide datasets with >=10,000 spatial locations require runtimes of several hours with 10 processor cores on a high-performance compute cluster
- Small negative values observed in estimated LR statistics, which are difficult to interpret; occurs mainly for lower-ranked genes
- P-value calibration for low-expressed genes could be improved via adaptive filtering thresholds, which are not yet implemented
- Developed for a single sample at a time; multiple-sample handling uses rank averaging rather than a principled statistical approach
- Effect size (prop_sv) does not distinguish technical from biological variance
- Runtime can be reduced with more stringent gene filtering, but default filtering parameters may still leave low-expressed genes that affect calibration

## Evidence

- [abstract] nnSVG is a scalable NNGP-based SVG method: "we propose nnSVG, a scalable approach to identify spatially variable genes based on nearest-neighbor Gaussian processes"
- [abstract] Gene-specific length scale parameters are a core feature: "uses gene-specific estimates of length scale parameters within the Gaussian process models"
- [discussion] Contrast with SPARK-X fixed length scales motivating the method: [evidence span withheld — anchor status: not_found]
- [readme] Results location and output schema: "The main results of interest are: LR_stat: likelihood ratio (LR) statistics used to rank SVGs"
- [readme] Default gene filtering thresholds: [evidence span withheld — anchor status: external_doc]
- [discussion] Runtime scaling caveat: "runtimes are on the order of several hours when using 10 processor cores on a high-performance compute cluster"
