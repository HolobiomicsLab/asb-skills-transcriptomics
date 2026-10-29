---
name: lda-topic-model-fitting
description: Use when a pixel-by-gene spatial transcriptomics matrix (e.g. Spatial
  Transcriptomics, 10X Visium, DBiT-seq) must be deconvolved into cell-type proportions
  without a single-cell reference, after overdispersed-gene feature selection.
when_to_use_negative:
- Sub-cellular pixel-resolution ST data, where methods specifically suited to that
  resolution are anticipated to be needed
- Datasets where cell-type proportions are homogeneous/uniform across pixels so genes
  lack variability, or ST of tissue slivers/thin structures covering only a few pixels
- Very sparse ST data with high stochastic drop-out rates, where the LDA model may
  struggle to identify distinct groups of co-expressed genes
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3523
tools:
- name: STdeconvolve
  role: R package providing fitLDA (topicmodels backend), optimalModel, and getBetaTheta
    for LDA-based reference-free deconvolution
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Execution environment; STdeconvolve is an open-source R package installable
    via remotes or BiocManager
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: remotes
  role: Installs STdeconvolve from GitHub (remotes::install_github('JEFworks-Lab/STdeconvolve'))
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Alternative installer for STdeconvolve through Bioconductor (requires R >=
    4.2 for release 3.15)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:4192a3cf3a5d21594b068c65eacc3955fd6a6337314cc16a77588b586a5e6adc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# lda-topic-model-fitting

## Summary

Fit latent Dirichlet allocation (LDA) topic models over a feature-selected pixel-by-gene count corpus to deconvolve multi-cellular spatial transcriptomics pixels into K cell types, yielding beta transcriptional profiles and theta pixel proportions, as implemented in STdeconvolve.

## When to use

Use when a pixel-by-gene spatial transcriptomics matrix (e.g. Spatial Transcriptomics, 10X Visium, DBiT-seq) must be deconvolved into cell-type proportions without a single-cell reference, after overdispersed-gene feature selection.

## When NOT to use

- Sub-cellular pixel-resolution ST data, where methods specifically suited to that resolution are anticipated to be needed
- Datasets where cell-type proportions are homogeneous/uniform across pixels so genes lack variability, or ST of tissue slivers/thin structures covering only a few pixels
- Very sparse ST data with high stochastic drop-out rates, where the LDA model may struggle to identify distinct groups of co-expressed genes

## Inputs

- Pixel-by-gene count matrix from multi-cellular pixel-resolution spatial transcriptomics (e.g. built-in mOB dataset in STdeconvolve)
- Feature-selected gene corpus (top ~1000 significantly overdispersed genes, genes expressed in 5-95% of pixels)
- Grid of candidate K values (number of cell types), e.g. K = 2 to 9

## Outputs

- beta: K x N' transcriptional-profile matrix of K cell types over N' selected genes
- theta: D pixels x K cell-type-proportion matrix
- Perplexity values per candidate K for model selection

## How to apply

After feature selection (restrict to genes expressed in 5-95% of pixels and retain top ~1000 significantly overdispersed genes via restrictCorpus), fit LDA models across a grid of candidate K values with fitLDA (Ks = seq(2, 9, by = 1) in the mOB example), recording perplexity for each K. Choose K using STdeconvolve's data-driven metrics (optimalModel with opt = 'min' picks minimal perplexity), rejecting models where a rare, low-proportion cell-type topic appears. Fit the final LDA at the selected K and extract results with getBetaTheta (e.g. perc.filt = 0.05, betaScale = 1000), yielding beta, the K x N' transcriptional-profile matrix over selected genes, and theta, the D pixels x K cell-type-proportion matrix. Verify dimensions of beta (K x N') and theta (D x K) before saving (e.g. beta_matrix.csv, theta_matrix.csv) and visualizing with vizAllTopics.

## Related tools

- **STdeconvolve** (R package providing fitLDA (topicmodels backend), optimalModel, and getBetaTheta for LDA-based reference-free deconvolution) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Execution environment; STdeconvolve is an open-source R package installable via remotes or BiocManager) — https://github.com/JEFworks-Lab/STdeconvolve
- **remotes** (Installs STdeconvolve from GitHub (remotes::install_github('JEFworks-Lab/STdeconvolve'))) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Alternative installer for STdeconvolve through Bioconductor (requires R >= 4.2 for release 3.15))

## Examples

```
library(STdeconvolve); ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1)); optLDA <- optimalModel(models = ldas, opt = "min"); results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)
```

## Evaluation signals

- beta matrix has dimensions K x N' (cell types x selected genes) and theta has dimensions D x K (pixels x cell types)
- Perplexity trend across the K grid shows a minimum at the chosen K (optimalModel with opt = 'min')
- No rare, low-proportion cell-type topic appears at the chosen K
- Theta rows are interpretable as cell-type proportions per pixel (scaled per betaScale = 1000) and can be visualized spatially with vizAllTopics
- Resulting matrices save cleanly as beta_matrix.csv and theta_matrix.csv with consistent K

## Limitations

- K must be set a priori for LDA; STdeconvolve provides data-driven metrics to guide estimation of an appropriate K, but the choice remains model-dependent
- LDA performance depends on dataset size in terms of number of pixels and number of genes
- Sparse ST data with high stochastic drop-out rates or homogeneous cell-type proportions across pixels can cause the LDA model to fail to identify distinct cell types

## Evidence

- [intro] STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [intro] STdeconvolve provides data-driven metrics to guide estimation of K: "STdeconvolve provides several data-driven metrics to guide the estimation of an appropriate K"
- [readme] README fitLDA example on mOB with Ks seq(2,9): "ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1))"
- [readme] README getBetaTheta extraction of theta and beta: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000); deconProp <- results$theta; deconGexp <- results$beta"
- [discussion] Discussion: LDA performance depends on dataset size: "the performance of LDA in accurately deconvolving cell types depends on the size of the dataset with respect to the number of pixels and the number of genes"
