---
name: perplexity-based-k-selection
description: Use when deconvolving multi-cellular pixel-resolution spatial transcriptomics
  data with STdeconvolve and an appropriate number of cell types K must be set a priori
  for the LDA model.
when_to_use_negative:
- Input is sub-cellular pixel-resolution ST data — STdeconvolve is not best suited
  for such resolutions
- Genes show no variability across pixels because cell-type proportions are homogeneous/uniform,
  so deconvolution is expected to fail regardless of K
- A curated single-cell RNA-seq reference exists and a supervised method (RCTD, SPOTlight,
  spatialDWLS) is preferred
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: STdeconvolve
  role: Provides fitLDA to fit LDA models over a K grid and optimalModel to pick the
    minimal-perplexity model
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Execution environment; STdeconvolve is an open-source R package (R >= 4.2
    via Bioconductor release 3.15)
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Bioconductor installer for the STdeconvolve package
- name: remotes
  role: 'Alternative installer from the GitHub development branch: remotes::install_github(''JEFworks-Lab/STdeconvolve'')'
  repo: https://github.com/JEFworks-Lab/STdeconvolve
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:37c04cd3d783f89ae416160adb331d970ddb3b3d499f327f46213deabd272749
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# perplexity-based-k-selection

## Summary

Data-driven selection of the number of cell types K in STdeconvolve by fitting latent Dirichlet allocation models over a grid of candidate K values (topicmodels backend), recording perplexity for each K, and choosing the K with minimal perplexity that avoids rare, low-proportion topics.

## When to use

Use when deconvolving multi-cellular pixel-resolution spatial transcriptomics data with STdeconvolve and an appropriate number of cell types K must be set a priori for the LDA model.

## When NOT to use

- Input is sub-cellular pixel-resolution ST data — STdeconvolve is not best suited for such resolutions
- Genes show no variability across pixels because cell-type proportions are homogeneous/uniform, so deconvolution is expected to fail regardless of K
- A curated single-cell RNA-seq reference exists and a supervised method (RCTD, SPOTlight, spatialDWLS) is preferred

## Inputs

- Feature-selected gene-pixel count corpus (corpus) produced by cleanCounts and restrictCorpus, e.g. the top 1000 significantly overdispersed genes
- Grid of candidate K values (e.g. 2 through 9)

## Outputs

- Set of fitted LDA models with recorded perplexity per candidate K (ldas)
- A single optimal LDA model (optLDA) at the selected K, ready for getBetaTheta extraction of beta and theta matrices

## How to apply

After feature selection (genes expressed in 5-95% of pixels, top significantly overdispersed genes), fit LDA models across a grid of candidate K values — e.g. Ks = seq(2, 9, by = 1) in the mOB example — using STdeconvolve's fitLDA, recording perplexity for each K. Select the optimal model with optimalModel(models = ldas, opt = 'min'), i.e. the K minimizing perplexity. As a sanity check, require that no rare, low-proportion cell-type topic appears at the chosen K; if it does, prefer an adjacent K. STdeconvolve provides these data-driven metrics because K cannot be estimated from the data by LDA itself and must be fixed before fitting.

## Related tools

- **STdeconvolve** (Provides fitLDA to fit LDA models over a K grid and optimalModel to pick the minimal-perplexity model) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Execution environment; STdeconvolve is an open-source R package (R >= 4.2 via Bioconductor release 3.15)) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Bioconductor installer for the STdeconvolve package)
- **remotes** (Alternative installer from the GitHub development branch: remotes::install_github('JEFworks-Lab/STdeconvolve')) — https://github.com/JEFworks-Lab/STdeconvolve

## Examples

```
library(STdeconvolve); ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1)); optLDA <- optimalModel(models = ldas, opt = "min")
```

## Evaluation signals

- fitLDA returns one fitted model per candidate K, each with a recorded perplexity value; perplexity should be traceable across the K grid
- The chosen K corresponds to the minimum perplexity among candidates without unstable (rare, low-proportion) topics
- The final LDA at the selected K yields a beta matrix of dimensions K x N' (genes) and a theta matrix of dimensions D pixels x K
- Theta rows (per-pixel cell-type proportions) are valid proportions summing to 1 across K
- A rare topic at the chosen K (very low average theta across pixels) flags an incorrect K choice warranting re-selection

## Limitations

- LDA performance depends on the size of the dataset with respect to the number of pixels and the number of genes
- ST profiling of tissue slivers or thin structures covering only a few pixels may present challenges to deconvolution
- Highly sparse data with high stochastic drop-out rates can hinder the LDA model's ability to identify distinct groups of co-expressed genes

## Evidence

- [intro] Data-driven K selection metrics: "STdeconvolve provides several data-driven metrics to guide the estimation of an appropriate K"
- [readme] fitLDA grid over candidate K: "ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1))"
- [readme] optimalModel minimizes perplexity: "optLDA <- optimalModel(models = ldas, opt = "min")"
- [intro] Overdispersed-gene feature selection feeding K selection: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels"
