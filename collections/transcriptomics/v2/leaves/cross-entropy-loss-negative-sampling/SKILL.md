---
name: cross-entropy-loss-negative-sampling
description: Use when a similarity matrix (e.g. EDGE's ensemble similarity matrix
  S) has been built and the initial spectral embedding must be refined so rare and
  common cell groups separate in the low-dimensional plot.
when_to_use_negative:
- The similarity matrix or k-NNG graph has not yet been constructed — the embedding
  optimization stage requires the weighted adjacency matrix S from the weak-learner
  ensemble.
- The goal is feature gene identification rather than embedding refinement; EDGE's
  gene-selection cutoff (mu - 1.5 x sigma of averaged entropy values) is a separate
  stage.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EDGE
  role: Implements the two-stage SGD embedding optimization with cross-entropy loss
    and uniform negative sampling via the endr() function
  repo: https://github.com/shawnstat/EDGE
- name: RSpectra
  role: Performs large-scale eigenvalue decomposition of the normalized Laplacian
    to obtain the initial spectral embedding that SGD refines
- name: Rcpp
  role: C++ dependency (>= 1.0.1) required to build the EDGE package
  repo: https://github.com/shawnstat/EDGE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:4abceab57c00d7e6692210894446fd00d0bf266e85d2a34287fa2af51ba8ae43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shawnstat/EDGE
  tools:
  - EDGE
  - R
  - RSpectra
  - Rcpp
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cross-entropy-loss-negative-sampling

## Summary

Optimize the low-dimensional cell embedding produced by EDGE's spectral embedding stage by minimizing a cross-entropy loss via two-stage stochastic gradient descent with decreasing step size, drawing negative samples randomly from a uniform distribution.

## When to use

Use when a similarity matrix (e.g. EDGE's ensemble similarity matrix S) has been built and the initial spectral embedding must be refined so rare and common cell groups separate in the low-dimensional plot.

## When NOT to use

- The similarity matrix or k-NNG graph has not yet been constructed — the embedding optimization stage requires the weighted adjacency matrix S from the weak-learner ensemble.
- The goal is feature gene identification rather than embedding refinement; EDGE's gene-selection cutoff (mu - 1.5 x sigma of averaged entropy values) is a separate stage.

## Inputs

- Log-normalized, log2-transformed expression matrix dat_log (e.g. 500 cells x 200 Splatter-simulated genes)
- Ensemble similarity matrix S produced by averaging the L weak-learner similarity matrices
- Custom parameter set custom_defs (n_neigs = 15, opt = TRUE, seed = 2020)

## Outputs

- Optimized 2D embedding matrix (simu.endr) with one row per cell
- edge_embedding_2d.csv embedding file

## How to apply

After constructing the k-nearest-neighbor graph with weighted adjacency matrix S (n_neigs = 15) and computing the top d = 2 eigenvectors of the normalized Laplacian D^(1/2)(D - S)D^(1/2) via RSpectra large-scale eigenvalue decomposition, use the resulting spectral embedding as the initialization. Enable optimization with custom_defs$opt = TRUE and set custom_defs$seed = 2020 so the two-stage stochastic gradient descent is reproducible. The optimizer minimizes a cross-entropy loss between observed and random similarity probabilities, where negative samples are randomly selected using a uniform distribution, and the step size decreases over iterations. Write the optimized embedding matrix (e.g. simu.endr) to edge_embedding_2d.csv with one row per cell. Validate that the optimized 2D embedding yields neat visual separation between the rare (2%, 15 cells) and common cell groups when colored by the group labels.

## Related tools

- **EDGE** (Implements the two-stage SGD embedding optimization with cross-entropy loss and uniform negative sampling via the endr() function) — https://github.com/shawnstat/EDGE
- **RSpectra** (Performs large-scale eigenvalue decomposition of the normalized Laplacian to obtain the initial spectral embedding that SGD refines)
- **Rcpp** (C++ dependency (>= 1.0.1) required to build the EDGE package) — https://github.com/shawnstat/EDGE

## Examples

```
custom_defs$opt <- TRUE; custom_defs$seed <- 2020; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- Optimized 2D embedding yields a neat visual separation between the rare (2%, 15 cells) and common cell groups when plotted with group labels
- Embedding has exactly d = 2 columns, one row per cell (500 rows for the Splatter simulation with BC = 500)
- custom_defs$opt = TRUE and custom_defs$seed = 2020 were set so re-running produces a reproducible embedding
- Output file edge_embedding_2d.csv is written with one row per cell
- EDGE embedding appears more compactly projected than t-SNE and UMAP layouts computed on the same dat_log matrix

## Limitations

- The number of negative samples used in the embedding optimization is not specified in the main text
- Hyperparameters a and b of the loss function are not reported; only a reference to nonlinear least squares estimation is given
- Details of the algorithm are deferred to Supplementary Algorithms 1 and 2 rather than given in the main text

## Evidence

- [discussion] a stochastic gradient descent algorithm with the decreasing step size is implemented to minimize the loss function: "a stochastic gradient descent algorithm with the decreasing step size is implemented to minimize the loss function"
- [discussion] For sufficiently large samples, the negative samples are randomly selected using a uniform distribution: "For sufficiently large samples, the negative samples are randomly selected using a uniform distribution"
- [discussion] we use the RSpectra package to perform the large-scale eigenvalue decomposition: "we use the RSpectra package to perform the large-scale eigenvalue decomposition"
- [readme] custom_defs$opt <- TRUE: "# low-dimensional embedding optimization
    custom_defs$opt <- TRUE"
- [discussion] When there exist rare cell types, EDGE provides a neat separation between rare and common cell types: "When there exist rare cell types, EDGE provides a neat separation between rare and common cell types"
