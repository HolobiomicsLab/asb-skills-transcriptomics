---
name: cell-similarity-network-construction
description: Use when you have multiple low-dimensional latent representation matrices
  of single cells (e.g. from a stacked variational autoencoder with latent dimension
  d = 15) and need to determine cell groups and cluster assignments.
when_to_use_negative:
- Input is raw high-dimensional scRNA-seq expression data — the non-negative kernel
  autoencoder feature-filtering module (top 5000 genes by coefficient variance) and
  VAE dimensionality reduction should run first
- Data is not normalized and min-max scaled to the 0-1 range per row, which is the
  required preprocessing before the autoencoder modules
- The goal is supervised cell-type classification with known labels rather than unsupervised
  clustering from latent representations
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: scCAN
  role: 'Implements the full pipeline: builds cell-similarity networks from VAE latent
    representations, performs SNF-based network fusion, eigengap cluster-number estimation,
    and spectral clustering; available as an R package'
  repo: https://cran.r-project.org/package=scCAN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:5aea0dc0318f7781792ef9de97e29b7e4bd5ac1cca271239c653df5f8270d089
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scCAN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cell-similarity-network-construction

## Summary

Constructs cell-similarity networks from low-dimensional latent representations of scRNA-seq data using local Gaussian kernels (k = 30 neighbors), then fuses multiple networks with the SNF-style iterative update and applies spectral clustering to assign cells to clusters.

## When to use

Use when you have multiple low-dimensional latent representation matrices of single cells (e.g. from a stacked variational autoencoder with latent dimension d = 15) and need to determine cell groups and cluster assignments.

## When NOT to use

- Input is raw high-dimensional scRNA-seq expression data — the non-negative kernel autoencoder feature-filtering module (top 5000 genes by coefficient variance) and VAE dimensionality reduction should run first
- Data is not normalized and min-max scaled to the 0-1 range per row, which is the required preprocessing before the autoencoder modules
- The goal is supervised cell-type classification with known labels rather than unsupervised clustering from latent representations

## Inputs

- Multiple low-dimensional latent representation matrices of cells (from a stacked variational autoencoder, latent dimension d = 15)
- User-controlled hyperparameter i for cluster number search (range 2 to 15)
- Number of neighbors k = 30 for local Gaussian kernels

## Outputs

- Fused cell-similarity network matrix P
- Cluster assignments for each cell (written to cluster_assignments.csv)
- Latent representation matrix (written to latents.csv)

## How to apply

For each latent representation matrix produced by the stacked variational autoencoder (latent dimension d = 15, re-parameterization z = mu + sigma * N(0,1)), build a cell-similarity network using local Gaussian kernels with the number of neighbors kept small at k = 30. Fuse the similarity networks iteratively with the SNF-style update P(v) = S(v) x (sum of other P(k) / (n-1)) x (S(v))^T to obtain a single fused network P. Determine the number of clusters with the eigengap method on the Laplacian (L = D - A), searching the user-controlled hyperparameter i from 2 to 15 and selecting the smallest i among the two highest eigengaps to avoid overestimation. Run spectral clustering on the fused network P to assign cells to clusters; on very large datasets (>5000 cells), split into a 5000-cell training set and a testing set, and use the k-nearest neighbor adaption of spectral clustering (k-nn SC) followed by k-NN classification of the remaining cells. Write the latent representation matrix to latents.csv and the cluster labels to cluster_assignments.csv.

## Related tools

- **scCAN** (Implements the full pipeline: builds cell-similarity networks from VAE latent representations, performs SNF-based network fusion, eigengap cluster-number estimation, and spectral clustering; available as an R package) — https://cran.r-project.org/package=scCAN

## Examples

```
result <- scCAN(data, r.seed = 1); cluster <- result$cluster; ari <- round(scCAN::adjustedRandIndex(cluster, label), 2)
```

## Evaluation signals

- Adjusted Rand index (ARI) between predicted cluster assignments and published cell-type labels (e.g. Pollen dataset) computed via scCAN::adjustedRandIndex
- The eigengap-estimated number of clusters (selected as the smallest i among the two highest eigengaps in the 2-15 search range) matches the known number of cell types in the dataset
- The fused network P is a valid similarity matrix produced by the SNF-style iterative update converging across all per-latent networks
- Output files latents.csv and cluster_assignments.csv are produced with one row per cell and consistent cell ordering

## Limitations

- The eigengap cluster-number search is bounded by a user-controlled hyperparameter i (2 to 15); datasets with more than ~15 expected cell types fall outside the search range
- Local Gaussian kernels use a small neighborhood (k = 30), which may miss structure requiring larger neighborhoods
- For large datasets, cluster assignments for non-training cells rely on k-NN classification from a 5000-cell training subset, introducing approximation error
- scCAN assumes the input is already normalized; the README applies an optional log2(data + 1) transformation only when max(data) > 100

## Evidence

- [methods] For each latent matrix, build a cell-similarity network using local Gaussian kernels with the number of neighbors kept small at k = 30.: "building a cell-similarity network for each of the representations"
- [methods] Fuse the similarity networks iteratively with the SNF-style update P(v) = S(v) x (sum of other P(k) / (n-1)) x (S(v))^T.: "The network fusion approach is adapted from SNF method45–47"
- [methods] Determine the number of clusters with the eigengap method on the Laplacian (L = D - A), searching the user-controlled hyperparameter i from 2 to 15 and selecting the smallest i among the two highest eigengaps to avoid overestimation.: "we use the eigengap method48 to determine the number of clusters"
- [methods] Run spectral clustering on the fused network P to assign cells to clusters.: "we use spectral clustering49 to partition the cells in the fused network P"
- [readme] Calculating adjusted Rand Index: `ari <- round(scCAN::adjustedRandIndex(cluster,label), 2)`: "Calculating adjusted Rand Index: `ari <- round(scCAN::adjustedRandIndex(cluster,label), 2)`"
