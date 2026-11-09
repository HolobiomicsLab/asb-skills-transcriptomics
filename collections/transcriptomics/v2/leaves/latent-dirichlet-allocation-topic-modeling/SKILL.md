---
name: latent-dirichlet-allocation-topic-modeling
description: Use when ST pixels are multi-cellular (e.g. Visium, DBiT-seq, Slide-seq)
  and no suitable scRNA-seq reference exists, or cell-type proportions and marker-enriched
  transcriptional profiles must be inferred in an unsupervised manner.
when_to_use_negative:
- ST data at sub-cellular pixel resolution, where methods specifically suited for
  that resolution will be needed
- Datasets where cell-type proportions are homogeneous/uniform across pixels so overdispersed
  genes are absent, or tissue slivers/thin structures covering only a few pixels
- Very sparse data with high stochastic drop-out rates, where the LDA model may struggle
  to identify distinct groups of co-expressed genes
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: STdeconvolve
  role: 'Reference-free LDA-based deconvolution: overdispersed gene feature selection,
    LDA fitting across K, extraction of theta and beta, and visualization of deconvolved
    proportions'
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Installs STdeconvolve from Bioconductor
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: remotes
  role: Installs the development version of STdeconvolve from GitHub
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Runtime environment for the STdeconvolve package (R >= 4.2 via Bioconductor
    release 3.15)
  repo: https://github.com/JEFworks-Lab/STdeconvolve
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:5d3a997587e3819b8d42d8c0d6e32843a8930b598fc1ad2361422fef324ffe07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# latent-dirichlet-allocation-topic-modeling

## Summary

Reference-free deconvolution of multi-cellular pixel-resolution spatial transcriptomics data by fitting latent Dirichlet allocation (LDA) to recover per-cell-type transcriptional profiles (beta) and per-pixel cell-type proportions (theta) without a single-cell RNA-seq reference.

## When to use

Use when ST pixels are multi-cellular (e.g. Visium, DBiT-seq, Slide-seq) and no suitable scRNA-seq reference exists, or cell-type proportions and marker-enriched transcriptional profiles must be inferred in an unsupervised manner.

## When NOT to use

- ST data at sub-cellular pixel resolution, where methods specifically suited for that resolution will be needed
- Datasets where cell-type proportions are homogeneous/uniform across pixels so overdispersed genes are absent, or tissue slivers/thin structures covering only a few pixels
- Very sparse data with high stochastic drop-out rates, where the LDA model may struggle to identify distinct groups of co-expressed genes

## Inputs

- Pixel-by-gene raw count matrix from multi-cellular pixel-resolution ST data (e.g. 10X Visium, DBiT-seq, Slide-seq)
- Pixel spatial coordinates for visualization (vizAllTopics)
- Pre-specified or candidate number of cell types K (e.g. K = 13, 14, 20)

## Outputs

- Per-pixel cell-type proportion matrix (theta)
- Per-topic (cell-type) transcriptional profile matrix (beta) whose top genes support marker-based annotation
- Diagnostic LDA fit metrics over the K grid to guide choice of optimal K

## How to apply

Install STdeconvolve from Bioconductor via BiocManager (R >= 4.2) or the GitHub devel branch via remotes::install_github('JEFworks-Lab/STdeconvolve'). Preprocess counts with cleanCounts (e.g. min.lib.size = 100) to drop low-content pixels, then feature select significantly overdispersed genes with restrictCorpus (e.g. removeAbove=1.0, removeBelow=0.05) so LDA models genes with higher-than-expected variance across pixels. Fit LDA over a range of K with fitLDA and select the appropriate number of cell types using STdeconvolve's data-driven metrics via optimalModel (K must be set a priori). Extract theta (per-pixel cell-type proportions) and beta (per-topic transcriptional profiles) with getBetaTheta (e.g. perc.filt = 0.05, betaScale = 1000), then annotate topics by matching recovered top genes to known markers (e.g. Myh6 atrium, Myh7 ventricle, Apoa2 liver, Purkinje/Bergmann glia signatures). The method has been validated at K=20 (Visium), K=13 (DBiT-seq E11 embryo), and K=14 (Slide-seq cerebellum).

## Related tools

- **STdeconvolve** (Reference-free LDA-based deconvolution: overdispersed gene feature selection, LDA fitting across K, extraction of theta and beta, and visualization of deconvolved proportions) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Installs STdeconvolve from Bioconductor) — https://github.com/JEFworks-Lab/STdeconvolve
- **remotes** (Installs the development version of STdeconvolve from GitHub) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Runtime environment for the STdeconvolve package (R >= 4.2 via Bioconductor release 3.15)) — https://github.com/JEFworks-Lab/STdeconvolve

## Examples

```
library(STdeconvolve); corpus <- restrictCorpus(cleanCounts(cd, min.lib.size = 100), removeAbove=1.0, removeBelow = 0.05); ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1)); results <- getBetaTheta(optimalModel(models = ldas, opt = "min"), perc.filt = 0.05, betaScale = 1000)
```

## Evaluation signals

- Each dataset's deconvolution yields exactly the pre-specified number of topics (e.g. K = 20, 13, 14) and optimalModel selects that K from the fitLDA diagnostics
- Expected marker genes appear as top genes of the corresponding annotated topics (e.g. Myh6 for atrium, Myh7 for ventricle, Apoa2 for liver, Hba.a2 for blood vessels)
- theta rows are valid per-pixel proportions over K cell types and beta is a per-topic gene profile after getBetaTheta filtering (perc.filt = 0.05)
- Overdispersed-gene corpus is non-empty after restrictCorpus filtering (removeAbove=1.0, removeBelow=0.05) and cleanCounts (min.lib.size = 100)
- Spatially coherent deconvolved proportions visualizable with vizAllTopics that align with known anatomical annotations

## Limitations

- LDA performance depends on dataset size in terms of number of pixels and number of genes, which may limit deconvolution when assumptions are not satisfied
- ST profiling of tissue slivers or thin structures covering only a few pixels presents challenges for deconvolution by STdeconvolve
- If genes do not exhibit variability across pixels due to homogeneous or uniform proportional representation of cell types, STdeconvolve may fail to deconvolve
- If gene expression is too sparse with high rates of stochastic drop-outs, the LDA model may struggle to identify distinct groups of co-expressed genes
- STdeconvolve may not be best suited for sub-cellular resolution ST data; the number of cells per pixel depends on cell size, which varies by organism, tissue, and disease state

## Evidence

- [intro] STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [intro] Overdispersed gene feature selection: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
- [intro] Data-driven K selection: "STdeconvolve provides several data-driven metrics to guide the estimation of an appropriate K"
- [readme] README example workflow with cleanCounts, restrictCorpus, fitLDA, getBetaTheta: [evidence span withheld — anchor status: not_found]
- [discussion] Sparse data failure mode: "if the gene expression in the ST data is too sparse with high rates of stochastic drop-outs40, then the LDA model may struggle to identify distinct groups of co-expressed genes"
