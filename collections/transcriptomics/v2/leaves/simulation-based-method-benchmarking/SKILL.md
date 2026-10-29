---
name: simulation-based-method-benchmarking
description: Use when comparing cell-type deconvolution or clustering methods on spatial
  transcriptomics data where ground truth is unavailable in real tissue, by simulating
  multi-cellular pixels with known cell-type proportions and evaluating RMSE of estimated
  proportions.
when_to_use_negative:
- Real datasets lacking simulated ground truth — per-pixel RMSE against true proportions
  cannot be computed
- Sub-cellular resolution ST data, where the paper notes STdeconvolve may not be best
  suited and new methods will be needed
- Datasets where cell-type proportions are homogeneous across pixels — the paper reports
  STdeconvolve may fail to deconvolve when genes lack variability across pixels
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2269
tools:
- name: STdeconvolve
  role: Reference-free LDA-based deconvolution method under evaluation; run on overdispersed-gene-filtered
    counts with K set to the known number of simulated cell types
  repo: https://github.com/JEFworks-Lab/STdeconvolve
- name: R
  role: Implementation environment for simulation, deconvolution, RMSE computation,
    Diebold-Mariano testing, and figure generation
- name: BiocManager
  role: Used to install STdeconvolve and Bioconductor dependencies (e.g., BayesSpace)
    for the benchmark
- name: remotes
  role: Installs the development version of STdeconvolve from GitHub
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30033-z
    title: Reference-free cell type deconvolution of multi-cellular pixel-resolution
      spatially resolved transcriptomics data
  source_content_hash: sha256:06f71cecf2563e51431b9faaf73ffbac903b4a7ddbf39b0bddc89409906f12a0
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# simulation-based-method-benchmarking

## Summary

Benchmark a deconvolution method against competing approaches (e.g., regular transcriptional clustering and BayesSpace resolution-enhanced clustering) on simulated multi-cellular pixel-resolution ST data with known ground-truth cell-type proportions, quantifying per-pixel RMSE and testing significance with the Diebold-Mariano test.

## When to use

Use when comparing cell-type deconvolution or clustering methods on spatial transcriptomics data where ground truth is unavailable in real tissue, by simulating multi-cellular pixels with known cell-type proportions and evaluating RMSE of estimated proportions.

## When NOT to use

- Real datasets lacking simulated ground truth — per-pixel RMSE against true proportions cannot be computed
- Sub-cellular resolution ST data, where the paper notes STdeconvolve may not be best suited and new methods will be needed
- Datasets where cell-type proportions are homogeneous across pixels — the paper reports STdeconvolve may fail to deconvolve when genes lack variability across pixels

## Inputs

- Simulated 3-cell-type multi-cellular pixel-resolution spatial transcriptomics (ST) dataset with ground-truth per-pixel cell-type proportions
- Simulated 100 um2-pixel MERFISH mouse brain dataset with thalamus-region subset (n=716 pixels) and ground-truth proportions
- Count matrices (pixels x genes) and pixel spatial coordinates for each simulated dataset

## Outputs

- Per-pixel RMSE values of estimated vs ground-truth cell-type proportions for each method (STdeconvolve, clustering, BayesSpace)
- Diebold-Mariano test p-values for pairwise method comparisons
- Comparison figure panels (Fig. 3h ST, Fig. 3l MERFISH thalamus inset)
- CSV table of RMSE and DM statistics (fig3_rmse_comparison.csv)

## How to apply

Simulate datasets with known ground-truth cell-type proportions — here, a 3-cell-type pixel-resolution ST dataset and a 100 um2-pixel MERFISH mouse brain dataset including a thalamus-region subset of n=716 pixels. Run the method under evaluation (STdeconvolve, after selecting significantly overdispersed genes and fitting LDA with the known number of topics K=3) plus the competitors (transcriptional clustering and BayesSpace resolution-enhanced clustering, mapping cluster assignments to per-pixel cell-type proportions). Compute per-pixel RMSE of estimated versus ground-truth proportions for every method on both datasets. Apply the Diebold-Mariano test for pairwise accuracy comparison, confirming significant improvement of the reference-free deconvolution (p < 2.2e-16). Render comparison panels (Fig. 3h for ST, Fig. 3l for MERFISH) and export RMSE and DM statistics to a CSV such as fig3_rmse_comparison.csv.

## Related tools

- **STdeconvolve** (Reference-free LDA-based deconvolution method under evaluation; run on overdispersed-gene-filtered counts with K set to the known number of simulated cell types) — https://github.com/JEFworks-Lab/STdeconvolve
- **R** (Implementation environment for simulation, deconvolution, RMSE computation, Diebold-Mariano testing, and figure generation)
- **BiocManager** (Used to install STdeconvolve and Bioconductor dependencies (e.g., BayesSpace) for the benchmark)
- **remotes** (Installs the development version of STdeconvolve from GitHub)

## Examples

```
results <- getBetaTheta(optLDA, perc.filt = 0.05, betaScale = 1000); deconProp <- results$theta
```

## Evaluation signals

- STdeconvolve achieves the lowest per-pixel RMSE of deconvolved cell-type proportions on both the simulated 3-cell-type ST data and the simulated MERFISH mouse brain data
- Diebold-Mariano pairwise comparisons of STdeconvolve vs transcriptional clustering and vs BayesSpace yield p < 2.2e-16
- Reproduced panels visually match Fig. 3h (ST) and Fig. 3l (MERFISH thalamus inset, n=716 pixels)
- fig3_rmse_comparison.csv contains RMSE and DM statistics for all three methods on both simulated datasets
- Number of LDA topics equals the known simulated K=3 cell types

## Limitations

- Results depend on the simulation design; RMSE conclusions hold for the simulated 3-cell-type ST and 100 um2-pixel MERFISH (thalamus) settings used in the paper
- LDA performance depends on dataset size in terms of number of pixels and number of genes
- Sparse ST data with high stochastic drop-out rates may cause the LDA model to struggle to identify distinct groups of co-expressed genes
- STdeconvolve may fail to deconvolve transcriptionally distinct cell types when cell-type proportions are homogeneous/uniform across pixels

## Evidence

- [intro] STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14: "STdeconvolve selects for significantly overdispersed genes, or genes with higher-than-expected expression variance across ST pixels14"
- [discussion] LDA performance depends on dataset size: "the performance of LDA in accurately deconvolving cell types depends on the size of the dataset with respect to the number of pixels and the number of genes"
- [readme] README installation and LDA workflow: "require(remotes)
remotes::install_github('JEFworks-Lab/STdeconvolve')"
- [readme] README deconvolution example: "ldas <- fitLDA(t(as.matrix(corpus)), Ks = seq(2, 9, by = 1))"
