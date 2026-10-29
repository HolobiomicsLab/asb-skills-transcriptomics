---
name: overdispersed-gene-feature-selection
description: Use when preparing multi-cellular pixel-resolution spatial transcriptomics
  counts (e.g., mOB, Visium, DBiT-seq, Slide-seq) before fitting LDA in STdeconvolve,
  and noisy, low-variance genes must be filtered out.
when_to_use_negative:
- The ST data is sub-cellular resolution or covers only a few pixels (e.g., tissue
  slivers), where overdispersed genes are insufficient for deconvolution
- Cell-type proportions are homogeneous/uniform across pixels so genes lack variability
  and overdispersion filtering yields no informative signal
- A pre-computed feature-selected gene set is already available and no re-filtering
  is needed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STdeconvolve
  role: Provides the overdispersed-gene feature-selection functions (e.g., cleanCounts,
    restrictCorpus) that build the corpus for LDA
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Installs the STdeconvolve package from Bioconductor
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: remotes
  role: Installs the development version of STdeconvolve from GitHub
  repo: https://github.com/JEFworks-Lab/STdeconvolve
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:43a8b286c8bb31c297a9b78f48c3569cd58f5d2f9703d09be231ec6d0da64b97
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
yes_rate: 0.6363636363636364
n_claims: 11
---

# overdispersed-gene-feature-selection

## Summary

STdeconvolve's unsupervised feature-selection step that retains significantly overdispersed genes — those with higher-than-expected expression variance across ST pixels — to prepare a pixel-by-gene count matrix for latent Dirichlet allocation (LDA) deconvolution.

## When to use

Use when preparing multi-cellular pixel-resolution spatial transcriptomics counts (e.g., mOB, Visium, DBiT-seq, Slide-seq) before fitting LDA in STdeconvolve, and noisy, low-variance genes must be filtered out.

## When NOT to use

- The ST data is sub-cellular resolution or covers only a few pixels (e.g., tissue slivers), where overdispersed genes are insufficient for deconvolution
- Cell-type proportions are homogeneous/uniform across pixels so genes lack variability and overdispersion filtering yields no informative signal
- A pre-computed feature-selected gene set is already available and no re-filtering is needed

## Inputs

- Pixel-by-gene spatial transcriptomics count matrix (e.g., mOB counts from data(mOB))
- Pixel coordinate table (e.g., mOB$pos)
- Optional count-cleaning parameters (min.lib.size, e.g. 100 transcripts per pixel)

## Outputs

- Filtered corpus of significantly overdispersed genes (reduced pixel-by-gene matrix, e.g., top 1000 genes)
- Corpus ready for STdeconvolve fitLDA input (genes x pixels)

## How to apply

Start from a cleaned pixel-by-gene count matrix (e.g., after cleanCounts with min.lib.size = 100 to drop pixels with too few genes). Apply STdeconvolve's feature selection, such as restrictCorus/restrictCorpus-style filtering, restricting genes to those expressed in 5–95% of pixels and retaining the top ~1000 significantly overdispersed genes (README example: removeAbove=1.0, removeBelow=0.05). The rationale is that LDA needs genes with variable expression across pixels to identify groups of co-expressed genes as cell-type topics; genes with higher-than-expected variance carry the cell-type signal. The output corpus is then transposed and passed to fitLDA across a grid of candidate K values. Validate the corpus by confirming it contains only overdispersed genes and remains large enough to support LDA at the chosen K.

## Related tools

- **STdeconvolve** (Provides the overdispersed-gene feature-selection functions (e.g., cleanCounts, restrictCorpus) that build the corpus for LDA) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Installs the STdeconvolve package from Bioconductor) — https://github.com/JEFworks-Lab/STdeconvolve
- **remotes** (Installs the development version of STdeconvolve from GitHub) — https://github.com/JEFworks-Lab/STdeconvolve

## Examples

```
library(STdeconvolve); counts <- cleanCounts(cd, min.lib.size = 100); corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow = 0.05)
```

## Evaluation signals

- Corpus dimensions: retained gene set is restricted to significantly overdispersed genes (e.g., expressed in 5–95% of pixels, top ~1000 genes per the methods)
- Downstream LDA converges and perplexity metrics can be computed across the K grid on the filtered corpus
- Final beta (K x N' genes) and theta (D pixels x K) matrices have the expected dimensions matching the filtered corpus gene count N'
- Deconvolved topics recover expected marker genes (e.g., Myh6 atrium, Myh7 ventricle in DBiT-seq E11), indicating informative gene selection
- No pixels with too few genes remain (cleanCounts min.lib.size = 100 removed them before feature selection)

## Limitations

- If gene expression is too sparse with high stochastic drop-out rates, the LDA model may struggle to identify distinct groups of co-expressed genes even after feature selection
- If genes do not exhibit variability across pixels due to homogeneous cell-type proportions, STdeconvolve may fail to deconvolve regardless of filtering
- LDA performance depends on dataset size with respect to number of pixels and number of genes retained

## Evidence

- [intro] STdeconvolve feature selects significantly overdispersed genes across ST pixels to improve the application of LDA to ST data.: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
- [readme] README preprocessing example: "counts <- cleanCounts(cd, min.lib.size = 100)
## feature select for genes
corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow = 0.05)"
- [methods] Methods workflow: restrict to 5-95% pixels, top 1000 overdispersed genes, then LDA: [evidence span withheld — anchor status: not_found]
- [discussion] Failure mode for homogeneous data: "if genes do not exhibit variability across pixels due to a homogeneous or uniform proportional representation of cell types across pixels"
