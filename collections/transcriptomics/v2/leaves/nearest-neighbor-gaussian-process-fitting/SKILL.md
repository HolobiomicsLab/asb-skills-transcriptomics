---
name: nearest-neighbor-gaussian-process-fitting
description: Use when you need to rank spatially variable genes in a spatial transcriptomics
  dataset (SpatialExperiment or coordinates + logcounts matrix), especially with thousands+
  spots or within a priori defined spatial domains, and require a flexible per-gene
  length scale parameter.
when_to_use_negative:
- The goal is to identify spatial domain marker genes (mean-level changes between
  domains) rather than SVGs with continuous spatial variation
- Grouping SVGs into gene groups or metagenes is required — nnSVG identifies individual
  SVGs but does not group them
- A principled multi-sample statistical model is needed — nnSVG is developed for a
  single sample at a time and handles multiple samples only by rank averaging
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: nnSVG
  role: R/Bioconductor package implementing NNGP-based SVG detection and LR-test ranking;
    results stored in rowData of the SpatialExperiment
  repo: https://bioconductor.org/packages/nnSVG
- name: BRISC
  role: Algorithm used by nnSVG for model fitting and parameter estimation in the
    nearest-neighbor Gaussian process models
- name: STexampleData
  role: Bioconductor data package supplying example SRT datasets such as Visium_humanDLPFC
    for the workflow
- name: scran
  role: Provides computeLibraryFactors and logNormCounts for generating the logcounts
    input required by nnSVG
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:dc31bcacf58b8c7ca9c720babed52599b615b6aef56932de83959a92decb96cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BRISC
  - Bioconductor
  - SPARK-X
  - STexampleData
  - nnSVG
  - scran
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# nearest-neighbor-gaussian-process-fitting

## Summary

Fits gene-specific nearest-neighbor Gaussian process (NNGP) models to spatially-resolved transcriptomics data via the BRISC algorithm to identify and rank spatially variable genes (SVGs) using likelihood-ratio (LR) tests, with runtime scaling linearly in the number of spatial locations.

## When to use

Use when you need to rank spatially variable genes in a spatial transcriptomics dataset (SpatialExperiment or coordinates + logcounts matrix), especially with thousands+ spots or within a priori defined spatial domains, and require a flexible per-gene length scale parameter.

## When NOT to use

- The goal is to identify spatial domain marker genes (mean-level changes between domains) rather than SVGs with continuous spatial variation
- Grouping SVGs into gene groups or metagenes is required — nnSVG identifies individual SVGs but does not group them
- A principled multi-sample statistical model is needed — nnSVG is developed for a single sample at a time and handles multiple samples only by rank averaging

## Inputs

- SpatialExperiment Bioconductor object with counts assay and spatial coordinates (e.g. Visium_humanDLPFC, ST_mouseOB, seqFISH_mouseEmbryo, SlideSeqV2_mouseHPC)
- Alternatively, numeric matrix of log-transformed normalized counts (logcounts) plus numeric matrix of spatial coordinates
- Optional covariate matrix of spatial domain labels for domain-level SVG detection

## Outputs

- rowData columns on the SpatialExperiment: gene_id, gene_name, sigma.sq, tau.sq, phi (length scale), loglik, loglik_lm, LR_stat, rank, pval, padj, prop_sv
- Ranked list of SVGs (select top-n by rank or padj <= 0.05) suitable for downstream spatially-aware clustering
- Per-gene results table exportable to CSV (e.g. nnsvg_visium_dlpfc_results.csv)

## How to apply

Load the data as a Bioconductor SpatialExperiment (e.g. Visium_humanDLPFC from STexampleData), keep spots over tissue, run the nnSVG filter_genes function to remove mitochondrial genes and genes with fewer than 3 counts in at least 0.5% of spatial locations, then compute logcounts (log-transformed normalized counts) with scran's computeLibraryFactors and logNormCounts. Invoke nnSVG on the SpatialExperiment with a fixed seed and set n_threads; the function fits per-gene NNGP models using BRISC, estimating a gene-specific length scale parameter (phi), and runs an LR test against a linear model (loglik_lm) per gene. Results are stored in rowData with columns LR_stat, rank, pval, padj, and prop_sv (proportion of spatial variance effect size); select significant SVGs at padj <= 0.05 or an arbitrary top-n set (e.g. top 100). For domain-level variation, include a priori spatial domain labels as covariates in the linear model (as done for Slide-seqV2 mouse HPC CA3 recovery). Runtime depends on the number of genes and locations, so more stringent gene filtering reduces runtime; the method scales linearly with the number of spatial locations.

## Related tools

- **nnSVG** (R/Bioconductor package implementing NNGP-based SVG detection and LR-test ranking; results stored in rowData of the SpatialExperiment) — https://bioconductor.org/packages/nnSVG
- **BRISC** (Algorithm used by nnSVG for model fitting and parameter estimation in the nearest-neighbor Gaussian process models)
- **STexampleData** (Bioconductor data package supplying example SRT datasets such as Visium_humanDLPFC for the workflow)
- **scran** (Provides computeLibraryFactors and logNormCounts for generating the logcounts input required by nnSVG)

## Examples

```
set.seed(123); spe <- nnSVG(spe, n_threads = 1); rowData(spe)[order(rowData(spe)$rank)[1:10], ]
```

## Evaluation signals

- On Visium human DLPFC: reproduce 2198 significant SVGs at padj <= 0.05 and recovery of 133 of 134 layer-specific marker genes, with MOBP, PCP4, SNAP25, HBB, IGKC, NPY in the top ranks (MOBP rank 1 in the README example)
- Known SVGs should appear with padj = 0 and large LR_stat values (e.g. MOBP LR_stat ≈ 3679, prop_sv ≈ 0.79 in the README vignette)
- Runtime versus number of spatial locations should show a near-linear trend when timed across subsampled spot grids (n = 200–3639 Visium spots; n = 1000–53208 Slide-seqV2 beads)
- Schema check: rowData must contain LR_stat, rank, pval, padj, and prop_sv columns after running nnSVG; ranks must be a permutation of 1..nrow after ordering
- Cross-method sanity: on Slide-seqV2 mouse HPC with spatial domain covariates, expect ~27 of 74 known CA3 SVGs recovered, outperforming SPARK-X/Moran's I (23/74) and HVGs (3/74)

## Limitations

- Some small negative values can appear in estimated LR statistics, which are difficult to interpret; this occurs mainly for lower-ranked genes
- P-value calibration for low-expressed genes could be improved; adaptive filtering thresholds have not yet been implemented
- Transcriptome-wide datasets with >=10,000 spatial locations require runtimes of several hours on 10 processor cores of a high-performance compute cluster
- Effect size (prop_sv) does not distinguish technical from biological variance
- QC metric thresholds are documented only in the package/code repository, not in the manuscript text

## Evidence

- [abstract] NNGP scalability claim: "we propose nnSVG, a scalable approach to identify spatially variable genes based on nearest-neighbor Gaussian processes"
- [abstract] Gene-specific length scale: "uses gene-specific estimates of length scale parameters within the Gaussian process models"
- [readme] Filtering thresholds: [evidence span withheld — anchor status: external_doc]
- [readme] Result columns: "LR_stat`: likelihood ratio (LR) statistics used to rank SVGs
- `rank`: rank of top SVGs according to LR statistics"
- [discussion] Selection strategy: [evidence span withheld — anchor status: not_found]
