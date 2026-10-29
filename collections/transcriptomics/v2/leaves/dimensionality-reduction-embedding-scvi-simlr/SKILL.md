---
name: dimensionality-reduction-embedding-scvi-simlr
description: Use when you have simulated or observed scRNA-seq count matrices (e.g.
  SymSim UMI counts with known population labels) and need to compare embedding methods
  (PCA, scVI, SIMLR) for clustering accuracy or rare-population detection.
when_to_use_negative:
- The data are non-UMI counts on a very small dataset — the article notes scVI is
  only applicable to large datasets, and non-UMI data were instead imputed with MAGIC
- The task is trajectory inference rather than discrete population clustering — use
  the dynwrap/dynmethods framework instead
- No ground-truth population labels are available, making adjusted Rand index computation
  impossible
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: PCA
  role: Baseline linear dimensionality reduction providing a latent space for k-means
    clustering of simulated cells
- name: scVI
  role: Generative-model-based dimensionality reduction whose latent space, followed
    by k-means, achieves the highest ARI in most settings
- name: SIMLR
  role: Kernel-based dimensionality reduction with its own clustering, applied as
    a competing embedding/clustering method
- name: Seurat
  role: Louvain graph-based clustering applied as an additional clustering method
    on the simulated datasets
- name: SymSim
  role: Generates the UMI count matrices and true population labels that are embedded
    and clustered
  repo: https://github.com/YosefLab/SymSim
- name: ggplot2
  role: Plots ARI vs N and other parameters to reproduce Fig. 5b-c rare-population
    detection curves
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:b6c2992c99b78c8ff8e554d81104d643a3c1a02b486c77f7a79ed93ea31247d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - PCA
  - RJAGS
  - SIMLR
  - Seurat
  - SymSim
  - ggplot2
  - scVI
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# dimensionality-reduction-embedding-scvi-simlr

## Summary

Embed simulated single-cell RNA-seq datasets in low-dimensional space with PCA, scVI, and SIMLR, then cluster the embeddings to benchmark which dimensionality-reduction strategy best recovers true cell populations, e.g. rare subpopulations in UMI-based SymSim simulations.

## When to use

Use when you have simulated or observed scRNA-seq count matrices (e.g. SymSim UMI counts with known population labels) and need to compare embedding methods (PCA, scVI, SIMLR) for clustering accuracy or rare-population detection.

## When NOT to use

- The data are non-UMI counts on a very small dataset — the article notes scVI is only applicable to large datasets, and non-UMI data were instead imputed with MAGIC
- The task is trajectory inference rather than discrete population clustering — use the dynwrap/dynmethods framework instead
- No ground-truth population labels are available, making adjusted Rand index computation impossible

## Inputs

- SymSim-simulated UMI observed count matrices from True2ObservedCounts (protocol='UMI')
- True cell population labels from SimulateTrueCounts meta cell information
- Simulation parameter grid values: N (total cells), Sigma (e.g. 0.4), alpha_mean/alpha_sd, depth_mean/depth_sd, rare-population proportion Prop (e.g. 5%, 10%)

## Outputs

- Low-dimensional cell embeddings from PCA, scVI, and SIMLR
- Cluster assignments from k-means (PCA/scVI spaces), Seurat Louvain clustering, and SIMLR
- ARI results table with one row per grid combination and clustering method
- Fig. 5a-style standardized regression coefficients and Fig. 5b-c ARI-vs-N curves

## How to apply

For each simulated dataset (generated with SymSim's SimulateTrueCounts using evf_type='discrete' and a tree such as Phyla5, then converted with True2ObservedCounts using protocol='UMI'), embed cells with three dimensionality-reduction methods: PCA, scVI, and SIMLR. In the PCA and scVI latent spaces, cluster cells with k-means, setting the number of clusters equal to the true number of populations; additionally cluster with Seurat Louvain graph-based clustering and with SIMLR's built-in clustering. Score every method/parameter combination using the adjusted Rand index (cal_ARI) between predicted and true population labels, and assemble the ARI results into a table indexed by the simulation grid (N, Sigma, alpha, Depth, rare-population proportion Prop). Rationale: the article shows scVI followed by k-means achieves the highest adjusted Rand index in most settings, while performance of Seurat varies with the rare-population proportion (5% vs 10%).

## Related tools

- **PCA** (Baseline linear dimensionality reduction providing a latent space for k-means clustering of simulated cells)
- **scVI** (Generative-model-based dimensionality reduction whose latent space, followed by k-means, achieves the highest ARI in most settings)
- **SIMLR** (Kernel-based dimensionality reduction with its own clustering, applied as a competing embedding/clustering method)
- **Seurat** (Louvain graph-based clustering applied as an additional clustering method on the simulated datasets)
- **SymSim** (Generates the UMI count matrices and true population labels that are embedded and clustered) — https://github.com/YosefLab/SymSim
- **ggplot2** (Plots ARI vs N and other parameters to reproduce Fig. 5b-c rare-population detection curves)

## Examples

```
devtools::install_github('YosefLab/SymSim'); sim <- True2ObservedCounts(SimulateTrueCounts(nCells=1000, nGene=1000, evf_type='discrete', phyla=Phyla5()), protocol='UMI')
```

## Evaluation signals

- The ARI results table contains one row per grid combination and clustering method
- scVI + k-means yields the highest adjusted Rand index in most simulation settings
- ARI increases with number of cells N and decreases with smaller rare-population proportion Prop, matching Fig. 5 trends
- The fitted regression's standardized coefficients qualitatively reproduce the Fig. 5a bar plot ordering of parameter effects
- k-means cluster count matches the true number of populations in each simulated dataset

## Limitations

- scVI is only applicable to large datasets, so imputation/embedding alternatives (e.g. MAGIC) may be needed for small non-UMI data
- Results are simulation-based; SymSim omits several steps including reverse transcription and library cleanup, so absolute ARI values may not transfer to all real datasets
- Method ranking depends on the rare-population proportion: at Prop=5% Seurat is second best, but at 10% the relative ordering shifts

## Evidence

- [discussion] Three dimensionality reduction methods applied to discrete populations: "We apply three different dimensionality reduction methods to cluster cells simulated from multiple discrete populations: PCA, scVI, and SIMLR."
- [abstract] SymSim for benchmarking clustering: "We demonstrate how SymSim can be used for benchmarking methods for clustering, differential expression and trajectory inference, and for examining the effects of various parameters on their"
- [abstract] Rare population detection evaluation: "We also show how SymSim can be used to evaluate the number of cells required to detect a rare population under various scenarios."
- [readme] SymSim installation via GitHub: "devtools::install_github("YosefLab/SymSim")"
- [methods] True2ObservedCounts major function: "**SimulateTrueCounts( )** and **True2ObservedCounts( )** are the two major functions to generate datasets."
