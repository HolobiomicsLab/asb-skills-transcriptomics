---
name: deconvolution-accuracy-rmse-benchmarking
description: Use when simulated or ground-truth-annotated multi-cellular pixel-resolution
  ST data (e.g. 100 um2 MPOA pixels) with known cell-type proportions is available
  and you need to quantify how well STdeconvolve's LDA-inferred proportions compare
  to supervised, scRNA-seq-reference-based deconvolution.
when_to_use_negative:
- Data is sub-cellular pixel-resolution ST; the article states STdeconvolve may not
  be best suited for sub-cellular resolution and new methods specific to that resolution
  will be needed
- No ground-truth (or reference) proportions exist to compute RMSE against, so accuracy
  cannot be quantified
- Cell-type proportions are homogeneous/uniform across pixels so genes lack variability
  — STdeconvolve may fail to deconvolve distinct cell types
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STdeconvolve
  role: Reference-free LDA-based deconvolution whose inferred per-pixel proportions
    are benchmarked by RMSE against ground truth
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: SPOTlight
  role: Supervised baseline using NMF seeded with cell-type marker genes from the
    scRNA-seq reference to estimate per-pixel proportions
- name: RCTD
  role: Supervised baseline building a probabilistic model from reference cell-type-specific
    mean expression of marker genes
- name: spatialDWLS
  role: Supervised baseline using signature-gene enrichment plus dampened weighted
    least squares to estimate per-pixel proportions
- name: remotes
  role: R package used to install STdeconvolve from GitHub
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: R package manager used to install STdeconvolve from Bioconductor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:12790d5711fd455fd6aa84fc77558a1c016ae9627a41565dce2f70df1f79c0ec
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
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# deconvolution-accuracy-rmse-benchmarking

## Summary

Benchmarks cell-type deconvolution accuracy on multi-cellular pixel-resolution spatial transcriptomics data by computing root-mean-square error (RMSE) of estimated versus ground-truth cell-type proportions, comparing reference-free STdeconvolve (LDA-based) against supervised, single-cell-reference-based methods (SPOTlight, RCTD, spatialDWLS).

## When to use

Use when simulated or ground-truth-annotated multi-cellular pixel-resolution ST data (e.g. 100 um2 MPOA pixels) with known cell-type proportions is available and you need to quantify how well STdeconvolve's LDA-inferred proportions compare to supervised, scRNA-seq-reference-based deconvolution.

## When NOT to use

- Data is sub-cellular pixel-resolution ST; the article states STdeconvolve may not be best suited for sub-cellular resolution and new methods specific to that resolution will be needed
- No ground-truth (or reference) proportions exist to compute RMSE against, so accuracy cannot be quantified
- Cell-type proportions are homogeneous/uniform across pixels so genes lack variability — STdeconvolve may fail to deconvolve distinct cell types

## Inputs

- Simulated multi-cellular pixel-resolution ST count matrix with known ground-truth cell-type proportions (e.g. 100 um2 MPOA ST data)
- Matched single-cell transcriptomics reference (e.g. MERFISH MPOA scRNA-seq with cell-type labels)
- Pixel spatial coordinates (e.g. mOB$pos-style position table)
- Grid of candidate cell-type numbers K for LDA (e.g. seq(2, 9, by = 1))

## Outputs

- Per-pixel estimated cell-type proportion matrices (theta) from STdeconvolve, SPOTlight, RCTD, and spatialDWLS
- RMSE comparison table of deconvolution accuracy vs ground truth for all methods (e.g. rmse_comparison_table.csv)
- Deconvolved cell-type transcriptional profiles (beta) and proportion visualizations (vizAllTopics plots)

## How to apply

