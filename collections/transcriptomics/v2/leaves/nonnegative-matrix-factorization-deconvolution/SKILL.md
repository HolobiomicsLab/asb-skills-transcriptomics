---
name: nonnegative-matrix-factorization-deconvolution
description: Use when estimating cell-type proportions from multi-cellular pixel-resolution
  ST data (e.g. simulated 100 µm² MPOA MERFISH-derived pixels) and comparing supervised
  (SPOTlight, RCTD, spatialDWLS) vs reference-free (STdeconvolve) deconvolution accuracy
  by RMSE.
when_to_use_negative:
- Data is sub-cellular pixel-resolution ST; the paper states new methods specifically
  suited to that resolution are needed and STdeconvolve may not be best suited
- Tissue is a sliver or thin structure covering only a few pixels, or cell-type proportions
  are homogeneous/uniform across pixels so genes lack variability
- ST gene expression is too sparse with high stochastic drop-out rates, which makes
  the LDA model struggle to identify distinct groups of co-expressed genes
tools:
- name: SPOTlight
  role: 'supervised deconvolution: seeds a non-negative matrix factorization with
    cell-type marker genes derived from the scRNA-seq reference'
- name: RCTD
  role: 'supervised deconvolution: builds a probabilistic model from cell-type-specific
    mean expression of reference marker genes'
- name: spatialDWLS
  role: 'supervised deconvolution: enriches likely cell types per pixel using signature
    genes, then applies dampened weighted least squares'
- name: STdeconvolve
  role: reference-free LDA-based deconvolution baseline and benchmarking framework;
    provides cleanCounts, restrictCorpus, fitLDA, getBetaTheta, vizAllTopics
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: installs STdeconvolve from Bioconductor (requires R >= 4.2 for release 3.15)
- name: remotes
  role: installs STdeconvolve from GitHub
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:58e35181f1c63053224a696cdf9b1d474298c9707178f2e4c49f8abebba3dcd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - RCTD
  - SPOTlight
  - STdeconvolve
  - remotes
  - spatialDWLS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# nonnegative-matrix-factorization-deconvolution

## Summary

Deconvolve multi-cellular pixel-resolution spatial transcriptomics (ST) data into per-pixel cell-type proportions using non-negative matrix factorization seeded by scRNA-seq marker genes (SPOTlight), and benchmark it against other supervised methods (RCTD, spatialDWLS) and the reference-free LDA approach STdeconvolve by computing RMSE against known ground-truth proportions.

## When to use

Use when estimating cell-type proportions from multi-cellular pixel-resolution ST data (e.g. simulated 100 µm² MPOA MERFISH-derived pixels) and comparing supervised (SPOTlight, RCTD, spatialDWLS) vs reference-free (STdeconvolve) deconvolution accuracy by RMSE.

## When NOT to use

- Data is sub-cellular pixel-resolution ST; the paper states new methods specifically suited to that resolution are needed and STdeconvolve may not be best suited
- Tissue is a sliver or thin structure covering only a few pixels, or cell-type proportions are homogeneous/uniform across pixels so genes lack variability
- ST gene expression is too sparse with high stochastic drop-out rates, which makes the LDA model struggle to identify distinct groups of co-expressed genes

## Inputs

- multi-cellular pixel-resolution ST count matrix (e.g. simulated 100 µm² MPOA pixel data with ground-truth cell-type proportions)
- matched single-cell transcriptomics reference (mouse preoptic area MERFISH scRNA-seq with cell-type labels)
- pixel spatial coordinates for visualization (e.g. vizAllTopics)

## Outputs

- per-pixel cell-type proportion matrix (theta) per method
- cell-type transcriptional profiles (beta) for reference-free LDA
- RMSE comparison table (rmse_comparison_table.csv) with rows for SPOTlight, RCTD, SpatialDWLS, and STdeconvolve
- spatial proportion plots (vizAllTopics)

## How to apply

