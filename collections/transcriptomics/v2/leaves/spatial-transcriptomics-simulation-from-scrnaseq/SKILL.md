---
name: spatial-transcriptomics-simulation-from-scrnaseq
description: Use when you need ground-truth-controlled ST data to test whether a deconvolution
  method recovers known cell-type proportions or condition-specific gene-expression
  changes (e.g., age-associated macrophage Cd274/Clec4d up, Coro1a down) from pixel
  mixtures.
when_to_use_negative:
- The ST data are sub-cellular resolution or tissue slivers covering only a few pixels,
  where LDA-based deconvolution assumptions fail
- Cell-type proportions are homogeneous/uniform across pixels so overdispersed-gene
  feature selection cannot distinguish cell types
- A suitable single-cell reference already exists and a reference-based method (RCTD,
  SPOTlight, spatialDWLS) is preferred for the actual analysis rather than benchmarking
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_2269
tools:
- name: STdeconvolve
  role: Reference-free LDA-based deconvolution applied to the simulated ST pixels
    to recover cell-type proportions and transcriptional profiles
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Language in which the simulation, aggregation into pseudo-pixels, and fold-change
    computation are implemented
- name: remotes
  role: Used to install STdeconvolve from GitHub for the simulation-benchmark workflow
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Alternative installer for STdeconvolve through Bioconductor
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:a54a7c3c12187c93b3dd7d604f82111d28079a584ed86a444f6c6de5eb114243
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# spatial-transcriptomics-simulation-from-scrnaseq

## Summary

Simulate multi-cellular pixel-resolution spatial transcriptomics (ST) data by aggregating single cells from scRNA-seq conditions (e.g., aged vs young mouse mammary gland) into pseudo-pixel mixtures, so that reference-free deconvolution methods like STdeconvolve can be benchmarked on recovering known perturbation-driven, cell-type-specific expression changes.

## When to use

Use when you need ground-truth-controlled ST data to test whether a deconvolution method recovers known cell-type proportions or condition-specific gene-expression changes (e.g., age-associated macrophage Cd274/Clec4d up, Coro1a down) from pixel mixtures.

## When NOT to use

- The ST data are sub-cellular resolution or tissue slivers covering only a few pixels, where LDA-based deconvolution assumptions fail
- Cell-type proportions are homogeneous/uniform across pixels so overdispersed-gene feature selection cannot distinguish cell types
- A suitable single-cell reference already exists and a reference-based method (RCTD, SPOTlight, spatialDWLS) is preferred for the actual analysis rather than benchmarking

## Inputs

- scRNA-seq count matrices from two biological conditions (e.g., aged and young mouse mammary gland single cells) with cell-type annotations
- STdeconvolve R package (installed via remotes::install_github('JEFworks-Lab/STdeconvolve') or BiocManager)

## Outputs

- Simulated aged and young pseudo-pixel ST count matrices
- Deconvolved cell-type proportions (theta) and transcriptional profiles (beta) per condition
- Gene-by-gene log2 fold-change table of aged vs young deconvolved macrophage profiles (aged_vs_young_macrophage_log2fc.csv)

## How to apply

Aggregate single cells from the two scRNA-seq conditions (aged and young mouse mammary gland) into pseudo-pixel mixtures representing aged and young tissue ST count matrices. Run STdeconvolve's pipeline on the simulated pixels: cleanCounts to remove low-content pixels (e.g., min.lib.size = 100), restrictCorpus to feature-select significantly overdispersed genes, fitLDA with the number of cell types fixed at K=2, then optimalModel and getBetaTheta (perc.filt = 0.05, betaScale = 1000) to extract cell-type proportions (theta) and transcriptional profiles (beta). Match the deconvolved cell-type profiles to the cell population of interest (macrophages) in each condition, compute gene-by-gene log2 fold-change of aged vs young macrophage profiles, and write the result (e.g., aged_vs_young_macrophage_log2fc.csv). The rationale is that simulation supplies known ground truth while retaining the multi-cellular pixel-mixture structure that LDA-based reference-free deconvolution assumes.

## Related tools

- **STdeconvolve** (Reference-free LDA-based deconvolution applied to the simulated ST pixels to recover cell-type proportions and transcriptional profiles) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Language in which the simulation, aggregation into pseudo-pixels, and fold-change computation are implemented)
- **remotes** (Used to install STdeconvolve from GitHub for the simulation-benchmark workflow) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Alternative installer for STdeconvolve through Bioconductor)

## Examples

```
library(STdeconvolve); corpus <- restrictCorpus(cleanCounts(cd, min.lib.size = 100), removeAbove=1.0, removeBelow = 0.05); ldas <- fitLDA(t(as.matrix(corpus)), Ks = c(2)); results <- getBetaTheta(optimalModel(models = ldas, opt = "min"), perc.filt = 0.05, betaScale = 1000)
```

## Evaluation signals

- aged_vs_young_macrophage_log2fc.csv shows positive log2 fold-change for Cd274 and Clec4d and negative log2 fold-change for Coro1a, matching Fig. 3c directionality
- Deconvolved K=2 LDA model recovers two coherent cell-type transcriptional profiles (beta) whose proportions (theta) vary across simulated pixels
- Overdispersed-gene feature selection via restrictCorpus yields a restricted corpus before fitting LDA
- Recovered aged-vs-young macrophage expression changes are consistent with the original scRNA-seq study's reported directionality

## Limitations

- LDA performance depends on dataset size in terms of number of pixels and number of genes
- If gene expression is too sparse with high stochastic drop-out rates, the LDA model may struggle to identify distinct groups of co-expressed genes
- STdeconvolve may fail to deconvolve transcriptionally distinct cell types when cell-type proportions are homogeneous or uniform across pixels, so genes lack variability
- STdeconvolve may not be suited for sub-cellular pixel-resolution ST data

## Evidence

- [intro] STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [intro] Overdispersed gene feature selection: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
- [readme] Example STdeconvolve invocation from README: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)"
- [readme] README preprocessing example with cleanCounts and restrictCorpus: "counts <- cleanCounts(cd, min.lib.size = 100)
## feature select for genes
corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow = 0.05)"
- [discussion] Sparse-data failure mode from discussion: "if the gene expression in the ST data is too sparse with high rates of stochastic drop-outs40, then the LDA model may struggle to identify distinct groups of co-expressed genes"
