---
name: scrna-seq-dimensionality-reduction
description: Use when you have a log2-transformed scRNA-seq matrix (cells x genes)
  and need a 2D embedding that reveals rare cell populations more cleanly than t-SNE
  or UMAP.
when_to_use_negative:
- When the input is not an expression matrix but an already-computed feature table
  or embedding — EDGE expects log-normalized gene expression as input
- When the analysis goal is only global/hierarchical structure recovery and no rare
  cell types or feature genes are of interest — t-SNE preserves local structure but
  can lose global geometry, and EDGE's advantage is specifically rare-type separation
- When memory is severely constrained and very large hash table sizes or many weak
  learners would be required — the README warns 'large memory will be required'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: EDGE
  role: 'Primary method: ensemble similarity learning plus spectral embedding and
    SGD-based embedding optimization via endr(dat_log, custom_defs)'
  repo: https://github.com/shawnstat/EDGE
- name: RSpectra
  role: Large-scale eigenvalue decomposition to compute the top d=2 eigenvectors of
    the normalized Laplacian for the initial spectral embedding
- name: Rcpp
  role: C++ backend dependency (>= 1.0.1) required to install and run EDGE
- name: Splatter
  role: Simulates the scRNA-seq test data (500 cells, 200 genes, two groups with minor
    group of 15 cells) via splatSimulate(method='groups')
- name: UMAP
  role: Baseline embedding method run with random_state=2020 and 15 nearest neighbors
    for side-by-side comparison
- name: t-SNE (Rtsne)
  role: Baseline embedding method run with set.seed(2020) via Rtsne(dat_log) for side-by-side
    comparison
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:4df14eb80f486257bf2ce320f2bed3c4ffe08c6610eda6dd559efaff488aac52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shawnstat/EDGE
  tools:
  - EDGE
  - R
  - RSpectra
  - Rcpp
  - Splatter
  - UMAP
  - t-SNE (Rtsne)
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# scrna-seq-dimensionality-reduction

## Summary

Two-dimensional embedding of log-normalized scRNA-seq expression matrices via EDGE, which estimates an initial spectral embedding from a similarity matrix built by weak learners and refines it with stochastic gradient descent to separate rare cell types while enabling feature gene extraction.

## When to use

Use when you have a log2-transformed scRNA-seq matrix (cells x genes) and need a 2D embedding that reveals rare cell populations more cleanly than t-SNE or UMAP.

## When NOT to use

- When the input is not an expression matrix but an already-computed feature table or embedding — EDGE expects log-normalized gene expression as input
- When the analysis goal is only global/hierarchical structure recovery and no rare cell types or feature genes are of interest — t-SNE preserves local structure but can lose global geometry, and EDGE's advantage is specifically rare-type separation
- When memory is severely constrained and very large hash table sizes or many weak learners would be required — the README warns 'large memory will be required'

## Inputs

- Raw scRNA-seq count matrix (e.g., Splatter-simulated 500 cells x 200 genes, seed 2020)
- Log-normalized expression matrix dat_log produced by sweep(counts_dt,2,rs/rs_med,'/') followed by t(log2(dat_norm+1))
- Custom endr_defs parameter list (n_wl, n_dm, n_neigs, opt, seed, H)

## Outputs

- Optimized 2D embedding matrix (simu.endr), one row per cell, suitable for export to edge_embedding_2d.csv
- Feature gene list selected by the ensemble weak learners (cutoff at mu - 1.5 x sigma of averaged entropy values)
- Three-panel scatter plot comparing EDGE, UMAP, and t-SNE embeddings colored by cell labels

## How to apply

