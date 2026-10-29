---
name: ensemble-weak-learner-averaging
description: Use when you have a log-normalized scRNA-seq expression matrix and need
  a bounded (0-1) cell-by-cell similarity matrix to seed downstream k-NNG, spectral
  embedding, or dimensionality reduction.
when_to_use_negative:
- You need an embedding or clustering directly — this skill only produces the similarity
  matrix; downstream k-NNG construction, spectral embedding, and SGD optimization
  are separate stages of EDGE
- The expression matrix is not normalized (raw counts) — the weak-learner gene-specific
  thresholds and sketches assume log-normalized input
- The number of genes is far outside the regime where B < 0.05*G yields a workable
  sketch, or memory cannot hold the required hash table
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: EDGE
  role: Provides the affinity() function and endr_defs configuration implementing
    the weak-learner sketching, modulo hashing, and voter averaging
  repo: github.com/shawnstat/EDGE
- name: Rcpp
  role: C++ dependency (>= 1.0.1) required to build EDGE's fast sketching/hashing
    backend
  repo: github.com/shawnstat/EDGE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:fa14401900576dad56a242926dc1108d5af04d23b9720028fe3cf0707b4c9bbc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - Rcpp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ensemble-weak-learner-averaging

## Summary

Builds a cell-to-cell similarity matrix for scRNA-seq by running many weak learners, each sketching a random subset of genes into binary bit vectors and hashing them, then averaging the pairwise similarity scores across all voters (Equation 1 of EDGE).

## When to use

Use when you have a log-normalized scRNA-seq expression matrix and need a bounded (0-1) cell-by-cell similarity matrix to seed downstream k-NNG, spectral embedding, or dimensionality reduction.

## When NOT to use

- You need an embedding or clustering directly — this skill only produces the similarity matrix; downstream k-NNG construction, spectral embedding, and SGD optimization are separate stages of EDGE
- The expression matrix is not normalized (raw counts) — the weak-learner gene-specific thresholds and sketches assume log-normalized input
- The number of genes is far outside the regime where B < 0.05*G yields a workable sketch, or memory cannot hold the required hash table

## Inputs

- log-normalized scRNA-seq expression matrix (cells x genes, e.g. dat_log from log2 of Splatter-normalized counts)
- endr_defs configuration with n_wl=1000 weak learners, n_dm=10 genes per learner, hash table size H=101107, random seed=2020

## Outputs

- n x n similarity matrix S (e.g. 500 x 500) with entries bounded between 0 and 1, representing averaged voter similarity scores (Equation 1), savable as similarity_matrix.rds

## How to apply

Load the log-normalized expression matrix (e.g. dat_log, 500 cells x 200 genes from log2 of Splatter library-size-normalized counts) and the endr_defs configuration from the EDGE package. Run the ensemble similarity search via affinity(dat, n_wl, n_dm, nk, H, seed): each of the n_wl = 1000 weak learners randomly selects n_dm = 10 genes (heuristic B < 0.05*G), computes a binary sketch vector V in {0,1}^B per cell by thresholding gene-specific values, maps V dot W via modulo hashing to a table of size H = 101107, and assigns similarity score 1 to cell pairs sharing a hash code. Average the similarity scores from all 1000 weak learners (voters) to obtain the final similarity matrix S per Equation 1. More weak learners and larger hash tables improve accuracy at the cost of memory; a seed (2020) makes the random gene and threshold draws reproducible.

## Related tools

- **EDGE** (Provides the affinity() function and endr_defs configuration implementing the weak-learner sketching, modulo hashing, and voter averaging) — github.com/shawnstat/EDGE
- **Rcpp** (C++ dependency (>= 1.0.1) required to build EDGE's fast sketching/hashing backend) — github.com/shawnstat/EDGE

## Examples

```
custom_defs <- endr_defs; custom_defs$n_wl <- 1000; custom_defs$n_dm <- 10; custom_defs$H <- 101107; custom_defs$seed <- 2020; S <- affinity(dat_log, custom_defs$n_wl, custom_defs$n_dm, nk, custom_defs$H, custom_defs$seed)
```

## Evaluation signals

- The saved object is a square n-by-n matrix matching the input cell count (500 x 500 for the Splatter example)
- All matrix entries are bounded similarity scores between 0 and 1, as expected from averaged binary voter scores
- Setting custom_defs$seed to a fixed value (2020) reproduces the same similarity matrix across runs
- Increasing n_wl or H should not change the qualitative similarity structure (README states more weak learners and larger hash tables give better results)
- The similarity matrix feeds a k-NNG whose spectral embedding separates known cell groups (e.g. the 15-cell minor group in the 500-cell Splatter simulation)

## Limitations

- Accuracy depends on ensemble size and hash table size: more weak learners and larger tables improve results but large memory is required (optimal table size 1,017,881; the example uses a reduced 101,107)
- The exact per-dataset value of B is not fixed; only the heuristic B < 0.05*G is given
- The similarity search is only the first stage of EDGE — it does not by itself produce an embedding or feature genes

## Evidence

- [discussion] Ensemble averaging across voters: "The final similarity matrix S is calculated by averaging the corresponding similarity scores from all voters"
- [discussion] Sketch and hashing mechanism: "We use modulo hashing technique to map V ⋅ W to one of the predefined hash codes"
- [discussion] Binary sketch construction: "For each cell, we calculate a sketch vector V ∈ {0, 1}B, which is a bit vector with B elements"
- [readme] Hash table size trade-off: "The optimal hash table size is 1,017,881. We reduce the hash table size to 101,107 in the example."
- [abstract] Rationale for ensemble of weak learners: "the proposed method implements an ensemble learning scheme that utilizes massive weak learners for an accurate similarity search"
