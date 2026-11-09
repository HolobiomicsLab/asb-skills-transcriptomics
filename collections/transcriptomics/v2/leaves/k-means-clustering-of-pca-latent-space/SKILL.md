---
name: k-means-clustering-of-pca-latent-space
description: Use when simulated (or real) scRNA-seq count matrices must be partitioned
  into known discrete cell populations and a simple, reproducible PCA + k-means baseline
  is needed for benchmarking against scVI, SIMLR, or Seurat Louvain clustering.
when_to_use_negative:
- Input is already an embedding or feature table rather than a raw count matrix requiring
  PCA
- The task is trajectory inference rather than clustering of discrete populations
  — the article addresses trajectories with other benchmarking machinery (dynwrap/dynmethods)
- The data are non-UMI and very small; the article notes scVI-type deep models are
  inapplicable to small datasets and dimensionality-reduction choices differ for non-UMI
  protocols
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: Simulates true transcript counts (SimulateTrueCounts) and observed UMI counts
    (True2ObservedCounts) over the sigma/alpha/Prop parameter grid for the benchmark
  repo: https://github.com/YosefLab/SymSim
- name: PCA
  role: Dimensionality reduction producing the latent space in which k-means is run;
    one of three reduction methods evaluated alongside scVI and SIMLR
- name: SIMLR
  role: Alternative dimensionality reduction and clustering method used as comparison
    baseline
- name: scVI
  role: Deep-learning latent embedding whose k-means clustering is the top-performing
    comparator in most settings
- name: Seurat
  role: Louvain graph-based clustering baseline compared against PCA+K-means ARI
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:b4299073a56a4722504f5eec949e42ac27065479a078e3bdde0afeae2e53b2d1
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
yes_rate: 0.8
n_claims: 10
---

# k-means clustering of PCA latent space

## Summary

Cluster cells from single-cell RNA-seq data by embedding them in a PCA-reduced latent space and running k-means with the number of centers set to the true number of populations, then scoring recovery with adjusted Rand index (ARI). This was one of the dimensionality-reduction-plus-clustering baselines benchmarked against scVI/k-means, SIMLR, and Seurat on SymSim-simulated UMI data.

## When to use

Use when simulated (or real) scRNA-seq count matrices must be partitioned into known discrete cell populations and a simple, reproducible PCA + k-means baseline is needed for benchmarking against scVI, SIMLR, or Seurat Louvain clustering.

## When NOT to use

- Input is already an embedding or feature table rather than a raw count matrix requiring PCA
- The task is trajectory inference rather than clustering of discrete populations — the article addresses trajectories with other benchmarking machinery (dynwrap/dynmethods)
- The data are non-UMI and very small; the article notes scVI-type deep models are inapplicable to small datasets and dimensionality-reduction choices differ for non-UMI protocols

## Inputs

- Observed UMI count matrix (genes x cells) from SymSim True2ObservedCounts with protocol='UMI'
- True cell-to-population label vector (from the SymSim simulation metadata, e.g. Phyla5 tree leaves)
- Number of true populations (used as the k-means number of centers)

## Outputs

- Cluster assignment vector for each cell from k-means in the PCA latent space
- Adjusted Rand index (via cal_ARI) per dataset and method, assembled into an ARI results table
- Standardized regression coefficients of ARI on simulation parameters (N, Sigma, alpha, Depth, Prop)

## How to apply

Simulate observed UMI counts with SymSim by calling SimulateTrueCounts (evf_type='discrete', e.g. a Phyla5 tree) and converting the result with True2ObservedCounts using protocol='UMI' over the Supplementary Table 6 grid of Sigma, alpha, depth, and rare-population proportion (Prop) settings. Log-transform the observed counts (e.g. log2(counts+1)), embed cells with PCA, and run k-means in the PCA latent space with the number of clusters matching the true populations (e.g. 5 for the Phyla5 tree). Compute the adjusted Rand index (cal_ARI) between predicted and true labels for every method and grid combination, and assemble the ARI table. Optionally fit a multiple linear regression of ARI on the simulation parameters (N, Sigma, alpha, Depth, Prop) to extract standardized coefficients and reproduce the Fig. 5a bar plot and Fig. 5b-c curves.

## Related tools

- **SymSim** (Simulates true transcript counts (SimulateTrueCounts) and observed UMI counts (True2ObservedCounts) over the sigma/alpha/Prop parameter grid for the benchmark) — https://github.com/YosefLab/SymSim
- **PCA** (Dimensionality reduction producing the latent space in which k-means is run; one of three reduction methods evaluated alongside scVI and SIMLR)
- **SIMLR** (Alternative dimensionality reduction and clustering method used as comparison baseline)
- **scVI** (Deep-learning latent embedding whose k-means clustering is the top-performing comparator in most settings)
- **Seurat** (Louvain graph-based clustering baseline compared against PCA+K-means ARI)

## Examples

```
pc <- prcomp(t(log2(observed_counts+1)))$x[,1:10]; km <- kmeans(pc, centers=5); ari <- cal_ARI(km$cluster, true_labels)
```

## Evaluation signals

- ARI results table contains one row per grid combination (N, Sigma, alpha, Depth, Prop) and clustering method
- ARI increases with number of cells N and decreases as the rare-population proportion shrinks (e.g. 10% vs 5%), matching Fig. 5 trends
- PCA/k-means ARI is competitive with Seurat but generally below scVI+k-means in most simulation settings
- k-means is initialized with the number of clusters equal to the true number of populations (e.g. 5 for Phyla5), so no cluster-number selection error is confounded into ARI
- Qualitative reproduction of Fig. 5a standardized regression coefficients when ARI is regressed on simulation parameters

## Limitations

- The simulation pipeline omits reverse transcription and library cleanup steps, so observed-count realism is bounded
- PCA-based k-means is a linear embedding; scVI followed by k-means achieved the highest ARI in most settings, so PCA+K-means is generally not the top-performing choice
- Performance depends on the rare-population proportion: with 5% rare cells Seurat is second best, and with 10% PCA/k-means performs slightly worse than Seurat

## Evidence

- [discussion] Dimensionality reduction applied before clustering: "We apply three different dimensionality reduction methods to cluster cells simulated from multiple discrete populations: PCA, scVI, and SIMLR."
- [methods] Simulation of observed UMI counts: "True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts"
- [methods] True count generation from a population tree: "SimulateTrueCounts generates true transcript counts for the given number of genes and cells"
- [methods] Multiple discrete populations require a tree: "When there are multiple populations, users need to provide a tree."
- [readme] SymSim installation: "devtools::install_github("YosefLab/SymSim")"
