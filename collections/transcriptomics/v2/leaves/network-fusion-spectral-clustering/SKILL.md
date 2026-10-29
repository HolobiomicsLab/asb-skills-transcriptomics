---
name: network-fusion-spectral-clustering
description: Use when you already have one or more low-dimensional cell representations
  (e.g. VAE latents) of scRNA-seq data and need cell-type clusters plus an estimate
  of the number of clusters from the fused similarity graph.
when_to_use_negative:
- The input is raw, unnormalized counts — scCAN requires already-normalized data with
  min-max scaling applied before network construction
- You need a single-feature-selection step rather than clustering; feature filtering
  is handled upstream by the non-negative-kernel autoencoder (bottleneck 50, top 5000
  genes), not by this module
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCAN
  role: Implements the local-Gaussian-kernel similarity networks, SNF-style network
    fusion, eigengap cluster-number estimation, and spectral clustering of the fused
    network as its Module 2
  repo: https://cran.r-project.org/package=scCAN
- name: SHARP
  role: Comparison baseline clustering method benchmarked against scCAN, including
    on the Zilionis two-stage rare-cell task
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:96c442641c08727d4cc2cc141bcca5b5453e6dae9a260cddc0034e16b6a4f2b5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SHARP
  - scCAN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Network Fusion Spectral Clustering

## Summary

Graph-based cell-segregation module (Module 2 of scCAN) that builds cell-similarity networks from multiple low-dimensional representations, fuses them SNF-style, and partitions the fused network with spectral clustering to yield cluster assignments.

## When to use

Use when you already have one or more low-dimensional cell representations (e.g. VAE latents) of scRNA-seq data and need cell-type clusters plus an estimate of the number of clusters from the fused similarity graph.

## When NOT to use

- The input is raw, unnormalized counts — scCAN requires already-normalized data with min-max scaling applied before network construction
- You need a single-feature-selection step rather than clustering; feature filtering is handled upstream by the non-negative-kernel autoencoder (bottleneck 50, top 5000 genes), not by this module

## Inputs

- Multiple low-dimensional latent representation matrices of cells (rows = cells) from the stacked VAE, latent dimension d = 15
- Normalized expression matrix with cells as rows and genes as columns (e.g. the 34,558-cell Zilionis matrix or the Pollen dataset), min-max scaled to 0-1

## Outputs

- Fused cell-similarity network P
- Cluster assignment labels per cell (e.g. cluster_assignments.csv)
- Latent representation matrix (e.g. latents.csv)
- Estimated number of clusters from the eigengap method

## How to apply

For each latent representation matrix produced by the stacked variational autoencoder (latent dimension d = 15 in scCAN), build a cell-similarity network using local Gaussian kernels with a small neighbor count k = 30. Fuse the networks iteratively with the SNF-style update P(v) = S(v) x (sum of other P(k) / (n-1)) x S(v)^T. Estimate the number of clusters with the eigengap method on the Laplacian L = D - A, searching the user-controlled hyperparameter i from 2 to 15 and selecting the smallest i among the two highest eigengaps to avoid overestimation. Finally run spectral clustering on the fused network P to assign cells to clusters; the fused P doubles as the latent representation. For datasets with more than 5,000 cells, apply this inside Module 3: cluster a 5,000-cell random training set and classify the remaining cells with k-NN.

## Related tools

- **scCAN** (Implements the local-Gaussian-kernel similarity networks, SNF-style network fusion, eigengap cluster-number estimation, and spectral clustering of the fused network as its Module 2) — https://cran.r-project.org/package=scCAN
- **SHARP** (Comparison baseline clustering method benchmarked against scCAN, including on the Zilionis two-stage rare-cell task)

## Examples

```
library(scCAN); result <- scCAN(data, r.seed = 1); cluster <- result$cluster; ari <- round(scCAN::adjustedRandIndex(cluster, label), 2)
```

## Evaluation signals

- The eigengap-estimated number of clusters should match the known number of cell types (e.g. on Pollen, the estimated k equals the known Pollen cell-type count)
- Adjusted Rand Index (ARI) between predicted cluster assignments and published cell-type labels should be computed and high (README example computes ARI on the SCE dataset)
- On the two-stage Zilionis task (samp.size = 5,000), stage two re-clustering should yield an F1 = 1 for separating the rare tRBC cells
- All similarity matrices must be built with k = 30 neighbors and fused via the SNF update P(v) = S(v) x (sum of other P(k) / (n-1)) x S(v)^T; the fused matrix P must be symmetric and usable directly by spectral clustering
- The eigengap search must scan i = 2 to 15 and return the smallest i among the two highest eigengaps

## Limitations

- The number of clusters search is bounded by the user-controlled hyperparameter range 2 to 15, so cell-type counts outside this range may be missed
- Datasets with n > 5,000 cells cannot be clustered in one pass; the 5,000-cell training/test split with k-NN classification introduces sampling dependence
- No software version number is stated for scCAN in the article, so reproducibility across package versions is not documented

## Evidence

- [methods] For each latent matrix, build a cell-similarity network using local Gaussian kernels with the number of neighbors kept small at k = 30.: [evidence span withheld — anchor status: not_found]
- [methods] Fuse the similarity networks iteratively with the SNF-style update P(v) = S(v) x (sum of other P(k) / (n-1)) x (S(v))^T.: [evidence span withheld — anchor status: not_found]
- [methods] Determine the number of clusters with the eigengap method on the Laplacian (L = D - A), searching the user-controlled hyperparameter i from 2 to 15 and selecting the smallest i among the two highest eigengaps to avoid overestimation.: [evidence span withheld — anchor status: not_found]
- [methods] we use spectral clustering49 to partition the cells in the fused network P: "we use spectral clustering49 to partition the cells in the fused network P"
- [readme] Generate clustering result, the input matrix has rows as samples and columns as genes: `result <- scCAN(data, r.seed = 1)`: "the input matrix has rows as samples and columns as genes: `result <- scCAN(data, r.seed = 1)`"
