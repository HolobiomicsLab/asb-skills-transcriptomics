---
name: spectral-embedding-of-normalized-laplacian
description: Use when a cell-cell similarity matrix is available and a low-dimensional
  (e.g., 2D) cell embedding must be initialized before SGD-based refinement, as in
  EDGE on scRNA-seq data.
when_to_use_negative:
- Input is raw expression counts without the ensemble similarity matrix S — the k-NNG
  requires weighted adjacency scores from the weak-learner stage
- The analysis goal is feature gene selection alone; EDGE performs embedding and feature
  gene extraction jointly, but this skill covers only the embedding stage
- Data dimensionality or graph size makes eigen-decomposition impractical and a purely
  direct embedding method (t-SNE, UMAP) is sufficient
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: EDGE
  role: Implements similarity-matrix-to-spectral-embedding pipeline with subsequent
    SGD optimization via endr(dat_log, custom_defs)
  repo: github.com/shawnstat/EDGE
- name: RSpectra
  role: Performs the large-scale eigenvalue decomposition of the normalized Laplacian
    to obtain the top d eigenvectors
- name: Rcpp
  role: C++ dependency (>= 1.0.1) for EDGE's embedding optimization implementation
  repo: github.com/shawnstat/EDGE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:fee43044b70ffd14d01fdbcc9a4b61ed6c24ac36504f388679d6a1d912bc41f8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - R
  - RSpectra
  - Rcpp
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# spectral-embedding-of-normalized-laplacian

## Summary

Initializes the low-dimensional cell embedding in EDGE by building a k-nearest-neighbor graph from the ensemble similarity matrix S, forming the normalized Laplacian D^(1/2)(D − S)D^(1/2), and computing its top d eigenvectors, which are then refined by SGD optimization.

## When to use

Use when a cell-cell similarity matrix is available and a low-dimensional (e.g., 2D) cell embedding must be initialized before SGD-based refinement, as in EDGE on scRNA-seq data.

## When NOT to use

- Input is raw expression counts without the ensemble similarity matrix S — the k-NNG requires weighted adjacency scores from the weak-learner stage
- The analysis goal is feature gene selection alone; EDGE performs embedding and feature gene extraction jointly, but this skill covers only the embedding stage
- Data dimensionality or graph size makes eigen-decomposition impractical and a purely direct embedding method (t-SNE, UMAP) is sufficient

## Inputs

- Ensemble similarity matrix S (averaged weak-learner similarity scores, cells x cells)
- Log-normalized, log2-transformed expression matrix (e.g., dat_log, 500 cells x 200 genes)
- Parameter list (custom_defs) with n_neigs = 15, opt = TRUE, seed = 2020

## Outputs

- Initial spectral embedding matrix (top d eigenvectors, one row per cell)
- Optimized 2D embedding matrix (e.g., simu.endr) for visualization and downstream SNN clustering

## How to apply

After the ensemble of weak learners produces the averaged similarity matrix S, construct a k-nearest-neighbor graph with the weighted adjacency matrix S (EDGE README uses n_neigs = 15). Form the normalized Laplacian D^(1/2)(D − S)D^(1/2) and compute the top d eigenvectors (d = 2 for a 2D embedding) using RSpectra's large-scale eigenvalue decomposition. This spectral embedding is only the initial estimate: EDGE then refines it with a two-stage stochastic gradient descent (custom_defs$opt = TRUE, seed = 2020) that minimizes a cross-entropy loss with randomly, uniformly selected negative samples and a decreasing step size, yielding the final optimized embedding matrix (e.g., simu.endr).

## Related tools

- **EDGE** (Implements similarity-matrix-to-spectral-embedding pipeline with subsequent SGD optimization via endr(dat_log, custom_defs)) — github.com/shawnstat/EDGE
- **RSpectra** (Performs the large-scale eigenvalue decomposition of the normalized Laplacian to obtain the top d eigenvectors)
- **Rcpp** (C++ dependency (>= 1.0.1) for EDGE's embedding optimization implementation) — github.com/shawnstat/EDGE

## Examples

```
custom_defs <- endr_defs; custom_defs$n_neigs <- 15; custom_defs$opt <- TRUE; custom_defs$seed <- 2020; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- The k-NNG is built with n_neigs = 15 nearest neighbors on the weighted adjacency matrix S
- Eigen-decomposition of D^(1/2)(D − S)D^(1/2) returns exactly d = 2 eigenvectors for the 2D embedding
- The optimized embedding (simu.endr) has one row per cell (500 rows for the Splatter simulation)
- Plotting the embedding with group labels shows a neat separation between the rare (2%, 15 cells) and common (98%) cell groups
- Setting custom_defs$opt = TRUE with seed 2020 makes the SGD stage reproducible across runs

## Limitations

- The number of negative samples in the SGD embedding optimization is not specified in the main text
- Hyperparameters a and b for the embedding are not reported; only nonlinear least squares estimation is referenced
- Detailed algorithms are deferred to Supplementary Algorithms 1 and 2 rather than the main text
- Larger numbers of weak learners and hash table sizes improve results but require large memory; the README reduces hash table size from 1,017,881 to 101,107

## Evidence

- [discussion] Spectral embedding via top d eigenvectors of the normalized Laplacian: "the spectral embedding is performed on the normalized Laplacian D1/2(D − S)D1/2"
- [discussion] Embedding optimization via two-stage SGD with decreasing step size: "a stochastic gradient descent algorithm with the decreasing step size is implemented to minimize the loss function"
- [discussion] RSpectra used for large-scale eigenvalue decomposition: "we use the RSpectra package to perform the large-scale eigenvalue decomposition"
- [discussion] k-NNG construction on weighted adjacency matrix S: "the construction of a k-nearest neighbor (k-NNG) graph with the weighted adjacency matrix S in Equation (1)"
- [readme] README parameters for the embedding example: "custom_defs$n_neigs <- 15"