Load the pixel-level count matrix (with known ground-truth proportions for simulated data) and a matched single-cell reference, e.g. mouse preoptic area MERFISH scRNA-seq. Fit SPOTlight, which seeds a non-negative matrix factorization with cell-type marker genes derived from the scRNA-seq reference, to estimate per-pixel proportions; fit RCTD, which builds a probabilistic model from cell-type-specific mean expression of reference marker genes; and fit spatialDWLS, which enriches likely cell types per pixel using signature genes then applies dampened weighted least squares. For the reference-free baseline, run STdeconvolve's LDA workflow: cleanCounts (min.lib.size = 100), restrictCorpus for overdispersed gene feature selection (removeAbove=1.0, removeBelow=0.05), fitLDA over a K grid, select the optimal model, and extract theta (proportions) and beta (transcriptional profiles). The decision point is choosing K a priori for LDA, guided by STdeconvolve's data-driven metrics. Compute root-mean-square error (RMSE) of estimated vs ground-truth proportions for each method and tabulate (e.g. rmse_comparison_table.csv), reproducing the Fig. 2e, f comparison.

## Related tools

- **SPOTlight** (supervised deconvolution: seeds a non-negative matrix factorization with cell-type marker genes derived from the scRNA-seq reference)
- **RCTD** (supervised deconvolution: builds a probabilistic model from cell-type-specific mean expression of reference marker genes)
- **spatialDWLS** (supervised deconvolution: enriches likely cell types per pixel using signature genes, then applies dampened weighted least squares)
- **STdeconvolve** (reference-free LDA-based deconvolution baseline and benchmarking framework; provides cleanCounts, restrictCorpus, fitLDA, getBetaTheta, vizAllTopics) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (installs STdeconvolve from Bioconductor (requires R >= 4.2 for release 3.15))
- **remotes** (installs STdeconvolve from GitHub)

## Examples

```
library(STdeconvolve); counts <- cleanCounts(cd, min.lib.size = 100); corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow = 0.05); ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1))
```

## Evaluation signals

- rmse_comparison_table.csv contains RMSE rows for all four methods: SPOTlight, RCTD, SpatialDWLS, and STdeconvolve
- STdeconvolve's RMSE is comparable to the reference-based methods when the ideal matched MERFISH single-cell reference is used, consistent with the paper's Fig. 2e, f finding
- Estimated proportions are non-negative and interpretable as per-pixel cell-type fractions; LDA-inferred theta rows align with the pixel IDs of the input count matrix
- All methods are scored on the same simulated pixels against the same ground-truth proportions using the same RMSE metric
- LDA model selection is justified by STdeconvolve's data-driven metrics for choosing K (e.g. 'min' criterion via optimalModel)

## Limitations

- LDA performance depends on dataset size in terms of number of pixels and number of genes, which may limit deconvolution when assumptions are not satisfied
- If genes do not exhibit variability across pixels due to homogeneous or uniform proportional representation of cell types, STdeconvolve may fail to deconvolve
- Sparse ST data with high stochastic drop-out rates can hinder LDA from identifying distinct groups of co-expressed genes
- Supervised approaches rely on availability of a suitable single-cell reference; reference-free is preferable when such a reference does not exist
- Number of cells per pixel depends on cell size, which varies by organism, tissue, and disease state

## Evidence

- [intro] SPOTlight seeds NMF with reference markers: "SPOTlight3 uses cell-type marker genes derived from a single-cell RNA-sequencing (scRNA-seq) reference to seed a non-negative matrix factorization."
- [intro] STdeconvolve applies LDA to infer proportions: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [abstract] Comparable performance when references exist: "STdeconvolve provides comparable performance to existing reference-based methods when suitable single-cell references are available"
- [readme] README scope statement: "`STdeconvolve` enables reference-free cell-type deconvolution of multi-cellular pixel-resolution spatial transcriptomics data."
- [intro] spatialDWLS enrichment then DWLS: "SpatialDWLS5 uses cell-type signature genes derived from a scRNA-seq reference to first enrich for cell types likely to be in each pixel"
