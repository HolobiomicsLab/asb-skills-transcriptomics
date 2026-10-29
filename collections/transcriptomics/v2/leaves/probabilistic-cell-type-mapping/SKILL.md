---
name: probabilistic-cell-type-mapping
description: Use when the input is a multi-cellular pixel-resolution ST count matrix
  (e.
when_to_use_negative:
- No suitable single-cell reference exists, or the reference suffers from batch effects,
  missing cell types, or perturbation-related mismatch with the ST data — in that
  case the reference-free LDA approach (STdeconvolve) is preferred, as reference-based
  methods rely on reference availability
- Input is single-cell-resolution or sub-cellular-resolution ST data rather than multi-cellular
  pixels — STdeconvolve and this benchmarking design are not suited to sub-cellular
  resolution data
- Cell-type proportions are homogeneous/uniform across pixels, or the ST data is too
  sparse with high stochastic drop-out rates — LDA-based deconvolution may fail to
  distinguish cell types and reference-free comparison is not meaningful
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STdeconvolve
  role: Reference-free LDA-based deconvolution used as the unsupervised comparator
    whose RMSE is benchmarked against the supervised reference-based methods
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: SPOTlight
  role: Supervised deconvolution method that uses cell-type marker genes from the
    scRNA-seq reference to seed a non-negative matrix factorization to estimate per-pixel
    proportions
- name: RCTD
  role: Supervised method that builds a probabilistic model from cell-type-specific
    mean expression of reference marker genes to estimate per-pixel proportions
- name: spatialDWLS
  role: Supervised method that enriches for likely cell types per pixel using reference
    signature genes and then applies dampened weighted least squares
- name: remotes
  role: R package used to install STdeconvolve from GitHub for the comparison workflow
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Installer used to obtain STdeconvolve via Bioconductor as an alternative installation
    path
  repo: https://github.com/JEFworks-Lab/STdeconvolve
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:0fc13a9317c9f4860770fb62e2bb2c98a77a9f7b01d0ad887a3e65f2bb7dd3da
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# probabilistic-cell-type-mapping

## Summary

Deconvolves multi-cellular pixel-resolution spatial transcriptomics data into per-pixel cell-type proportions using supervised, single-cell-reference-based methods (SPOTlight NMF seeding, RCTD's probabilistic marker-gene model, spatialDWLS's signature-gene enrichment plus dampened weighted least squares), and benchmarks them against the reference-free LDA-based STdeconvolve using RMSE against known ground-truth proportions.

## When to use

Use when the input is a multi-cellular pixel-resolution ST count matrix (e.g., simulated 100 um2 MPOA MERFISH-derived pixels) with a matched scRNA-seq/MERFISH single-cell reference available, and you need per-pixel cell-type proportions or an RMSE benchmark of reference-based vs reference-free deconvolution.

## When NOT to use

- No suitable single-cell reference exists, or the reference suffers from batch effects, missing cell types, or perturbation-related mismatch with the ST data — in that case the reference-free LDA approach (STdeconvolve) is preferred, as reference-based methods rely on reference availability
- Input is single-cell-resolution or sub-cellular-resolution ST data rather than multi-cellular pixels — STdeconvolve and this benchmarking design are not suited to sub-cellular resolution data
- Cell-type proportions are homogeneous/uniform across pixels, or the ST data is too sparse with high stochastic drop-out rates — LDA-based deconvolution may fail to distinguish cell types and reference-free comparison is not meaningful

## Inputs

- Multi-cellular pixel-resolution spatial transcriptomics count matrix (pixels x genes), e.g., simulated 100 um2 MPOA ST pixel data
- Matched single-cell transcriptomics reference (scRNA-seq or MERFISH) with cell-type labels for the same tissue (mouse preoptic area)
- Ground-truth per-pixel cell-type proportions (for simulated data, used for RMSE evaluation)
- Pixel spatial coordinates for visualization (e.g., mOB$pos-style input)

## Outputs

- Per-pixel cell-type proportion matrices (theta) for each method (SPOTlight, RCTD, spatialDWLS, STdeconvolve)
- RMSE comparison table of deconvolved proportions vs ground truth (e.g., rmse_comparison_table.csv)
- Deconvolved cell-type transcriptional profiles (beta) from STdeconvolve's LDA

## How to apply

Load the pixel count matrix (with known ground-truth proportions if simulated, e.g., the 100 um2 mouse preoptic area MPOA data) and the matched single-cell transcriptomics reference. Fit SPOTlight, which uses cell-type marker genes from the scRNA-seq reference to seed a non-negative matrix factorization; fit RCTD, which builds a probabilistic model from cell-type-specific mean expression of reference marker genes; and fit spatialDWLS, which first enriches for cell types likely in each pixel using signature genes and then applies dampened weighted least squares. Run STdeconvolve's LDA pipeline as the unsupervised comparator (cleanCounts with min.lib.size=100, restrictCorpus for overdispersed-gene feature selection, fitLDA over K values, optimalModel, getBetaTheta). Compare each method's estimated per-pixel proportions to the known simulated ground-truth proportions by computing root-mean-square error (RMSE). Tabulate the RMSE values for all methods (e.g., rmse_comparison_table.csv); the expected result is that STdeconvolve's RMSE is comparable to SPOTlight, RCTD, and spatialDWLS when the ideal matched reference is used, reproducing Fig. 2e, f.

## Related tools

- **STdeconvolve** (Reference-free LDA-based deconvolution used as the unsupervised comparator whose RMSE is benchmarked against the supervised reference-based methods) — https://github.com/JEFworks-Lab/STdeconvolve
- **SPOTlight** (Supervised deconvolution method that uses cell-type marker genes from the scRNA-seq reference to seed a non-negative matrix factorization to estimate per-pixel proportions)
- **RCTD** (Supervised method that builds a probabilistic model from cell-type-specific mean expression of reference marker genes to estimate per-pixel proportions)
- **spatialDWLS** (Supervised method that enriches for likely cell types per pixel using reference signature genes and then applies dampened weighted least squares)
- **remotes** (R package used to install STdeconvolve from GitHub for the comparison workflow) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Installer used to obtain STdeconvolve via Bioconductor as an alternative installation path) — https://github.com/JEFworks-Lab/STdeconvolve