Normalize raw counts by scaling each cell's library size to the median total count and log2-transform via log2(norm+1) to obtain dat_log. Configure endr_defs with n_wl=1000 weak learners, n_dm=10 genes per learner, n_neigs=15 nearest neighbors, opt=TRUE, seed=2020, and hash table size H=101107 (optimal is 1,017,881 but 101,107 usually suffices and uses less memory). Run endr(dat_log, custom_defs): EDGE builds a k-NNG from the ensemble similarity matrix S, computes the top d=2 eigenvectors of the normalized Laplacian D^(1/2)(D-S)D^(1/2) via RSpectra, then minimizes a cross-entropy loss with uniformly drawn negative samples by two-stage SGD with a decreasing step size. Compare against umap(dat_log, random_state=2020) with 15 neighbors and Rtsne(dat_log) with set.seed(2020), plotting all three embeddings colored by cell labels.

## Related tools

- **EDGE** (Primary method: ensemble similarity learning plus spectral embedding and SGD-based embedding optimization via endr(dat_log, custom_defs)) — https://github.com/shawnstat/EDGE
- **RSpectra** (Large-scale eigenvalue decomposition to compute the top d=2 eigenvectors of the normalized Laplacian for the initial spectral embedding)
- **Rcpp** (C++ backend dependency (>= 1.0.1) required to install and run EDGE)
- **Splatter** (Simulates the scRNA-seq test data (500 cells, 200 genes, two groups with minor group of 15 cells) via splatSimulate(method='groups'))
- **UMAP** (Baseline embedding method run with random_state=2020 and 15 nearest neighbors for side-by-side comparison)
- **t-SNE (Rtsne)** (Baseline embedding method run with set.seed(2020) via Rtsne(dat_log) for side-by-side comparison)

## Examples

```
custom_defs <- endr_defs; custom_defs$n_wl <- 1000; custom_defs$n_dm <- 10; custom_defs$n_neigs <- 15; custom_defs$opt <- TRUE; custom_defs$seed <- 2020; custom_defs$H <- 101107; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- The optimized 2D embedding shows a neat visual separation between the rare group (2%, 15 of 500 Splatter-simulated cells) and the common group when plotted with group labels
- On PBMC over 100+ replications, the mean silhouette index of the four rare cell types reaches ~0.851 (SD 0.045), exceeding UMAP (~0.801) and t-SNE (~0.838)
- The endr call completes with the specified parameter set (n_wl=1000, n_dm=10, n_neigs=15, opt=TRUE, seed=2020, H=101107) and returns a 2-column embedding with one row per cell
- The three-panel plot (EDGE, UMAP, t-SNE) is produced with matching colors from labs and identical data input dat_log
- EDGE's projection appears more compact than t-SNE and UMAP on real datasets, as claimed in the discussion

## Limitations

- The more weak learners and larger hash table sizes used, the better the result, but large memory is required; the optimal hash table size is 1,017,881, and the reduced size 101,107 'works well most of the time' but is a trade-off
- The number of negative samples used in the embedding optimization is not specified in the paper
- The exact per-dataset number of genes per weak learner B is not stated, only a heuristic (values for B are typically < 0.05*G)
- Detailed algorithms are deferred to Supplementary Algorithms 1 and 2 rather than the main text

## Evidence

- [abstract] Spectral embedding and SGD optimization: "the low-dimensional embedding of the data is estimated and optimized through spectral embedding and stochastic gradient descent"
- [discussion] Normalized Laplacian spectral embedding: "the spectral embedding is performed on the normalized Laplacian D1/2(D − S)D1/2"
- [discussion] SGD with decreasing step size: "a stochastic gradient descent algorithm with the decreasing step size is implemented to minimize the loss function"
- [discussion] RSpectra for eigenvalue decomposition: "we use the RSpectra package to perform the large-scale eigenvalue decomposition"
- [readme] README invocation and parameters: "custom_defs$n_wl <- 1000
    # the number of genes selected to construct weak learners
    custom_defs$n_dm <- 10
    # the number of nearest neighbors
    custom_defs$n_neigs <- 15"
- [readme] Hash table size trade-off: "The optimal hash table size is 1,017,881. We reduce the hash table size to 101,107 in the example. Most of the time, this works well and uses less memory."
