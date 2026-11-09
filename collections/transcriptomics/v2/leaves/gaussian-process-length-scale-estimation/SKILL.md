---
name: gaussian-process-length-scale-estimation
description: Use when spot-level spatial transcriptomics data (e.g., Visium DLPFC
  counts plus coordinates) must be screened for SVGs whose spatial expression ranges
  differ, or when per-gene length scale parameters are the analysis target.
when_to_use_negative:
- The goal is to identify spatial domain marker genes (mean-level changes between
  domains) rather than genes varying continuously across the tissue — nnSVG targets
  SVGs, not domain markers
- Genes must be grouped into metagenes or gene groups — nnSVG identifies individual
  SVGs but does not group them
- A fixed, non-length-scale covariance test suffices and runtime at transcriptome
  scale with >=10,000 spots cannot exceed minutes (nnSVG may need several hours on
  10 cores of an HPC cluster)
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: nnSVG
  role: R/Bioconductor package that fits per-gene nearest-neighbor Gaussian process
    models via BRISC, estimating length scale phi, LR statistics, and adjusted p-values
    stored in rowData
  repo: https://bioconductor.org/packages/nnSVG
- name: Bioconductor
  role: Framework hosting nnSVG and the SpatialExperiment input object class whose
    rowData holds the fitted length scale results
- name: SpatialExperiment
  role: Bioconductor container for spot-level counts and spatial coordinates; nnSVG
    results are stored in its rowData
  repo: https://bioconductor.org/packages/SpatialExperiment
- name: BRISC
  role: Underlying algorithm used by nnSVG for fast nearest-neighbor Gaussian process
    model fitting and parameter estimation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:ee35a6ee8f29337d6fca716af416e6903b87f291f55d029b156ea9227ba64f45
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BRISC
  - Bioconductor
  - SpatialExperiment
  - nnSVG
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gaussian-process-length-scale-estimation

## Summary

Fits a nearest-neighbor Gaussian process model per gene in spatially-resolved transcriptomics data to estimate a gene-specific length scale parameter (phi) alongside LR-test statistics, enabling ranking of SVGs by their spatial range of variation.

## When to use

Use when spot-level spatial transcriptomics data (e.g., Visium DLPFC counts plus coordinates) must be screened for SVGs whose spatial expression ranges differ, or when per-gene length scale parameters are the analysis target.

## When NOT to use

- The goal is to identify spatial domain marker genes (mean-level changes between domains) rather than genes varying continuously across the tissue — nnSVG targets SVGs, not domain markers
- Genes must be grouped into metagenes or gene groups — nnSVG identifies individual SVGs but does not group them
- A fixed, non-length-scale covariance test suffices and runtime at transcriptome scale with >=10,000 spots cannot exceed minutes (nnSVG may need several hours on 10 cores of an HPC cluster)

## Inputs

- Spot-level gene count matrix plus spatial coordinates as a Bioconductor SpatialExperiment object (e.g., Visium_humanDLPFC() from STexampleData, 33538 genes x 4992 spots before QC)
- Optionally, a numeric matrix of log-transformed normalized counts (logcounts) and a numeric matrix of spatial coordinates

## Outputs

- rowData columns of the SpatialExperiment: gene_id, gene_name, sigma.sq, tau.sq, phi (length scale), loglik, LR_stat, rank, pval, padj, prop_sv
- Table of gene, estimated length scale l, and threshold flag (e.g., l < 0.1 vs l >= 0.1), e.g., written to length_scale_results.csv
- List of significant SVGs at padj <= 0.05 ranked by LR_stat

## How to apply

