---
name: modulo-hashing-similarity-estimation
description: Use when estimating cell-cell similarity for scRNA-seq dimensionality
  reduction (e.g. EDGE) without full pairwise distance computation, by hashing binary
  gene-expression sketches from many weak learners.
when_to_use_negative:
- Input is raw counts or unnormalized expression — sketch thresholds require log-normalized
  values (e.g. log2 of counts scaled to the median library size).
- Exact pairwise distances are required rather than hash-collision-based similarities,
  or the dataset's memory budget cannot accommodate the chosen hash table size H.
- The goal is direct embedding/visualization (t-SNE/UMAP) rather than building a similarity
  matrix for downstream k-NNG and spectral embedding.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EDGE
  role: Provides the affinity() function and endr_defs configuration implementing
    the ensemble weak-learner sketching and modulo-hashing similarity search
  repo: https://github.com/shawnstat/EDGE
- name: Rcpp
  role: C++ dependency (>= 1.0.1, C++11) used to build/run EDGE's compiled hashing
    routines
  repo: https://github.com/shawnstat/EDGE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:6f805d196262708ce908549dbc3130e1d7754ff2bfbc8a5b3d46c25e29fafed0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shawnstat/EDGE
  tools:
  - EDGE
  - Rcpp
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# modulo-hashing-similarity-estimation

## Summary

Ensemble weak-learner similarity search from the EDGE method: each of many weak learners sketches cells into binary bit vectors, applies modulo hashing to map sketch dot-products to hash codes, and cell pairs sharing a hash code receive similarity score 1; final similarities are averaged across all learners.

## When to use

Use when estimating cell-cell similarity for scRNA-seq dimensionality reduction (e.g. EDGE) without full pairwise distance computation, by hashing binary gene-expression sketches from many weak learners.

## When NOT to use

- Input is raw counts or unnormalized expression — sketch thresholds require log-normalized values (e.g. log2 of counts scaled to the median library size).
- Exact pairwise distances are required rather than hash-collision-based similarities, or the dataset's memory budget cannot accommodate the chosen hash table size H.
- The goal is direct embedding/visualization (t-SNE/UMAP) rather than building a similarity matrix for downstream k-NNG and spectral embedding.

## Inputs

- log-normalized scRNA-seq expression matrix (cells x genes, e.g. dat_log, 500 cells x 200 genes, log2 count-per-median normalized)
- configuration of similarity-search parameters (n_wl, n_dm, nk, H, seed), e.g. endr_defs from the EDGE package

## Outputs

- n-by-n cell-cell similarity matrix S with entries bounded in [0,1] (averaged weak-learner voter scores), e.g. similarity_matrix.rds
- per-weak-learner hash-based pairwise similarity scores (intermediate)

## How to apply

Load a log-normalized scRNA-seq expression matrix (e.g. dat_log, cells x genes) and set similarity-search parameters: n_wl weak learners (e.g. 1000), n_dm genes per learner (e.g. 10, heuristic B < 0.05*G for G genes), hash table size H (e.g. 101107; optimal ~1,017,881 given memory), and a random seed. For each weak learner, randomly select B genes and a gene-specific threshold, compute a binary sketch vector V in {0,1}^B per cell by thresholding expression, use modulo hashing to map V·W to one of the predefined hash codes, and assign similarity score 1 to cell pairs sharing a hash code. Average the similarity scores across all weak learners (voters) to obtain the final n-by-n similarity matrix S (Equation 1), then save it (e.g. similarity_matrix.rds). This ensemble sketching trades exact distance computation for fast, accurate similarity search that underlies EDGE's subsequent k-NNG construction and spectral embedding.

## Related tools

- **EDGE** (Provides the affinity() function and endr_defs configuration implementing the ensemble weak-learner sketching and modulo-hashing similarity search) — https://github.com/shawnstat/EDGE
- **Rcpp** (C++ dependency (>= 1.0.1, C++11) used to build/run EDGE's compiled hashing routines) — https://github.com/shawnstat/EDGE

## Examples

```
custom_defs <- endr_defs; custom_defs$n_wl <- 1000; custom_defs$n_dm <- 10; custom_defs$H <- 101107; custom_defs$seed <- 2020; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- Saved similarity object is an n-by-n matrix matching the input cell count (e.g. 500 x 500).
- All entries of the similarity matrix S lie in [0, 1], reflecting averaged voter scores.
- Parameters used match the heuristic: n_dm (B) < 0.05 * number of genes (e.g. 10 genes for 200 genes).
- Setting custom_defs$seed to a fixed value (e.g. 2020) makes the similarity search reproducible across runs.
- Increasing n_wl (weak learners) or hash table size H should not degrade similarity estimates; the README notes more learners and larger H give better results.

## Limitations

- More weak learners and larger hash tables improve results but require large memory; the optimal hash table size is 1,017,881, so a reduced size (101,107) is often used with some loss in fidelity.
- The exact per-dataset value of B is not stated in the paper, only the heuristic that B is typically < 0.05*G.
- Hash-collision-based similarity is stochastic; results depend on the random seed and the number of voters averaged.

## Evidence

- [discussion] ensemble sketching averaging: "For each cell, we calculate a sketch vector V ∈ {0, 1}B, which is a bit vector with B elements"
- [discussion] modulo hashing: "We use modulo hashing technique to map V ⋅ W to one of the predefined hash codes"
- [discussion] ensemble averaging: "The final similarity matrix S is calculated by averaging the corresponding similarity scores from all voters"
- [readme] README parameters: "The optimal hash table size is 1,017,881. We reduce the hash table size to 101,107 in the example. Most of the time, this works well and uses less memory."
- [methods] affinity call: "`affinity(dat, n_wl, n_dm, nk, H, seed)`"
