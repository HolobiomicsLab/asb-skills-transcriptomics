---
name: lda-topic-modeling-of-st-pixels
description: Use when the input is a pixel-by-gene count matrix from multi-cellular
  pixel-resolution ST (e.g., ST, 10X Visium, DBiT-seq, MERFISH) and cell-type proportions/profiles
  are needed without a matched scRNA-seq reference.
when_to_use_negative:
- ST data at sub-cellular pixel resolution — the authors note methods specifically
  suited for sub-cellular resolution ST data will be needed
- Data where gene expression is too sparse with high stochastic drop-out rates, or
  where cell-type proportions are homogeneous across pixels so overdispersed genes
  lack variability
- Tissue slivers or thin structures covering only a few pixels, or datasets too small
  in pixels/genes for LDA assumptions to hold
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: STdeconvolve
  role: 'R package implementing the LDA-based reference-free deconvolution: cleanCounts,
    restrictCorpus, fitLDA, optimalModel, getBetaTheta, vizAllTopics'
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Installs STdeconvolve from Bioconductor (release 3.15 requires R >=4.2)
- name: remotes
  role: Installs the development version of STdeconvolve from GitHub
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Runtime environment for the STdeconvolve pipeline and downstream log2 fold-change/visual
    analyses
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:a96be36bcc4324e63a79e635849ba071531fa8bc99d01218293f805b1a7a5a33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# lda-topic-modeling-of-st-pixels

## Summary

Reference-free deconvolution of multi-cellular pixel-resolution spatial transcriptomics data by fitting latent Dirichlet allocation (LDA) to infer cell-type transcriptional profiles (beta) and per-pixel cell-type proportions (theta), as implemented in the STdeconvolve R package.

## When to use

Use when the input is a pixel-by-gene count matrix from multi-cellular pixel-resolution ST (e.g., ST, 10X Visium, DBiT-seq, MERFISH) and cell-type proportions/profiles are needed without a matched scRNA-seq reference.

## When NOT to use

- ST data at sub-cellular pixel resolution — the authors note methods specifically suited for sub-cellular resolution ST data will be needed
- Data where gene expression is too sparse with high stochastic drop-out rates, or where cell-type proportions are homogeneous across pixels so overdispersed genes lack variability
- Tissue slivers or thin structures covering only a few pixels, or datasets too small in pixels/genes for LDA assumptions to hold

## Inputs

- pixel-by-gene count matrix from multi-cellular pixel-resolution ST data (e.g., the built-in mOB counts matrix, 10X Visium, DBiT-seq, MERFISH)
- pixel spatial coordinates (e.g., mOB$pos) for visualization
- optional tissue layer annotations (e.g., mOB$annot) for overlaying on proportion plots
- optional single-cell mixtures/pseudo-pixels simulated from scRNA-seq (e.g., aged vs young mouse mammary gland) for validation

## Outputs

- theta matrix: per-pixel cell-type proportions (deconProp)
- beta matrix: per-cell-type transcriptional profiles (deconGexp)
- fitted LDA model objects over the Ks grid and the selected optimal model
- spatial visualization of per-pixel cell-type proportions (vizAllTopics plot)
- derived tables such as aged_vs_young_macrophage_log2fc.csv or mob_k12_deconvolution.rds

## How to apply

First preprocess the pixel-by-gene count matrix: remove pixels with too few genes using cleanCounts (e.g., min.lib.size = 100), then feature-select significantly overdispersed genes (higher-than-expected expression variance across pixels) with restrictCorpus (e.g., removeAbove = 1.0, removeBelow = 0.05). Fit LDA over a range of topics, e.g., fitLDA with Ks = seq(2, 9), and choose the optimal K using STdeconvolve's data-driven metrics (optimalModel with opt = 'min'); K must be set a priori, so this search is the key decision point. Extract theta (per-pixel cell-type proportions) and beta (cell-type transcriptional profiles) with getBetaTheta (perc.filt = 0.05, betaScale = 1000), and visualize spatial proportions with vizAllTopics. Annotate recovered cell types by their top upregulated genes (e.g., X7 markers Sox11, Nrep, Dcx) and, for perturbation comparisons, match deconvolved profiles across conditions and compute gene-by-gene log2 fold-changes. Because the model is unsupervised, validate against known spatial anatomy (e.g., X7 localizing to the granule cell layer at the expected rostral migratory stream in the mOB data) or expected perturbation directionality rather than a reference-based signature.

## Related tools

- **STdeconvolve** (R package implementing the LDA-based reference-free deconvolution: cleanCounts, restrictCorpus, fitLDA, optimalModel, getBetaTheta, vizAllTopics) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Installs STdeconvolve from Bioconductor (release 3.15 requires R >=4.2))
- **remotes** (Installs the development version of STdeconvolve from GitHub) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Runtime environment for the STdeconvolve pipeline and downstream log2 fold-change/visual analyses)

## Examples

```
library(STdeconvolve); counts <- cleanCounts(mOB$counts, min.lib.size = 100); corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow=0.05); ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2,9,by=1)); results <- getBetaTheta(optimalModel(ldas, opt='min'), perc.filt=0.05, betaScale=1000)
```

## Evaluation signals

- The overdispersion-filtered corpus is non-empty and pixels with low library size (< min.lib.size = 100) have been removed before fitting
- The chosen K is supported by STdeconvolve's data-driven metrics; e.g., the mOB benchmark recovers K=12 cell types
- Recovered cell-type proportions reproduce known anatomy: e.g., cluster X7 localizes over the granule cell layer at the expected rostral migratory stream position in the mOB data
- Recovered transcriptional profiles match expected marker genes: X7 top genes Sox11, Nrep, Dcx; deconvolved macrophage profiles show positive log2FC for Cd274 and Clec4d and negative log2FC for Coro1a (aged vs young)
- Against simulated mixtures with known proportions, per-pixel RMSE of theta is lower than transcriptional clustering and BayesSpace (Diebold-Mariano p < 2.2e-16)

## Limitations

- LDA performance depends on dataset size with respect to number of pixels and number of genes; assumptions may not be satisfied for small datasets
- Sparse ST data with high stochastic drop-out rates may prevent the LDA model from identifying distinct groups of co-expressed genes
- If genes lack variability across pixels due to homogeneous or uniform cell-type proportions, STdeconvolve may fail to deconvolve transcriptionally distinct cell types
- ST profiling of tissue slivers or thin structures covering only a few pixels presents challenges for deconvolution
- STdeconvolve may not be best suited for sub-cellular pixel-resolution ST data, for which new methods are anticipated

## Evidence

- [intro] LDA applies to ST pixels: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [intro] overdispersed gene feature selection: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels"
- [intro] K guidance: "STdeconvolve provides several data-driven metrics to guide the estimation of an appropriate K"
- [readme] README example pipeline: "corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow = 0.05)"
- [readme] extract theta and beta: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)"
- [discussion] sparse-data failure mode: "if the gene expression in the ST data is too sparse with high rates of stochastic drop-outs40, then the LDA model may struggle to identify distinct groups of co-expressed genes"
