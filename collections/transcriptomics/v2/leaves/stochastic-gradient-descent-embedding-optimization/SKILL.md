---
name: stochastic-gradient-descent-embedding-optimization
description: Use when a cell-cell similarity matrix (e.g., ensemble weak-learner similarity
  S from EDGE) must be converted into an optimized low-dimensional embedding for visualization
  or downstream clustering of scRNA-seq data.
when_to_use_negative:
- Input is raw counts or a similarity matrix has not yet been computed — the optimization
  stage requires the ensemble similarity matrix S from the weak-learner stage
- Goal is to identify feature/differentially expressed genes — that is EDGE's separate
  feature gene extraction output, not the embedding optimization stage
tools:
- name: EDGE
  role: Runs the endr() pipeline that performs spectral embedding initialization and
    SGD-based embedding optimization
  repo: https://github.com/shawnstat/EDGE
- name: RSpectra
  role: Performs the large-scale eigenvalue decomposition of the normalized Laplacian
    to obtain the initial spectral embedding
- name: Rcpp
  role: C++ backend (>= 1.0.1, C++11) required to install and run EDGE's similarity
    and optimization routines
  repo: https://github.com/shawnstat/EDGE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:981645318197f04d41935c30522581aae7caf65ac5c3216daef7e60c63718b84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shawnstat/EDGE
  tools:
  - EDGE
  - R
  - RSpectra
  - Rcpp
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# stochastic-gradient-descent-embedding-optimization

## Summary

EDGE estimates the initial low-dimensional embedding of cells via spectral embedding of the normalized Laplacian built from the ensemble similarity matrix, then optimizes that embedding by minimizing a cross-entropy loss with a two-stage stochastic gradient descent (SGD) using a decreasing step size and uniformly sampled negative examples.

## When to use

Use when a cell-cell similarity matrix (e.g., ensemble weak-learner similarity S from EDGE) must be converted into an optimized low-dimensional embedding for visualization or downstream clustering of scRNA-seq data.

## When NOT to use

- Input is raw counts or a similarity matrix has not yet been computed — the optimization stage requires the ensemble similarity matrix S from the weak-learner stage
- Goal is to identify feature/differentially expressed genes — that is EDGE's separate feature gene extraction output, not the embedding optimization stage

## Inputs

- Log-normalized, log2-transformed expression matrix dat_log (e.g., 500 cells x 200 Splatter-simulated genes)
- Ensemble cell-cell similarity matrix S averaged across weak learners
- EDGE parameter list custom_defs (n_neigs=15, opt=TRUE, seed=2020)

## Outputs

- Optimized 2D embedding matrix simu.endr (one row per cell), exportable as edge_embedding_2d.csv
- 2D scatter embedding suitable for visualization and clustering

## How to apply

From the log2-transformed, library-size-normalized expression matrix (dat_log) and the ensemble similarity matrix S, build a k-nearest-neighbor graph using n_neigs = 15 neighbors with weighted adjacency matrix S. Compute the normalized Laplacian D^(1/2)(D − S)D^(1/2) and obtain the initial d = 2 dimensional spectral embedding via large-scale eigenvalue decomposition with the RSpectra package. Optimize the embedding by a two-stage stochastic gradient descent with decreasing step size (custom_defs$opt = TRUE, seed 2020), minimizing a cross-entropy loss in which negative samples are randomly selected from a uniform distribution. The optimized 2D embedding (simu.endr) is written out (e.g., edge_embedding_2d.csv) and used for visualization and shared-nearest-neighbor modularity clustering.

## Related tools

- **EDGE** (Runs the endr() pipeline that performs spectral embedding initialization and SGD-based embedding optimization) — https://github.com/shawnstat/EDGE
- **RSpectra** (Performs the large-scale eigenvalue decomposition of the normalized Laplacian to obtain the initial spectral embedding)
- **Rcpp** (C++ backend (>= 1.0.1, C++11) required to install and run EDGE's similarity and optimization routines) — https://github.com/shawnstat/EDGE

## Examples

```
custom_defs <- endr_defs; custom_defs$n_neigs <- 15; custom_defs$opt <- TRUE; custom_defs$seed <- 2020; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- The optimized embedding yields a neat visual separation between the rare (2%, 15 cells) and common cell groups when plotted with group labels
- Embedding dimension is d = 2, with one row per cell in the output matrix
- custom_defs$opt = TRUE and custom_defs$seed = 2020 were set so the two-stage SGD with decreasing step size actually runs and is reproducible
- The k-NNG used n_neigs = 15 neighbors with weighted adjacency matrix S before spectral decomposition
- EDGE's embedding appears more compactly projected than t-SNE and UMAP (both run with seed 2020, UMAP also with 15 neighbors) on the same input

## Limitations

- The number of negative samples used in the embedding optimization is not specified in the article
- Hyperparameters a and b (linking similarity probability to embedding) are not reported; only nonlinear least squares estimation via a reference is given
- Detailed algorithm steps are deferred to Supplementary Algorithms 1 and 2
- Large hash table sizes (optimal 1,017,881) improve weak-learner similarity quality but require large memory; the README reduces H to 101,107

## Evidence

- [discussion] Spectral embedding of normalized Laplacian: "the spectral embedding is performed on the normalized Laplacian D1/2(D − S)D1/2"
- [discussion] SGD with decreasing step size: "a stochastic gradient descent algorithm with the decreasing step size is implemented to minimize the loss function"
- [discussion] Negative sampling strategy: "the negative samples are randomly selected using a uniform distribution"
- [methods] Concrete endr invocation: "simu.endr <- endr(dat_log,custom_defs)"
- [readme] Memory/parameter tradeoff from README: "The optimal hash table size is 1,017,881. We reduce the hash table size to 101,107 in the example."
