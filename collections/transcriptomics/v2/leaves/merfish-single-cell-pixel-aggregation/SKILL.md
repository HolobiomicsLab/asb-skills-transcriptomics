---
name: merfish-single-cell-pixel-aggregation
description: 'Use when benchmarking pixel-resolution spatial transcriptomics deconvolution
  against known single-cell composition: single-cell MERFISH counts must be aggregated
  into simulated multi-cellular pixels with ground-truth cell-type proportions for
  RMSE/correlation evaluation.'
when_to_use_negative:
- Input is already a pixel-resolution ST count matrix (e.g. 10X Visium or Spatial
  Transcriptomics spots) — no single-cell aggregation step is needed
- The goal is deconvolving sub-cellular resolution ST data, which the authors state
  requires new methods rather than multi-cellular pixel deconvolution
- Cell-type proportions are homogeneous across pixels so genes lack variability —
  STdeconvolve may fail to deconvolve, so a benchmark built this way would not be
  informative
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STdeconvolve
  role: Reference-free LDA deconvolution applied to the simulated pixel-by-gene count
    matrix to infer cell-type proportions (theta) and transcriptional profiles (beta)
    at K=9 for benchmarking
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Language in which the aggregation, simulation, and STdeconvolve benchmark
    are implemented
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: BiocManager
  role: Installs the STdeconvolve package from Bioconductor
  repo: https://github.com/JEFworks-Lab/STdeconvolve
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
  source_content_hash: sha256:0380417d2531d66c5fabe0794977a134d14c5eb21ca0d0057feec174122229f0
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
yes_rate: 0.8666666666666667
n_claims: 15
---

# merfish-single-cell-pixel-aggregation

## Summary

Aggregate single-cell MERFISH transcript counts of the mouse MPOA into simulated 100 µm2-resolution spatial transcriptomics pixels (pixel-by-gene count matrix, n=3072 pixels) with known ground-truth cell-type proportions, enabling benchmarking of STdeconvolve's reference-free LDA deconvolution against ground truth.

## When to use

Use when benchmarking pixel-resolution spatial transcriptomics deconvolution against known single-cell composition: single-cell MERFISH counts must be aggregated into simulated multi-cellular pixels with ground-truth cell-type proportions for RMSE/correlation evaluation.

## When NOT to use

- Input is already a pixel-resolution ST count matrix (e.g. 10X Visium or Spatial Transcriptomics spots) — no single-cell aggregation step is needed
- The goal is deconvolving sub-cellular resolution ST data, which the authors state requires new methods rather than multi-cellular pixel deconvolution
- Cell-type proportions are homogeneous across pixels so genes lack variability — STdeconvolve may fail to deconvolve, so a benchmark built this way would not be informative

## Inputs

- Single-cell MERFISH transcript count matrix of the mouse MPOA (cell-by-gene counts)
- Spatial coordinates of single cells (for assigning cells to 100 µm2 pixel bins)
- Ground-truth single-cell type annotations (for computing per-pixel true proportions)

## Outputs

- Pixel-by-gene count matrix from 3072 simulated 100 µm2 MPOA pixels
- Ground-truth per-pixel cell-type proportion matrix
- deconvolution_accuracy.csv containing proportion RMSE and profile/proportion correlation metrics over all 3072 pixels

## How to apply

Aggregate per-cell MERFISH transcript counts of the mouse MPOA into 100 µm2 pixels to construct a pixel-by-gene count matrix (n=3072 pixels) while recording the true per-pixel cell-type proportions from cell centroids. Clean the counts by removing pixels with too few genes and feature-select significantly overdispersed genes across pixels so the count matrix is suitable for LDA. Fit the STdeconvolve latent Dirichlet allocation model with the number of topics fixed at K=9, then match inferred cell types (theta proportions, beta transcriptional profiles) to ground-truth cell types. Quantify accuracy via the RMSE of deconvolved proportions across pixels plus correlations between inferred and ground-truth transcriptional profiles and proportions, writing results to deconvolution_accuracy.csv.

## Related tools

- **STdeconvolve** (Reference-free LDA deconvolution applied to the simulated pixel-by-gene count matrix to infer cell-type proportions (theta) and transcriptional profiles (beta) at K=9 for benchmarking) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Language in which the aggregation, simulation, and STdeconvolve benchmark are implemented) — https://github.com/JEFworks-Lab/STdeconvolve
- **BiocManager** (Installs the STdeconvolve package from Bioconductor) — https://github.com/JEFworks-Lab/STdeconvolve
- **remotes** (Installs the latest STdeconvolve development version from GitHub) — https://github.com/JEFworks-Lab/STdeconvolve

## Examples

```
library(STdeconvolve); counts <- cleanCounts(cd, min.lib.size = 100); corpus <- restrictCorpus(counts, removeAbove=1.0, removeBelow = 0.05); ldas <- fitLDA(t(as.matrix(corpus)), Ks = 9); results <- getBetaTheta(optimalModel(models = ldas), perc.filt = 0.05, betaScale = 1000)
```

## Evaluation signals

- Output table deconvolution_accuracy.csv contains a proportion RMSE value and profile/proportion correlation values computed over all 3072 pixels
- Deconvolved vs ground-truth transcriptional profiles and cell-type proportions show strong (high) correlations
- Proportion RMSE across pixels is low, consistent with the reported simulation performance of STdeconvolve
- LDA model fitted with K=9 topics matches the number of ground-truth cell types recovered in the simulation
- Preprocessing succeeds: pixels with too few genes are removed and only significantly overdispersed genes enter the LDA corpus

## Limitations

- LDA performance depends on dataset size with respect to the number of pixels and number of genes, which may limit deconvolution when assumptions are not satisfied
- If gene expression in the ST data is too sparse with high rates of stochastic drop-outs, the LDA model may struggle to identify distinct groups of co-expressed genes
- If genes do not exhibit variability across pixels due to homogeneous or uniform proportional representation of cell types, STdeconvolve may fail to deconvolve

## Evidence

- [intro] STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel: "STdeconvolve applies LDA to infer the putative transcriptional profile for each cell type and the proportional representation of each cell type in each multi-cellular ST pixel"
- [intro] Overdispersed gene feature selection for LDA: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
- [readme] Pixel cleaning and feature selection commands: "counts <- cleanCounts(cd, min.lib.size = 100)"
- [readme] LDA fitting and extraction of beta/theta: "results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000)"
