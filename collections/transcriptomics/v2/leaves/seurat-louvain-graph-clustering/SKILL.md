---
name: seurat-louvain-graph-clustering
description: Use when benchmarking clustering methods on UMI-based simulated scRNA-seq
  data — especially detecting a rare subpopulation — and you need a graph-based baseline
  to compare against k-means on PCA/scVI latent spaces and SIMLR.
when_to_use_negative:
- The task is trajectory inference rather than discrete-population clustering — the
  paper handles trajectories with dynwrap/dynmethods, not this benchmark
- Input data are non-UMI read counts; the benchmark described targets UMI-protocol
  simulations (True2ObservedCounts protocol='UMI')
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Louvain graph-based clustering of simulated cells; compared as a clustering
    method against k-means and SIMLR
- name: SymSim
  role: Generates the simulated UMI count data (SimulateTrueCounts + True2ObservedCounts)
    and true population labels on which clustering is evaluated
  repo: https://github.com/YosefLab/SymSim
- name: PCA
  role: Dimensionality reduction baseline whose latent space is clustered with k-means
    for comparison
- name: scVI
  role: Deep generative latent embedding clustered with k-means; the top-performing
    method in the benchmark
- name: SIMLR
  role: Alternative dimensionality reduction + clustering method included in the comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:bdd04bff5e64a0e30e04a435c9252b3b6b8856de11f84aa4ee9fabc04732347f
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# seurat-louvain-graph-clustering

## Summary

Applies Seurat's Louvain graph-based clustering to SymSim-simulated UMI count matrices as one of several clustering baselines (alongside PCA/k-means, scVI/k-means, and SIMLR) for benchmarking rare-population detection, scored by adjusted Rand index (ARI) against the true population labels.

## When to use

Use when benchmarking clustering methods on UMI-based simulated scRNA-seq data — especially detecting a rare subpopulation — and you need a graph-based baseline to compare against k-means on PCA/scVI latent spaces and SIMLR.

## When NOT to use

- The task is trajectory inference rather than discrete-population clustering — the paper handles trajectories with dynwrap/dynmethods, not this benchmark
- Input data are non-UMI read counts; the benchmark described targets UMI-protocol simulations (True2ObservedCounts protocol='UMI')

## Inputs

- UMI observed count matrix from SymSim True2ObservedCounts (protocol='UMI')
- True population labels for each simulated cell (from Phyla5 tree populations)
- Simulation parameter grid values: N (total cells), Sigma, alpha_mean/alpha_sd, depth_mean/depth_sd, Prop (rare-population proportion)

## Outputs

- Cluster assignments for each cell from Seurat Louvain graph-based clustering
- Adjusted Rand index (via cal_ARI) per method per grid combination
- ARI results table and regression coefficients of ARI on simulation parameters

## How to apply

Simulate data with SymSim by calling SimulateTrueCounts (evf_type='discrete', Phyla5 tree) over the Supplementary Table 6 grid of N, Sigma (e.g., 0.4), alpha_mean/alpha_sd, depth_mean/depth_sd, and rare-population proportion Prop, then convert to observed counts with True2ObservedCounts using protocol='UMI'. For each dataset, run Seurat's Louvain graph-based clustering alongside PCA/k-means, scVI/k-means, and SIMLR, keeping the number of clusters equal to the number of true populations. Compute the adjusted Rand index between predicted and true labels using cal_ARI for every method and grid combination, and assemble the ARI results into a results table. Optionally fit a multiple linear regression of ARI on (N, Sigma, alpha, Depth, Prop) to extract standardized coefficients for a Fig. 5a-style bar plot, and subset by Prop to plot ARI-vs-N rare-population detection curves. Rationale: the paper uses this design to test how within-population variability and mRNA capture efficiency affect clustering performance, finding that Seurat ranks second behind scVI+k-means in most settings (with prop-specific differences between Seurat and PCA/k-means at 5% vs 10% rare-cell proportions).

## Related tools

- **Seurat** (Louvain graph-based clustering of simulated cells; compared as a clustering method against k-means and SIMLR)
- **SymSim** (Generates the simulated UMI count data (SimulateTrueCounts + True2ObservedCounts) and true population labels on which clustering is evaluated) — https://github.com/YosefLab/SymSim
- **PCA** (Dimensionality reduction baseline whose latent space is clustered with k-means for comparison)
- **scVI** (Deep generative latent embedding clustered with k-means; the top-performing method in the benchmark)
- **SIMLR** (Alternative dimensionality reduction + clustering method included in the comparison)

## Examples

```
devtools::install_github("YosefLab/SymSim"); counts_res <- True2ObservedCounts(true_counts, meta_cell, protocol="UMI", alpha_mean=0.1, alpha_sd=0.05)
```

## Evaluation signals

- The ARI table contains exactly one row per grid combination × clustering method
- ARI increases with number of cells (N) and decreases as the rare-population proportion shrinks
- Seurat's ARI ranks second behind scVI+k-means in most settings, matching the reported Fig. 5 trends
- Number of clusters in k-means/Seurat output matches the number of true populations in the Phyla5 tree
- Standardized regression coefficients of ARI on (N, Sigma, alpha, Depth, Prop) qualitatively reproduce the Fig. 5a bar plot

## Limitations

- Clustering performance depends on simulation parameters; relative rankings shift with rare-population proportion (5% vs 10% changes Seurat's ordering vs PCA/k-means)
- The SymSim pipeline omits several steps, including reverse transcription and library cleaning up, so observed-count properties are approximate

## Evidence

- [discussion] dimensionality reduction methods for clustering benchmark: "We apply three different dimensionality reduction methods to cluster cells simulated from multiple discrete populations: PCA, scVI, and SIMLR."
- [abstract] rare population cell-count evaluation: "We also show how SymSim can be used to evaluate the number of cells required to detect a rare population under various scenarios."
- [readme] SymSim installation: "This package can be installed with R package devtools."
- [readme] SymSim package purpose: "SymSim (Synthetic model of multiple variability factors for Simulation) is an R package for simulation of single cell RNA-Seq data."
