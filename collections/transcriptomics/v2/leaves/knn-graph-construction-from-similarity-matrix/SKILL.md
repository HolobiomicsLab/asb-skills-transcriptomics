---
name: knn-graph-construction-from-similarity-matrix
description: Use when a precomputed cell-by-cell similarity matrix (e.g., EDGE's averaged
  weak-learner matrix S) must be converted into a k-NNG before spectral embedding
  or graph-based dimensionality reduction.
when_to_use_negative:
- Input is raw or unnormalized counts rather than the log2-transformed normalized
  expression matrix and ensemble similarity matrix — EDGE's similarity learning and
  graph construction assume normalized data
- The goal is embedding directly from raw expression profiles without a cell-to-cell
  similarity structure (e.g., running UMAP or t-SNE from scratch on dat_log instead
  of graphing S)
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EDGE
  role: Produces the ensemble similarity matrix S via weak learners; the endr() call
    performs k-NNG construction and spectral embedding internally (n_neigs parameter)
  repo: github.com/shawnstat/EDGE
- name: RSpectra
  role: Performs the large-scale eigenvalue decomposition of the normalized Laplacian
    to obtain the top d eigenvectors (initial spectral embedding)
- name: Rcpp
  role: C++ dependency (>= 1.0.1) required to build the EDGE package
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:b8204ef552adf09c4413a7bd06fdf821ffca76289cc58e049a95a2fb83ba76d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - R
  - RSpectra
  - Rcpp
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# kNN graph construction from similarity matrix

## Summary

Builds a k-nearest-neighbor graph from EDGE's ensemble similarity matrix S, using the top-n_neigs weighted neighbors per cell, to serve as the substrate for spectral embedding of scRNA-seq data.

## When to use

Use when a precomputed cell-by-cell similarity matrix (e.g., EDGE's averaged weak-learner matrix S) must be converted into a k-NNG before spectral embedding or graph-based dimensionality reduction.

## When NOT to use

- Input is raw or unnormalized counts rather than the log2-transformed normalized expression matrix and ensemble similarity matrix — EDGE's similarity learning and graph construction assume normalized data
- The goal is embedding directly from raw expression profiles without a cell-to-cell similarity structure (e.g., running UMAP or t-SNE from scratch on dat_log instead of graphing S)

## Inputs

- Cell-by-cell similarity matrix S (averaged over weak-learner voters)
- Log-normalized, log2-transformed expression matrix dat_log (cells x genes)
- Number of nearest neighbors (n_neigs, e.g., 15) from endr_defs/custom_defs

## Outputs

- k-nearest-neighbor graph with weighted adjacency matrix S
- Normalized Laplacian D^(1/2)(D - S)D^(1/2)
- Initial spectral embedding: top d = 2 eigenvectors (per-cell 2D coordinates)

## How to apply

Take the final similarity matrix S produced by averaging the L weak-learner similarity matrices and construct a k-nearest-neighbor graph with S as the weighted adjacency matrix. In the EDGE example workflow, the input is the log2-transformed, library-size-normalized matrix dat_log (500 cells x 200 Splatter-simulated genes) and its ensemble similarity S, with n_neigs set to 15 via custom_defs. From the k-NNG, form the normalized Laplacian D^(1/2)(D - S)D^(1/2) and compute the top d = 2 eigenvectors using the RSpectra package's large-scale eigenvalue decomposition to obtain the initial spectral embedding. The rationale is that the sparse k-NNG encodes local similarity structure from the ensemble learners, while the normalized Laplacian eigenvectors provide the initial low-dimensional coordinates that are later refined by stochastic gradient descent.

## Related tools

- **EDGE** (Produces the ensemble similarity matrix S via weak learners; the endr() call performs k-NNG construction and spectral embedding internally (n_neigs parameter)) — github.com/shawnstat/EDGE
- **RSpectra** (Performs the large-scale eigenvalue decomposition of the normalized Laplacian to obtain the top d eigenvectors (initial spectral embedding))
- **Rcpp** (C++ dependency (>= 1.0.1) required to build the EDGE package)

## Examples

```
custom_defs$n_neigs <- 15; custom_defs$opt <- TRUE; custom_defs$seed <- 2020; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- k-NNG connectivity: each cell is connected to its n_neigs = 15 most similar neighbors under the weighted adjacency matrix S
- Eigen-decomposition returns exactly d = 2 eigenvectors of the normalized Laplacian, yielding a per-cell 2D embedding matrix (one row per cell, e.g., 500 rows for the simulated data)
- Embedding optimization with custom_defs$opt = TRUE and seed 2020 must be reproducible across runs
- Downstream check: the optimized 2D embedding visually separates the rare group (2%, 15 cells) from the common group (98%) when colored by labs
- Comparison baseline: EDGE embedding should show more compact projected structure than UMAP (n_neigs = 15) and t-SNE run on the same dat_log

## Limitations

- Memory scales with weak-learner and hash-table settings; the optimal hash table size is 1,017,881 but the README example reduces it to 101,107 to save memory, which may reduce similarity accuracy
- The k-NNG quality depends entirely on the ensemble similarity matrix S; edge cases in weak-learner hashing (modulo collisions in the H-size hash table) propagate into the graph
- Hyperparameter details (e.g., negative sample counts, nonlinear-LS-estimated parameters a and b) are deferred to Supplementary Algorithms 1 and 2 and not stated in the main text

## Evidence

- [discussion] k-NNG with weighted adjacency: "the construction of a k-nearest neighbor (k-NNG) graph with the weighted adjacency matrix S in Equation (1)"
- [discussion] spectral embedding on normalized Laplacian: "the spectral embedding is performed on the normalized Laplacian D1/2(D − S)D1/2"
- [discussion] RSpectra eigenvalue decomposition: "we use the RSpectra package to perform the large-scale eigenvalue decomposition"
- [discussion] similarity matrix averaging: "The final similarity matrix S is calculated by averaging the corresponding similarity scores from all voters"
- [readme] n_neigs and opt settings in README example: "# the number of nearest neighbors
    custom_defs$n_neigs <- 15"