Load spot-level counts and spatial coordinates into a SpatialExperiment object, keep spots over tissue (in_tissue == 1), and run the nnSVG filter_genes function to remove mitochondrial genes and genes with fewer than 3 counts in at least 0.5% of spatial locations. Compute logcounts with scran's computeLibraryFactors and logNormCounts, then run nnSVG (set.seed for reproducibility), which fits the nearest-neighbor Gaussian process via the BRISC algorithm per gene, estimating sigma.sq, tau.sq, the length scale parameter phi, loglik, and the likelihood-ratio statistic with approximate p-values and padj. Results are stored in rowData of the SpatialExperiment; extract gene_name, phi, LR_stat, rank, and padj per gene. In nnSVG's parameterization, smaller phi values correspond to longer spatial ranges and larger phi values to shorter spatial ranges, so genes can be classified by estimated length scale (e.g., l-hat < 0.1 vs >= 0.1 on a rescaled axis) to compare blood/immune SVGs (HBB, IGKC, NPY) against cortical-layer SVGs (MOBP, PCP4, SNAP25). Filtering low-count genes both reduces runtime and improves p-value calibration for the remaining genes.

## Related tools

- **nnSVG** (R/Bioconductor package that fits per-gene nearest-neighbor Gaussian process models via BRISC, estimating length scale phi, LR statistics, and adjusted p-values stored in rowData) — https://bioconductor.org/packages/nnSVG
- **Bioconductor** (Framework hosting nnSVG and the SpatialExperiment input object class whose rowData holds the fitted length scale results)
- **SpatialExperiment** (Bioconductor container for spot-level counts and spatial coordinates; nnSVG results are stored in its rowData) — https://bioconductor.org/packages/SpatialExperiment
- **BRISC** (Underlying algorithm used by nnSVG for fast nearest-neighbor Gaussian process model fitting and parameter estimation)

## Examples

```
spe <- filter_genes(spe); spe <- computeLibraryFactors(spe); spe <- logNormCounts(spe); set.seed(123); spe <- nnSVG(spe, n_threads = 1); rowData(spe)
```

## Evaluation signals

- All six known SVGs rank at the top by LR_stat: MOBP (3679.46), SNAP25 (1942.99), IGKC (1289.78), HBB (926.05), PCP4 (894.69), NPY (273.28) with padj = 0 in the README example
- Blood/immune SVGs (HBB, IGKC, NPY) show estimated length scale l-hat < 0.1 while cortical-layer SVGs (MOBP, PCP4, SNAP25) show l-hat >= 0.1; in raw phi, IGKC (20.1), HBB (27.8), NPY (71.7) have larger phi than MOBP (1.10), SNAP25 (3.03), PCP4 (8.27)
- Row lengths match expectations after filtering: 33538 x 4992 loaded, 3639 spots kept over tissue, and 33525 genes reduced after filter_genes removes 13 mitochondrial and 30216 low-expression genes
- table(rowData(spe)$padj <= 0.05) yields 9 significant SVGs among the 16 tested genes in the README workflow
- set.seed(123) is set before both gene subsetting and nnSVG() so results are reproducible run-to-run

## Limitations

- Small negative LR statistics can occur, mainly for lower-ranked genes, and are difficult to interpret
- P-value calibration can be poor for low-expressed genes; adaptive filtering thresholds to fix this are not yet implemented
- The effect size prop_sv does not distinguish technical from biological variance
- Multiple-sample handling averages ranks across samples rather than using a principled joint statistical model
- Transcriptome-wide datasets with >=10,000 spatial locations require runtimes of several hours with 10 processor cores on an HPC cluster
- Some reproducibility details (QC metric thresholds) live only in the package documentation and code repository, not the manuscript

## Evidence

- [abstract] nnSVG uses gene-specific estimates of length scale parameters within the Gaussian process models: [evidence span withheld — anchor status: not_found]
- [abstract] nnSVG scales linearly with the number of spatial locations: "nnSVG scales linearly with the number of spatial locations"
- [discussion] Runtime can be reduced with more stringent gene filtering: "Since runtime depends on the number of genes, this can be reduced with more stringent gene filtering"
- [readme] filter_genes defaults: mitochondrial removal and >=3 counts in >=0.5% of spots: [evidence span withheld — anchor status: external_doc]
- [readme] Outputs stored in rowData of SpatialExperiment: "In this case, the outputs are stored in the `rowData` of the `SpatialExperiment` object."
- [readme] Main results: LR_stat, rank, pval, padj, prop_sv: [evidence span withheld — anchor status: external_doc]
- [discussion] Small negative LR statistics for lower-ranked genes are hard to interpret: "we observe some small negative values in the estimated LR statistics, which are difficult to interpret"