Load the simulated pixel count matrix with known ground-truth proportions and the matched single-cell transcriptomics reference (e.g. mouse preoptic area MERFISH). Run STdeconvolve's reference-free LDA pipeline: cleanCounts to remove pixels with too few genes (min.lib.size = 100), restrictCorpus to feature-select significantly overdispersed genes (e.g. removeAbove=1.0, removeBelow=0.05), fitLDA over a K grid with data-driven metrics to pick K, then getBetaTheta to extract per-pixel proportions (theta). In parallel fit SPOTlight (NMF seeded with reference-derived cell-type marker genes), RCTD (probabilistic model from reference cell-type-specific mean expression of marker genes), and spatialDWLS (signature-gene enrichment followed by dampened weighted least squares) using the reference to estimate per-pixel proportions. Compute RMSE between each method's estimated proportions and the simulated ground-truth proportions per pixel, tabulate for all four methods (e.g. rmse_comparison_table.csv), and interpret comparability as evidence that STdeconvolve matches supervised methods when a suitable single-cell reference exists.

## Related tools

- **STdeconvolve** (Reference-free LDA-based deconvolution whose inferred per-pixel proportions are benchmarked by RMSE against ground truth) — https://github.com/JEFworks-Lab/STdeconvolve
- **SPOTlight** (Supervised baseline using NMF seeded with cell-type marker genes from the scRNA-seq reference to estimate per-pixel proportions)
- **RCTD** (Supervised baseline building a probabilistic model from reference cell-type-specific mean expression of marker genes)
- **spatialDWLS** (Supervised baseline using signature-gene enrichment plus dampened weighted least squares to estimate per-pixel proportions)
- **remotes** (R package used to install STdeconvolve from GitHub) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (R package manager used to install STdeconvolve from Bioconductor)

## Examples

```
library(STdeconvolve); counts <- cleanCounts(cd, min.lib.size = 100); corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow = 0.05); ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1)); results <- getBetaTheta(optimalModel(models = ldas, opt = "min"), perc.filt = 0.05, betaScale = 1000)
```

## Evaluation signals

- RMSE table contains rows for all four methods: SPOTlight, RCTD, spatialDWLS, and STdeconvolve
- STdeconvolve's RMSE is comparable to the reference-based methods on the simulated MPOA data, consistent with the paper's Fig. 2e, f finding
- All estimated per-pixel proportions are non-negative and row-normalized (sum to 1 per pixel)
- The number of cell types K used for LDA is justified by STdeconvolve's data-driven metrics from the fitLDA K grid
- Overdispersed gene feature selection (restrictCorpus with removeAbove/removeBelow thresholds) was applied before fitting LDA

## Limitations

- LDA performance depends on dataset size in terms of number of pixels and number of genes, which may limit deconvolution when assumptions are not satisfied
- ST profiling of tissue slivers or thin structures covering only a few pixels presents challenges for deconvolution by STdeconvolve
- If gene expression in the ST data is too sparse with high stochastic drop-out rates, the LDA model may struggle to identify distinct groups of co-expressed genes
- Reference-based baselines depend on reference quality — batch effects, inter- and intra-sample heterogeneity, and missing or inconsistent cell types can bias them; STdeconvolve may perform better when no suitable reference exists

## Evidence

- [abstract] STdeconvolve provides comparable performance to existing reference-based methods when suitable single-cell references are available: "STdeconvolve provides comparable performance to existing reference-based methods when suitable single-cell references are available, as well as potentially superior performance when suitable"
- [intro] SPOTlight uses NMF seeded from reference marker genes: "Of these, SPOTlight3 uses cell-type marker genes derived from a single-cell RNA-sequencing (scRNA-seq) reference to seed a non-negative matrix factorization."
- [discussion] LDA performance depends on dataset size: "the performance of LDA in accurately deconvolving cell types depends on the size of the dataset with respect to the number of pixels and the number of genes"
- [readme] STdeconvolve installed via remotes: "remotes::install_github('JEFworks-Lab/STdeconvolve')"
- [intro] STdeconvolve applies LDA to infer cell-type profiles and proportions: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
