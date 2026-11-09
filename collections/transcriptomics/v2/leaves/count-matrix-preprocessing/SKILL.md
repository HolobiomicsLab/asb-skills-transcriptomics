---
name: count-matrix-preprocessing
description: Use when you have a raw multi-cellular pixel-resolution spatial transcriptomics
  pixel-by-gene count matrix (e.g. ST, 10X Visium, DBiT-seq) that needs cleaning and
  feature selection before LDA deconvolution.
when_to_use_negative:
- Data are sub-cellular resolution ST measurements — the paper notes STdeconvolve
  may not be best suited for sub-cellular resolution data
- The count matrix is already filtered and feature-selected, or pixels/genes are so
  few (e.g. thin tissue slivers covering only a few pixels) that filtering would remove
  most of the data
- You need reference-based deconvolution with a scRNA-seq signature (e.g. SPOTlight,
  RCTD, spatialDWLS) rather than reference-free preprocessing
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3308
tools:
- name: STdeconvolve
  role: R package providing cleanCounts and restrictCorpus functions for pixel filtering
    and overdispersed-gene feature selection prior to LDA
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Runtime environment in which the STdeconvolve preprocessing functions are
    executed
- name: BiocManager
  role: Used to install STdeconvolve from Bioconductor
- name: remotes
  role: Used to install STdeconvolve from GitHub via remotes::install_github('JEFworks-Lab/STdeconvolve')
  repo: https://github.com/JEFworks-Lab/STdeconvolve
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:771583b1d7d792fe1a8126daf8bc430504d077198c5026463fd3781f71a80b90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# count-matrix-preprocessing

## Summary

Preprocesses a pixel-by-gene count matrix for STdeconvolve LDA deconvolution by filtering low-content pixels and feature-selecting overdispersed genes, since LDA performs poorly on noisy, non-informative features.

## When to use

Use when you have a raw multi-cellular pixel-resolution spatial transcriptomics pixel-by-gene count matrix (e.g. ST, 10X Visium, DBiT-seq) that needs cleaning and feature selection before LDA deconvolution.

## When NOT to use

- Data are sub-cellular resolution ST measurements — the paper notes STdeconvolve may not be best suited for sub-cellular resolution data
- The count matrix is already filtered and feature-selected, or pixels/genes are so few (e.g. thin tissue slivers covering only a few pixels) that filtering would remove most of the data
- You need reference-based deconvolution with a scRNA-seq signature (e.g. SPOTlight, RCTD, spatialDWLS) rather than reference-free preprocessing

## Inputs

- Pixel-by-gene count matrix from multi-cellular pixel-resolution spatial transcriptomics data (e.g. mOB$counts from the STdeconvolve built-in mOB dataset)
- Pixel position/annotation metadata (e.g. mOB$pos, mOB$annot) for downstream visualization

## Outputs

- Cleaned pixel-by-gene count matrix after removal of low-content pixels
- Feature-selected gene corpus (restricted to significantly overdispersed genes expressed in 5-95% of pixels, e.g. top 1000 genes) suitable as input to fitLDA

## How to apply

Start from a pixel-by-gene count matrix (e.g. the built-in mOB dataset loaded via data(mOB), using mOB$counts) and first call cleanCounts with a minimum library size (e.g. min.lib.size = 100) to remove pixels expressing too few genes. Then apply restrictCorpus for feature selection, restricting genes to those expressed in a 5-95% range of pixels (e.g. removeAbove=1.0, removeBelow=0.05) and retaining significantly overdispersed genes — genes with higher-than-expected expression variance across ST pixels — such as the top 1000 overdispersed genes. The rationale is that LDA relies on co-expressed gene modules, so low-content pixels and non-variable genes dilute the signal; overdispersed genes are the ones informative about cell-type mixtures. The resulting corpus is transposed (genes as rows/pixels as columns via t(as.matrix(corpus))) before passing to fitLDA.

## Related tools

- **STdeconvolve** (R package providing cleanCounts and restrictCorpus functions for pixel filtering and overdispersed-gene feature selection prior to LDA) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Runtime environment in which the STdeconvolve preprocessing functions are executed)
- **BiocManager** (Used to install STdeconvolve from Bioconductor)
- **remotes** (Used to install STdeconvolve from GitHub via remotes::install_github('JEFworks-Lab/STdeconvolve')) — https://github.com/JEFworks-Lab/STdeconvolve

## Examples

```
library(STdeconvolve); data(mOB); counts <- cleanCounts(mOB$counts, min.lib.size = 100); corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow = 0.05)
```

## Evaluation signals

- cleanCounts removes pixels below the min.lib.size threshold (e.g. 100) — verify the count matrix now contains only pixels with sufficient gene content
- restrictCorpus output should contain only genes expressed in the 5-95% pixel range (removeBelow=0.05, removeAbove=1.0), with a reduced gene set (e.g. top 1000 overdispersed genes)
- The corpus must be transposed before fitLDA so that dimensions match (genes x pixels via t(as.matrix(corpus))); a shape mismatch is a failure sign
- Downstream fitLDA should run without errors on the corpus and produce perplexity values across candidate K values, confirming the preprocessing retained usable signal

## Limitations

- LDA performance depends on dataset size with respect to the number of pixels and genes; small or sparse datasets with high stochastic drop-out rates may hinder deconvolution
- If genes lack variability across pixels (homogeneous/uniform cell-type proportions), feature selection cannot rescue deconvolution — STdeconvolve may fail
- Thin tissue structures covering only a few pixels present challenges for deconvolution after filtering
- The number of cells per pixel varies by organism, tissue, and disease state, so fixed thresholds like min.lib.size may need tuning

## Evidence

- [readme] Pixel filtering threshold: "## remove pixels with too few genes
counts <- cleanCounts(cd, min.lib.size = 100)"
- [readme] Feature selection thresholds: "## feature select for genes
corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow = 0.05)"
- [intro] Overdispersed gene rationale: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels"
- [discussion] Sparsity limitation: "if the gene expression in the ST data is too sparse with high rates of stochastic drop-outs40, then the LDA model may struggle to identify distinct groups of co-expressed genes"
- [discussion] Homogeneity limitation: "if genes do not exhibit variability across pixels due to a homogeneous or uniform proportional representation of cell types across pixels (Supplementary Fig. S20), STdeconvolve may fail to deconvolve"
