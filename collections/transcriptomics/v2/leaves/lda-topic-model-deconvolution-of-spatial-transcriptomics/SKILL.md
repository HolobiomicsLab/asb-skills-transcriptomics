---
name: lda-topic-model-deconvolution-of-spatial-transcriptomics
description: Use when input is a pixel-by-gene count matrix from multi-cellular pixel-resolution
  ST data (e.g., 10X Visium, DBiT-seq, MERFISH-derived pixels) and cell-type proportions/profiles
  are needed but no suitable scRNA-seq reference exists.
when_to_use_negative:
- Sub-cellular pixel-resolution ST data, where new methods specifically suited to
  that resolution are anticipated to be needed
- Datasets where cell-type proportions are homogeneous/uniform across pixels so overdispersed
  genes lack variability, or tissue slivers/thin structures covering only a few pixels
- Very sparse ST data with high stochastic drop-out rates, where the LDA model may
  struggle to identify distinct groups of co-expressed genes
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: STdeconvolve
  role: 'Reference-free LDA-based deconvolution pipeline: cleanCounts, restrictCorpus,
    fitLDA, optimalModel, getBetaTheta, vizAllTopics'
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Installs STdeconvolve from Bioconductor
- name: remotes
  role: Installs the latest STdeconvolve development version from GitHub
  repo: https://github.com/JEFworks-Lab/STdeconvolve
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:f97c48232b3051b13e8d8bf37fba757ff05443d72309f349ab8348518a31c55a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# lda-topic-model-deconvolution-of-spatial-transcriptomics

## Summary

Reference-free deconvolution of multi-cellular pixel-resolution spatial transcriptomics data using latent Dirichlet allocation (LDA) via the STdeconvolve R package to recover per-pixel cell-type proportions (theta) and cell-type transcriptional profiles (beta) without a single-cell RNA-seq reference.

## When to use

Use when input is a pixel-by-gene count matrix from multi-cellular pixel-resolution ST data (e.g., 10X Visium, DBiT-seq, MERFISH-derived pixels) and cell-type proportions/profiles are needed but no suitable scRNA-seq reference exists.

## When NOT to use

- Sub-cellular pixel-resolution ST data, where new methods specifically suited to that resolution are anticipated to be needed
- Datasets where cell-type proportions are homogeneous/uniform across pixels so overdispersed genes lack variability, or tissue slivers/thin structures covering only a few pixels
- Very sparse ST data with high stochastic drop-out rates, where the LDA model may struggle to identify distinct groups of co-expressed genes

## Inputs

- pixel-by-gene count matrix from multi-cellular pixel-resolution spatial transcriptomics data (e.g., 100 um2 pixels aggregated from single-cell MERFISH MPOA counts, n=3072 pixels)
- spatial pixel position table (e.g., mOB$pos) for visualization
- optional ground-truth cell-type proportions/profiles from simulated data for validation

## Outputs

- theta: per-pixel cell-type proportion matrix (deconProp)
- beta: per-cell-type transcriptional profile matrix (deconGexp)
- LDA model selection diagnostics across candidate K values
- accuracy metrics such as proportion RMSE and profile/proportion correlations vs ground truth (e.g., deconvolution_accuracy.csv)

## How to apply

Install STdeconvolve from Bioconductor via BiocManager (R >= 4.2 for release 3.15) or the devel branch via remotes::install_github('JEFworks-Lab/STdeconvolve'). Preprocess counts with cleanCounts to remove pixels with too few genes (e.g., min.lib.size = 100), then feature-select significantly overdispersed genes with restrictCorpus (e.g., removeAbove=1.0, removeBelow=0.05) so the LDA model is applied to an informative gene set. Choose the number of cell types K using STdeconvolve's data-driven metrics (e.g., fitLDA over a K grid such as seq(2,9), then optimalModel with opt='min'), and extract beta (transcriptional profiles, betaScale=1000) and theta (per-pixel proportions) via getBetaTheta with perc.filt=0.05. If ground truth is available (e.g., simulated MERFISH-derived 100 um2 pixels), match inferred to true cell types and score performance by RMSE of proportions and correlations of profiles and proportions across all pixels.

## Related tools

- **STdeconvolve** (Reference-free LDA-based deconvolution pipeline: cleanCounts, restrictCorpus, fitLDA, optimalModel, getBetaTheta, vizAllTopics) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Installs STdeconvolve from Bioconductor)
- **remotes** (Installs the latest STdeconvolve development version from GitHub) — https://github.com/JEFworks-Lab/STdeconvolve

## Examples

```
library(STdeconvolve); counts <- cleanCounts(cd, min.lib.size=100); corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow=0.05); ldas <- fitLDA(t(as.matrix(corpus)), Ks=seq(2,9)); results <- getBetaTheta(optimalModel(ldas, opt='min'), perc.filt=0.05, betaScale=1000)
```

## Evaluation signals

- Deconvolved theta row sums per pixel equal 1 and dimensions match the filtered corpus pixels (e.g., 3072 pixels in the simulated MPOA dataset)
- When ground truth exists, proportion RMSE across pixels is low and correlations of inferred vs ground-truth transcriptional profiles and proportions are high
- Number of topics K matches a data-driven optimum from fitLDA diagnostics (e.g., K=9 for the simulated MPOA data)
- Deconvolved beta profiles correlate strongly with known cell-type marker genes after matching inferred to ground-truth cell types
- vizAllTopics plots show spatially coherent cell-type proportion distributions consistent with tissue annotations

## Limitations

- LDA performance depends on dataset size in terms of number of pixels and number of genes
- If genes lack variability across pixels due to homogeneous cell-type proportions, STdeconvolve may fail to deconvolve
- Sparse data with high stochastic drop-out rates can hinder identification of distinct co-expressed gene groups
- Not best suited for sub-cellular resolution ST data; thin structures covering only a few pixels are challenging
- Number of cells per pixel depends on cell size, which varies by organism, tissue, and disease state

## Evidence

- [intro] STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [intro] Feature selection over overdispersed genes: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels"
- [readme] Installation via remotes from GitHub: "require(remotes)
remotes::install_github('JEFworks-Lab/STdeconvolve')"
- [readme] Extract theta and beta from the optimal LDA model: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)"
- [discussion] Homogeneous proportions limit deconvolution: "if genes do not exhibit variability across pixels due to a homogeneous or uniform proportional representation of cell types across pixels"
