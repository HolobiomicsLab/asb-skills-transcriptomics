---
name: reference-free-cell-type-deconvolution
description: Use when analyzing multi-cellular pixel-resolution ST data (e.g., Spatial
  Transcriptomics, 10X Visium, DBiT-seq) and no appropriate scRNA-seq reference exists,
  or when the goal is to recover cell-type-specific gene-expression changes from mixed
  pixel measurements.
when_to_use_negative:
- Input is sub-cellular pixel-resolution ST data; the authors state STdeconvolve may
  not be best suited for that resolution and new methods will be needed
- Genes lack variability across pixels due to homogeneous/uniform cell-type proportions
  across pixels, or the data are too sparse with high stochastic drop-out rates
- Tissue covers only a few pixels (e.g., tissue slivers or thin structures), which
  presents challenges for deconvolution
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STdeconvolve
  role: Core reference-free LDA-based deconvolution package; performs overdispersed-gene
    feature selection, LDA fitting, and extraction of cell-type proportions (theta)
    and transcriptional profiles (beta)
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Runtime environment; STdeconvolve is an open-source R package (R >=4.2 required
    via Bioconductor release 3.15)
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Used to install STdeconvolve from Bioconductor
- name: remotes
  role: Used to install STdeconvolve from GitHub via remotes::install_github('JEFworks-Lab/STdeconvolve')
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:5955500f7ef11ec9e00d5e35477d14e0bafe4f8115d1a95436628ac82ccf5e55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# reference-free-cell-type-deconvolution

## Summary

STdeconvolve applies latent Dirichlet allocation (LDA) to multi-cellular pixel-resolution spatial transcriptomics (ST) data to recover cell-type transcriptional profiles (beta) and their pixel-level proportions (theta) without any single-cell RNA-seq reference. It is an unsupervised alternative to reference-based methods (SPOTlight, RCTD, spatialDWLS) when no suitable scRNA-seq reference exists.

## When to use

Use when analyzing multi-cellular pixel-resolution ST data (e.g., Spatial Transcriptomics, 10X Visium, DBiT-seq) and no appropriate scRNA-seq reference exists, or when the goal is to recover cell-type-specific gene-expression changes from mixed pixel measurements.

## When NOT to use

- Input is sub-cellular pixel-resolution ST data; the authors state STdeconvolve may not be best suited for that resolution and new methods will be needed
- Genes lack variability across pixels due to homogeneous/uniform cell-type proportions across pixels, or the data are too sparse with high stochastic drop-out rates
- Tissue covers only a few pixels (e.g., tissue slivers or thin structures), which presents challenges for deconvolution

## Inputs

- Pixel-resolution spatial transcriptomics counts matrix (pixels x genes), e.g., Spatial Transcriptomics, 10X Visium, or DBiT-seq data
- Pixel spatial coordinates (e.g., mOB$pos)
- Simulated ST data built from scRNA-seq pseudo-pixel mixtures (e.g., aged vs young mouse mammary gland single cells)

## Outputs

- theta matrix: deconvolved cell-type proportions per pixel
- beta matrix: deconvolved cell-type transcriptional profiles
- LDA model fit metrics for choosing the number of cell types K
- Optional downstream tables such as gene-wise log2 fold-change tables comparing deconvolved profiles across conditions (e.g., aged_vs_young_macrophage_log2fc.csv)

## How to apply

Install STdeconvolve from Bioconductor (requires R >=4.2 for release 3.15) or GitHub via remotes. Preprocess counts with cleanCounts (e.g., min.lib.size = 100 to remove pixels with too few genes), then feature-select overdispersed genes with restrictCorpus (e.g., removeAbove=1.0, removeBelow=0.05). Fit LDA models across a range of K values with fitLDA (e.g., Ks = seq(2, 9, by = 1)), select the optimal model using data-driven metrics via optimalModel (e.g., opt = 'min'), and extract theta (cell-type proportions per pixel) and beta (cell-type transcriptional profiles) with getBetaTheta (e.g., perc.filt = 0.05, betaScale = 1000). For perturbation analyses (e.g., aged vs young simulated mammary ST data), run the model per condition with K fixed (e.g., K=2), match deconvolved profiles to the cell type of interest (e.g., macrophages), and compute gene-wise log2 fold-changes to validate against known directional changes (e.g., Cd274/Clec4d up, Coro1a down).

## Related tools

- **STdeconvolve** (Core reference-free LDA-based deconvolution package; performs overdispersed-gene feature selection, LDA fitting, and extraction of cell-type proportions (theta) and transcriptional profiles (beta)) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Runtime environment; STdeconvolve is an open-source R package (R >=4.2 required via Bioconductor release 3.15)) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Used to install STdeconvolve from Bioconductor)
- **remotes** (Used to install STdeconvolve from GitHub via remotes::install_github('JEFworks-Lab/STdeconvolve'))

## Examples

```
library(STdeconvolve); counts <- cleanCounts(cd, min.lib.size=100); corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow=0.05); ldas <- fitLDA(t(as.matrix(corpus)), Ks=seq(2,9,by=1)); results <- getBetaTheta(optimalModel(ldas, opt='min'), perc.filt=0.05, betaScale=1000)
```

## Evaluation signals

- Validation of perturbation recovery: log2 fold-change table of aged vs young deconvolved macrophage profiles must show positive values for Cd274 and Clec4d and negative for Coro1a, matching Fig. 3c directionality
- Number of latent topics K is justified by STdeconvolve's data-driven metrics (e.g., minimal perplexity/Gini choice via optimalModel)
- Beta matrices are cell-type transcriptional profiles and theta matrices are per-pixel proportions; theta rows should sum to cell-type proportions per pixel
- Recovered cell-type spatial organization and profiles should match known tissue annotations (e.g., visualization of deconProp over pixel positions with known layers)
- Comparability check: STdeconvolve performance should be comparable to reference-based methods (SPOTlight, RCTD, spatialDWLS) when a suitable scRNA-seq reference exists

## Limitations

- LDA performance depends on dataset size with respect to number of pixels and number of genes, which may limit deconvolution when assumptions are not satisfied
- May fail to deconvolve transcriptionally distinct cell types when cell-type proportions are homogeneous across pixels so genes lack variability
- Sparse ST data with high stochastic drop-out rates may hinder LDA's ability to identify distinct groups of co-expressed genes
- Tissue slivers or thin structures covering only a few pixels present challenges for deconvolution
- Number of cells per pixel depends on cell size, which varies by organism, tissue, and disease state

## Evidence

- [intro] STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [intro] Overdispersed gene feature selection: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
- [intro] Data-driven K selection: "STdeconvolve provides several data-driven metrics to guide the estimation of an appropriate K"
- [methods] Installation via GitHub: "require(remotes)
remotes::install_github('JEFworks-Lab/STdeconvolve')"
- [readme] README example pipeline with concrete parameters: "counts <- cleanCounts(cd, min.lib.size = 100)
## feature select for genes
corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow = 0.05)"
- [readme] Extraction of theta and beta: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)
deconProp <- results$theta
deconGexp <- results$beta"
- [discussion] Homogeneous cell-type proportion failure mode: "if genes do not exhibit variability across pixels due to a homogeneous or uniform proportional representation of cell types across pixels (Supplementary Fig. S20), STdeconvolve may fail to deconvolve"