## Examples

```
require(remotes); remotes::install_github('JEFworks-Lab/STdeconvolve')
```

## Evaluation signals

- The RMSE comparison table contains rows for SPOTlight, RCTD, spatialDWLS, and STdeconvolve, each scored against the known simulated ground-truth proportions
- STdeconvolve's RMSE is comparable to (not substantially worse than) SPOTlight, RCTD, and spatialDWLS when the ideal matched MERFISH single-cell reference is used, consistent with Fig. 2e, f
- Per-pixel proportions from each method are valid proportions (rows sum to ~1, non-negative), as expected for theta from LDA and constrained reference-based deconvolutions
- STdeconvolve pipeline intermediates are sane: cleanCounts removes pixels with too few genes (min.lib.size=100) and restrictCorpus feature-selects overdispersed genes before fitLDA over a range of K
- Deconvolved LDA beta profiles are on a comparable scale (getBetaTheta with perc.filt=0.05, betaScale=1000) so inferred cell-type programs can be matched to reference cell types

## Limitations

- Reference-based deconvolution is limited by batch effects, inter- and intra-sample heterogeneity, missing or inconsistent cell types, and different perturbations between the scRNA-seq reference and the ST dataset
- LDA performance depends on dataset size in terms of number of pixels and number of genes; thin structures covering only a few pixels present challenges
- If genes lack variability across pixels (homogeneous cell-type proportions) or the data are too sparse with high stochastic drop-out, STdeconvolve may fail to deconvolve distinct cell types

## Evidence

- [intro] RCTD builds a probabilistic model from reference marker genes: "RCTD4 uses the cell-type-specific mean expression of marker genes derived from a scRNA-seq reference to build a probabilistic model"
- [intro] SPOTlight seeds NMF with reference-derived marker genes: "SPOTlight3 uses cell-type marker genes derived from a single-cell RNA-sequencing (scRNA-seq) reference to seed a non-negative matrix factorization."
- [intro] spatialDWLS enriches cell types then applies DWLS: "SpatialDWLS5 uses cell-type signature genes derived from a scRNA-seq reference to first enrich for cell types likely to be in each pixel"
- [abstract] STdeconvolve comparable to reference-based methods: "STdeconvolve provides comparable performance to existing reference-based methods when suitable single-cell references are available"
- [intro] Reference-based approaches depend on reference availability: "As such, these approaches rely on the availability of a suitable single-cell reference, which may present limitations if such a reference does not exist"
- [readme] STdeconvolve installation via remotes: "require(remotes)
remotes::install_github('JEFworks-Lab/STdeconvolve')"
