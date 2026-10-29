---
name: covariate-adjusted-spatial-modeling
description: Use when analyzing spatially-resolved transcriptomics data (e.g., SpatialExperiment
  objects) where known spatial domain labels exist and you need to detect SVGs adjusted
  for domain structure, or benchmark SVG detection against SPARK-X, HVGs, and Moran's
  I.
when_to_use_negative:
- When the goal is to identify spatial domain marker genes (mean-level changes between
  domains) rather than genes with continuous spatial variation — the authors explicitly
  focus on SVGs, not spatial domain marker genes
- When no spatial domain labels or covariates are available and you only need unadjusted
  whole-tissue SVG detection (run nnSVG without covariates instead)
- When genes have not been filtered for low expression — unfiltered transcriptome-wide
  input will inflate runtime and may miscalibrate p-values for low-expressed genes
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3308
tools:
- name: nnSVG
  role: 'Core method: fits gene-specific length scale parameters in nearest-neighbor
    Gaussian processes (BRISC algorithm) to rank SVGs by LR test adjusted p-values,
    optionally with spatial domain covariates'
  repo: https://bioconductor.org/packages/nnSVG
- name: SPARK-X
  role: 'Comparison method: nonparametric covariance tests for SVG detection benchmarking'
- name: Bioconductor
  role: Framework providing SpatialExperiment and STexampleData for loading and representing
    the SRT datasets
  repo: https://bioconductor.org/packages/nnSVG
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:6e4e2e0218d4e9b90ce8d1a60ca3d2e3ff4124c19fcc58b59f315f7f17947356
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - SPARK-X
  - nnSVG
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# covariate-adjusted-spatial-modeling

## Summary

Fit gene-specific nearest-neighbor Gaussian process models (via nnSVG) with a priori spatial domain labels included as covariates in the linear model, to identify spatially variable genes (SVGs) that vary within known spatial domains and to benchmark recovery against SPARK-X, HVGs, and Moran's I.

## When to use

Use when analyzing spatially-resolved transcriptomics data (e.g., SpatialExperiment objects) where known spatial domain labels exist and you need to detect SVGs adjusted for domain structure, or benchmark SVG detection against SPARK-X, HVGs, and Moran's I.

## When NOT to use

- When the goal is to identify spatial domain marker genes (mean-level changes between domains) rather than genes with continuous spatial variation — the authors explicitly focus on SVGs, not spatial domain marker genes
- When no spatial domain labels or covariates are available and you only need unadjusted whole-tissue SVG detection (run nnSVG without covariates instead)
- When genes have not been filtered for low expression — unfiltered transcriptome-wide input will inflate runtime and may miscalibrate p-values for low-expressed genes

## Inputs

- SpatialExperiment Bioconductor object with raw counts and spatial coordinates
- Alternatively: numeric matrix of log-transformed normalized counts plus numeric matrix of spatial coordinates
- A priori spatial domain labels per location (e.g., known CA3 layer annotations for Slide-seqV2 mouse hippocampus)
- Curated known SVG / marker gene lists for recovery assessment

## Outputs

- rowData of the SpatialExperiment populated with LR_stat, rank, pval, padj, and prop_sv per gene
- Ranked list of significant SVGs (padj <= 0.05) with effect size as proportion of spatial variance
- svg_recovery_metrics.csv with per-method recovery counts (e.g., nnSVG 5/7 ST mouse OB, 7/12 seqFISH embryo, 27/74 Slide-seqV2) and significant-SVG counts

## How to apply

Load the SRT datasets (e.g., ST_mouseOB, seqFISH_mouseEmbryo, SlideSeqV2_mouseHPC) as Bioconductor SpatialExperiment objects, apply QC and gene filtering with nnSVG's filter_genes (removing mitochondrial genes and genes with fewer than 3 counts in at least 0.5% of spatial locations), then compute logcounts with scran (computeLibraryFactors + logNormCounts). Run nnSVG, which fits gene-specific length scale parameters (phi) within nearest-neighbor Gaussian processes using the BRISC algorithm, ranking SVGs by LR test statistics and adjusted p-values. For datasets with a priori spatial domains (e.g., Slide-seqV2 mouse hippocampus CA3), re-run nnSVG with the spatial-domain labels included as covariates in the linear model so genes varying within domains are detected. For comparison, run SPARK-X nonparametric covariance tests, select top HVGs, and compute Moran's I, then intersect each method's ranked gene list with curated known SVG/marker lists to record recovery counts (e.g., nnSVG 27/74 vs SPARK-X/Moran's I 23/74 vs HVGs 3/74 on Slide-seqV2 CA3).

## Related tools

- **nnSVG** (Core method: fits gene-specific length scale parameters in nearest-neighbor Gaussian processes (BRISC algorithm) to rank SVGs by LR test adjusted p-values, optionally with spatial domain covariates) — https://bioconductor.org/packages/nnSVG
- **SPARK-X** (Comparison method: nonparametric covariance tests for SVG detection benchmarking)
- **Bioconductor** (Framework providing SpatialExperiment and STexampleData for loading and representing the SRT datasets) — https://bioconductor.org/packages/nnSVG

## Examples

```
spe <- filter_genes(spe); spe <- computeLibraryFactors(spe); spe <- logNormCounts(spe); set.seed(123); spe <- nnSVG(spe, n_threads = 1)
```

## Evaluation signals

- rowData of the SpatialExperiment contains the expected columns (LR_stat, rank, pval, padj, prop_sv) after running nnSVG
- Known marker genes rank highly: on Slide-seqV2 mouse hippocampus with covariates, nnSVG recovers 27 of 74 known CA3 SVGs, versus 23/74 for SPARK-X and Moran's I and 3/74 for HVGs
- Number of significant SVGs at padj <= 0.05 matches reported counts (e.g., 559 significant SVGs across 4216 spots for ST mouse OB; 1024 across 8883 locations for Slide-seqV2 with covariates)
- Gene filtering report shows mitochondrial genes removed and low-expression genes removed at the default threshold (at least 3 counts in at least 0.5% of spatial locations)
- Logcounts computed via scran library size factors are present in the assays (assayNames includes 'counts' and 'logcounts') before running nnSVG

## Limitations

- Transcriptome-wide datasets with >=10,000 spatial locations require runtimes of several hours with 10 processor cores on a high-performance compute cluster
- Small negative values in estimated LR statistics can occur, mainly for lower-ranked genes, and are difficult to interpret
- P-value calibration for low-expressed genes could be improved; adaptive filtering thresholds are not yet implemented
- Effect size (prop_sv) does not distinguish technical from biological variance
- Model developed for a single sample at a time; multiple-sample handling uses rank averaging rather than a principled statistical approach

## Evidence

- [abstract] nnSVG uses covariates and gene-specific length scales: "identifies genes that vary in expression continuously across the entire tissue or within a priori defined spatial domains"
- [abstract] nnSVG scales linearly: "scales linearly with the number of spatial locations"
- [discussion] SPARK-X limitation: "thereby leading to reduced flexibility to identify SVGs with different spatial ranges in expression"
- [readme] input format and outputs: "the inputs can also be provided as a numeric matrix of normalized and transformed counts (e.g. log-transformed normalized counts, also known as logcounts) and a numeric matrix of spatial coordinates"
- [readme] default filtering thresholds: [evidence span withheld — anchor status: external_doc]
- [readme] results stored in rowData: "The main results of interest are: - `LR_stat`: likelihood ratio (LR) statistics used to rank SVGs"
