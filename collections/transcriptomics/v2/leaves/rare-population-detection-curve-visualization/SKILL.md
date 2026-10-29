---
name: rare-population-detection-curve-visualization
description: Use when experimental-design questions ask how many cells (N) must be
  sequenced to detect a rare subpopulation (e.g., 5% or 10% of cells) and ARI-vs-N
  curves per method are needed.
when_to_use_negative:
- The goal is trajectory inference benchmarking (dynwrap/dynmethods territory), not
  rare-population detection power curves.
- Data are real experimental counts without known ground-truth population labels,
  so adjusted Rand index against true labels cannot be computed.
- The simulation uses non-UMI protocols or continuous EVF trajectories rather than
  the discrete-population UMI setting this curve analysis was validated on.
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: SymSim
  role: Simulates true counts (SimulateTrueCounts, evf_type='discrete', Phyla5 tree)
    and UMI observed counts (True2ObservedCounts) across the N/Prop grid
  repo: https://github.com/YosefLab/SymSim
- name: scVI
  role: Latent embedding fed to k-means; top-performing method in the rare-population
    detection curves
- name: PCA
  role: Baseline dimensionality reduction whose latent space is clustered with k-means
- name: SIMLR
  role: Dimensionality reduction and standalone clustering method compared in the
    ARI curves
- name: Seurat
  role: Louvain graph-based clustering compared in the ARI-vs-N curves
- name: RJAGS
  role: Gibbs sampler used to fit the regression of ARI on simulation parameters
- name: ggplot2
  role: Renders the ARI-vs-N detection curves and standardized coefficient bar plot
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:fcd085536c3940bcb1f8d7d2544f165d1b7f7059154b889c091a91da949797fd
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
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Rare-population detection curve visualization

## Summary

Plots adjusted Rand index (ARI) as a function of total cell number N (and simulation parameters like Sigma, alpha, depth, and rare-population proportion Prop) for each clustering method, reproducing Fig. 5b–c rare-population detection curves from SymSim UMI-based simulations.

## When to use

Use when experimental-design questions ask how many cells (N) must be sequenced to detect a rare subpopulation (e.g., 5% or 10% of cells) and ARI-vs-N curves per method are needed.

## When NOT to use

- The goal is trajectory inference benchmarking (dynwrap/dynmethods territory), not rare-population detection power curves.
- Data are real experimental counts without known ground-truth population labels, so adjusted Rand index against true labels cannot be computed.
- The simulation uses non-UMI protocols or continuous EVF trajectories rather than the discrete-population UMI setting this curve analysis was validated on.

## Inputs

- ARI results table with one row per simulation grid combination (N, Sigma, alpha, Depth, Prop) and clustering method (scVI+k-means, PCA+k-means, SIMLR, Seurat)
- SymSim UMI-observed count matrices simulated with SimulateTrueCounts + True2ObservedCounts (evf_type='discrete', Phyla5 tree)
- True population labels per cell and rare-population proportion Prop (e.g., 5%, 10%)

## Outputs

- ARI-versus-N detection curves faceted by Sigma, alpha, depth, and Prop (Fig. 5b–c analogs)
- Standardized regression coefficients of ARI on simulation parameters (Fig. 5a bar plot)
- Results table of cal_ARI values per method and grid combination

## How to apply

Fit SymSim simulations across the Supplementary Table 6 grid (N, Sigma=0.4, alpha_mean/alpha_sd, depth_mean/depth_sd, Prop) using SimulateTrueCounts with evf_type='discrete' and the Phyla5 tree, then True2ObservedCounts with protocol='UMI'. Embed cells with PCA, scVI, and SIMLR, cluster with k-means (k = number of true populations), Seurat Louvain clustering, and SIMLR, and compute cal_ARI against true labels. Fit a multiple linear regression of ARI on N, Sigma, alpha, Depth, and Prop with RJAGS/ggplot2 to obtain standardized coefficients (Fig. 5a bar plot). Finally subset the ARI table by Prop and plot ARI versus N (and other parameters) per method with ggplot2 to reproduce Fig. 5b–c, where curves should show ARI increasing with N and decreasing as the rare population shrinks from 10% to 5%.

## Related tools

- **SymSim** (Simulates true counts (SimulateTrueCounts, evf_type='discrete', Phyla5 tree) and UMI observed counts (True2ObservedCounts) across the N/Prop grid) — https://github.com/YosefLab/SymSim
- **scVI** (Latent embedding fed to k-means; top-performing method in the rare-population detection curves)
- **PCA** (Baseline dimensionality reduction whose latent space is clustered with k-means)
- **SIMLR** (Dimensionality reduction and standalone clustering method compared in the ARI curves)
- **Seurat** (Louvain graph-based clustering compared in the ARI-vs-N curves)
- **RJAGS** (Gibbs sampler used to fit the regression of ARI on simulation parameters)
- **ggplot2** (Renders the ARI-vs-N detection curves and standardized coefficient bar plot)

## Examples

```
devtools::install_github('YosefLab/SymSim'); phyla1 <- Phyla5(); true_counts_res <- SimulateTrueCounts(nvar=1000, nsample=Prop*N, ...)
```

## Evaluation signals

- ARI table contains exactly one row per grid combination and clustering method across N, Sigma, alpha, Depth, Prop.
- Fig. 5b–c analog curves qualitatively show ARI increasing with number of cells N.
- At Prop=5% Seurat is second best after scVI+k-means; at Prop=10% Seurat and PCA/k-means ordering shifts as reported.
- Standardized regression coefficients show the expected sign for N (positive) and smaller rare-population proportion (lower ARI).
- Simulated counts reproduce SymSim properties (high zero abundance, protocol-specific noise) before clustering, confirming True2ObservedCounts ran with protocol='UMI'.

## Limitations

- Detection curves depend on Supplementary Table 6 grid parameters; Supplementary Tables 2–5 fit parameters are referenced but not included in the provided text.
- scVI imputation/analysis is only applicable to large datasets; the article notes scVI was not applied to small non-UMI data.
- SymSim's pipeline omits several steps, including reverse transcription and library cleanup, which bounds the biological fidelity of the simulated curves.

## Evidence

- [abstract] SymSim can be used to evaluate the number of cells required to detect a rare population under various scenarios: "We also show how SymSim can be used to evaluate the number of cells required to detect a rare population under various scenarios."
- [discussion] Dimensionality reduction and clustering methods applied to simulated discrete populations: "We apply three different dimensionality reduction methods to cluster cells simulated from multiple discrete populations: PCA, scVI, and SIMLR."
- [methods] Two major SymSim functions generating datasets: "**SimulateTrueCounts( )** and **True2ObservedCounts( )** are the two major functions to generate datasets."
- [readme] Installing SymSim from GitHub: "devtools::install_github("YosefLab/SymSim")"
- [methods] Tree with five leaves for multiple discrete populations: "A tree with five leaves (five populations) can be generated as follows"
