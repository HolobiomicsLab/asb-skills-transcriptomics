---
name: reference-free-cell-type-proportion-inference
description: Use when the input is a multi-cellular pixel-resolution ST count matrix
  (e.g. Spatial Transcriptomics, 10X Visium, DBiT-seq pixels) and no suitable matched
  scRNA-seq reference exists for supervised deconvolution.
when_to_use_negative:
- Sub-cellular pixel-resolution ST data, where new methods specific to that resolution
  are anticipated to be needed
- Datasets where cell-type proportions are homogeneous/uniform across pixels so genes
  lack variability, or where ST profiles thin tissue structures covering only a few
  pixels
- Very sparse ST data with high stochastic drop-out rates, where the LDA model may
  struggle to identify distinct groups of co-expressed genes
tools:
- name: STdeconvolve
  role: Core reference-free LDA-based deconvolution package that infers cell-type
    proportions and transcriptional profiles
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Language in which STdeconvolve is implemented and run
- name: BiocManager
  role: Installs STdeconvolve from Bioconductor
- name: remotes
  role: Installs the latest development version of STdeconvolve from GitHub
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:8e988611146ef581d574433269636834a4291f362072d3d23cbd11bb30a52aaa
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
yes_rate: 0.8125
n_claims: 16
---

# reference-free-cell-type-proportion-inference

## Summary

Reference-free (unsupervised) deconvolution of multi-cellular pixel-resolution spatial transcriptomics data using latent Dirichlet allocation (STdeconvolve), which recovers per-pixel cell-type proportions (theta) and cell-type transcriptional profiles (beta) without a single-cell reference.

## When to use

Use when the input is a multi-cellular pixel-resolution ST count matrix (e.g. Spatial Transcriptomics, 10X Visium, DBiT-seq pixels) and no suitable matched scRNA-seq reference exists for supervised deconvolution.

## When NOT to use

- Sub-cellular pixel-resolution ST data, where new methods specific to that resolution are anticipated to be needed
- Datasets where cell-type proportions are homogeneous/uniform across pixels so genes lack variability, or where ST profiles thin tissue structures covering only a few pixels
- Very sparse ST data with high stochastic drop-out rates, where the LDA model may struggle to identify distinct groups of co-expressed genes

## Inputs

- Pixel-by-gene count matrix from multi-cellular pixel-resolution ST data (e.g. 100 µm2 pixels; the mOB built-in dataset provides pos, counts, annot)
- Pixel spatial position coordinates (pos) for visualization
- Optional: matched MERFISH/single-cell reference and ground-truth proportions (simulated data) for benchmarking

## Outputs

- Per-pixel cell-type proportion matrix (theta / deconProp)
- Per-cell-type transcriptional profile matrix (beta / deconGexp)
- Accuracy metrics such as proportion RMSE and profile/proportion correlations (e.g. written to deconvolution_accuracy.csv or rmse_comparison_table.csv)
- Visualization of deconvolved cell-type proportions across tissue positions

## How to apply

Fit STdeconvolve's latent Dirichlet allocation model to a pixel-by-gene count matrix: first clean pixels with low library size (e.g. cleanCounts with min.lib.size = 100), then feature-select significantly overdispersed genes with restrictCorpus (e.g. removeAbove=1.0, removeBelow=0.05) since LDA relies on co-expressed gene groups varying across pixels. Choose the number of cell types K a priori guided by data-driven metrics (e.g. fitLDA over Ks = seq(2, 9, by = 1) and optimalModel with opt = 'min'); in the paper's simulated 100 µm2 mouse MPOA pixels aggregated from MERFISH (n=3072 pixels), K=9 ground-truth cell types were recovered. Extract deconvolved proportions (theta) and transcriptional profiles (beta) with getBetaTheta (perc.filt = 0.05, betaScale = 1000) and visualize with vizAllTopics. Assess accuracy by matching inferred to known cell types and computing RMSE of proportions and profile/proportion correlations; STdeconvolve's RMSE was comparable to supervised SPOTlight, RCTD, and spatialDWLS when an ideal matched MERFISH reference was available.

## Related tools

- **STdeconvolve** (Core reference-free LDA-based deconvolution package that infers cell-type proportions and transcriptional profiles) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Language in which STdeconvolve is implemented and run)
- **BiocManager** (Installs STdeconvolve from Bioconductor)
- **remotes** (Installs the latest development version of STdeconvolve from GitHub)

## Examples

```
ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1)); optLDA <- optimalModel(models = ldas, opt = "min"); results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)
```

## Evaluation signals

- Output theta rows sum to per-pixel cell-type proportions over the expected number of pixels (e.g. all 3072 simulated 100 µm2 MPOA pixels)
- Accuracy table contains a proportion RMSE value plus profile and proportion correlation values; correlations should be high while RMSE is low
- STdeconvolve's RMSE should be comparable to supervised SPOTlight, RCTD, and spatialDWLS when the ideal matched MERFISH reference is used (Fig. 2e, f comparison)
- Data-driven diagnostics from fitLDA/optimalModel support the chosen number of topics K (K=9 in the simulated MPOA case)
- Inferred beta profiles can be matched to known/ground-truth cell-type transcriptional profiles with strong correlation

## Limitations

- LDA performance depends on dataset size with respect to the number of pixels and the number of genes, and assumptions may not be satisfied
- If genes do not exhibit variability across pixels due to homogeneous or uniform cell-type representation, STdeconvolve may fail to deconvolve transcriptionally distinct cell types
- Sparse data with high stochastic drop-out rates can hinder LDA; thin structures covering only a few pixels present challenges; sub-cellular resolution data is not well suited
- The number of cells per pixel depends on cell size, which varies by organism, tissue, and disease state

## Evidence

- [intro] LDA application rationale: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [intro] Feature selection rationale: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
- [readme] Install via remotes: "require(remotes)
remotes::install_github('JEFworks-Lab/STdeconvolve')"
- [readme] Concrete pipeline parameters: "counts <- cleanCounts(cd, min.lib.size = 100)
corpus <- restrictCorus"
- [readme] Model fitting and extraction: "ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1))
optLDA <- optimalModel(models = ldas, opt = "min")"
- [discussion] Failure mode caveat: "if the gene expression in the ST data is too sparse with high rates of stochastic drop-outs40, then the LDA model may struggle to identify distinct groups of co-expressed genes"
