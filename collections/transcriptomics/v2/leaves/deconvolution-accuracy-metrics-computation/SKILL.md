---
name: deconvolution-accuracy-metrics-computation
description: Use when simulated pixel-resolution spatial transcriptomics data with
  known ground-truth cell-type proportions and profiles is available and you must
  quantify deconvolution accuracy via RMSE of proportions and correlations of inferred
  vs true profiles.
when_to_use_negative:
- No ground truth exists (real tissue ST data) — RMSE and correlation against true
  proportions/profiles cannot be computed
- Data is sub-cellular pixel-resolution ST, where the paper states STdeconvolve 'may
  not be best suited' and new methods are needed
- Cell-type proportions are homogeneous/uniform across pixels so genes lack variability,
  a documented failure mode for LDA
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: STdeconvolve
  role: Reference-free LDA deconvolution used to infer per-pixel cell-type proportions
    (theta) and transcriptional profiles (beta) at K=9 for accuracy assessment
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Language in which the STdeconvolve package is distributed and in which RMSE/correlation
    metrics are computed
- name: BiocManager
  role: Installs STdeconvolve from Bioconductor
- name: remotes
  role: Installs the latest development version of STdeconvolve from GitHub
  repo: https://github.com/JEFworks-Lab/STdeconvolve
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:96bb2d51c474ccf5adbb31a67484d55887f4e65002ad349bc34b5dd45e082ae2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JEFworks-Lab/STdeconvolve
  tools:
  - BiocManager
  - R
  - STdeconvolve
  - remotes
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# deconvolution-accuracy-metrics-computation

## Summary

Quantify how well STdeconvolve's LDA-based, reference-free deconvolution recovers cell-type proportions and transcriptional profiles by fitting the model at a fixed K on simulated MERFISH-derived 100 µm2 MPOA ST pixels (n=3072) and computing proportion RMSE plus profile/proportion correlations against known ground truth.

## When to use

Use when simulated pixel-resolution spatial transcriptomics data with known ground-truth cell-type proportions and profiles is available and you must quantify deconvolution accuracy via RMSE of proportions and correlations of inferred vs true profiles.

## When NOT to use

- No ground truth exists (real tissue ST data) — RMSE and correlation against true proportions/profiles cannot be computed
- Data is sub-cellular pixel-resolution ST, where the paper states STdeconvolve 'may not be best suited' and new methods are needed
- Cell-type proportions are homogeneous/uniform across pixels so genes lack variability, a documented failure mode for LDA

## Inputs

- Pixel-by-gene count matrix from single-cell MERFISH MPOA data aggregated into 100 µm2 pixels (3072 pixels)
- Ground-truth per-pixel cell-type proportions from the simulation
- Ground-truth cell-type transcriptional profiles
- Known number of cell types (K=9)

## Outputs

- deconvolution_accuracy.csv containing proportion RMSE and profile/proportion correlation values over all 3072 pixels
- Per-pixel deconvolved cell-type proportion matrix (theta) and cell-type transcriptional profile matrix (beta)

## How to apply

Aggregate single-cell MERFISH MPOA transcript counts into 100 µm2 pixels to build a pixel-by-gene count matrix (n=3072 pixels) with known ground-truth cell-type proportions, then feature-select significantly overdispersed genes across pixels so the corpus is suitable for LDA. Fit the STdeconvolve latent Dirichlet allocation model with the number of topics fixed at K=9 (use fitLDA with Ks covering the known K and optimalModel to select it), and extract per-pixel cell-type proportions (theta) and cell-type transcriptional profiles (beta) with getBetaTheta. Match inferred cell types to ground-truth cell types (e.g., by profile correlation), then compute the RMSE of deconvolved proportions across all 3072 pixels and the correlations between inferred and ground-truth transcriptional profiles and proportions. Write the metrics to a table such as deconvolution_accuracy.csv; correct application yields high correlations and low RMSE, consistent with the paper's reported simulation performance.

## Related tools

- **STdeconvolve** (Reference-free LDA deconvolution used to infer per-pixel cell-type proportions (theta) and transcriptional profiles (beta) at K=9 for accuracy assessment) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Language in which the STdeconvolve package is distributed and in which RMSE/correlation metrics are computed)
- **BiocManager** (Installs STdeconvolve from Bioconductor)
- **remotes** (Installs the latest development version of STdeconvolve from GitHub) — https://github.com/JEFworks-Lab/STdeconvolve

## Examples

```
ldas <- fitLDA(t(as.matrix(corpus)), Ks = 9); results <- getBetaTheta(optimalModel(models = ldas, opt = 'min'), perc.filt = 0.05, betaScale = 1000)
```

## Evaluation signals

- deconvolution_accuracy.csv contains a proportion RMSE value computed over all 3072 pixels
- Correlations between inferred and ground-truth transcriptional profiles and between inferred and ground-truth proportions are computed and are high
- Proportion RMSE is low, consistent with STdeconvolve's reported simulation performance
- Number of fitted topics equals the known K=9 cell types and inferred cell types can be matched one-to-one to ground-truth cell types
- Beta matrix (profiles) and theta matrix (proportions) are extracted from the optimal LDA model with getBetaTheta and have dimensions consistent with 9 cell types and the filtered corpus

## Limitations

- LDA performance depends on dataset size in terms of number of pixels and number of genes; assumptions may not hold for small datasets
- If gene expression is too sparse with high stochastic drop-out rates, LDA may struggle to identify distinct groups of co-expressed genes
- STdeconvolve may fail to deconvolve transcriptionally distinct cell types when cell-type proportions are homogeneous/uniform across pixels so genes lack variability
- ST profiling of tissue slivers or thin structures covering only a few pixels presents challenges for deconvolution
- Number of cells per pixel depends on cell size, which varies by organism, tissue, and disease state

## Evidence

- [intro] STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [intro] Overdispersed gene feature selection supports LDA application: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
- [abstract] Simulation benchmark quantifies accuracy via RMSE and correlations: "Using simulated as well as real ST datasets from diverse spatial transcriptomics technologies comprising a variety of spatial resolutions such as Spatial Transcriptomics, 10X Visium, DBiT-seq, and"
- [readme] README example shows the extract-theta-beta step used in the metric computation: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)
deconProp <- results$theta
deconGexp <- results$beta"
- [discussion] Sparse-data failure mode for LDA: "if the gene expression in the ST data is too sparse with high rates of stochastic drop-outs40, then the LDA model may struggle to identify distinct groups of co-expressed genes"
