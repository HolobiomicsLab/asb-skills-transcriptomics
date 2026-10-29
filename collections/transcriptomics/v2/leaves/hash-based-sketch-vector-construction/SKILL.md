---
name: hash-based-sketch-vector-construction
description: Use when you need cell-to-cell similarity scores from a log-normalized
  scRNA-seq expression matrix without full pairwise distance computation, e.g. building
  the similarity matrix S that feeds EDGE's k-NNG and spectral embedding.
when_to_use_negative:
- The goal is a standalone 2-D embedding or clustering — sketch similarity alone is
  only the front end of EDGE; you still need k-NNG construction, spectral embedding,
  and SGD optimization.
- The input matrix is raw counts — gene-specific thresholds and sketching assume log-normalized
  expression (median-CTP normalized, log2(x+1)).
- Memory for the hash table is unavailable at the desired H — the article notes large
  H (optimal 1,017,881) is better but 'large memory will be required'.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EDGE
  role: R package whose affinity() weak learners build sketch vectors, apply modulo
    hashing, and average voter similarity scores
  repo: https://github.com/shawnstat/EDGE
- name: Rcpp
  role: C++ backend dependency (>= 1.0.1, C++11) for the performance-critical sketching/hashing
    loops in EDGE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:d3b96ae65edce830d2cf1281af127155ddb07f324da33d81d8b81810b0be0f3e
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
yes_rate: 0.5
n_claims: 8
---

# hash-based-sketch-vector-construction

## Summary

Constructs binary sketch vectors V ∈ {0,1}^B per cell by thresholding a randomly selected subset of B genes, then maps the inner product V·W via modulo hashing to a predefined hash table so that cell pairs sharing a hash code receive a pairwise similarity score of 1 — the core weak-learner operation inside EDGE's ensemble similarity search.

## When to use

Use when you need cell-to-cell similarity scores from a log-normalized scRNA-seq expression matrix without full pairwise distance computation, e.g. building the similarity matrix S that feeds EDGE's k-NNG and spectral embedding.

## When NOT to use

- The goal is a standalone 2-D embedding or clustering — sketch similarity alone is only the front end of EDGE; you still need k-NNG construction, spectral embedding, and SGD optimization.
- The input matrix is raw counts — gene-specific thresholds and sketching assume log-normalized expression (median-CTP normalized, log2(x+1)).
- Memory for the hash table is unavailable at the desired H — the article notes large H (optimal 1,017,881) is better but 'large memory will be required'.

## Inputs

- Log-normalized scRNA-seq expression matrix (cells x genes, e.g. dat_log from log2(dat_norm+1))
- endr_defs configuration: n_wl (weak learners, 1000), n_dm (genes per learner, 10), H (hash table size, 101107), seed (2020)

## Outputs

- Averaged cell-by-cell similarity matrix S (n x n, e.g. 500 x 500, entries in [0,1]) per Equation 1
- Per-weak-learner hash-code assignments (implicit; cells sharing a hash code get similarity 1)

## How to apply

Start from a log2-normalized cell-by-gene matrix (e.g. dat_log, 500 cells x 200 genes produced by sweep-based median-CTP normalization in the EDGE README). For each weak learner, randomly select B genes (in the README example n_dm = 10, consistent with the article heuristic B < 0.05*G) and randomly pick a gene-specific threshold; threshold each cell's expression to obtain a bit vector V ∈ {0,1}^B. Compute V·W and apply modulo hashing with a hash table of size H (101,107 in the example; article notes the optimal size is 1,017,881) to map the score to one of the predefined hash codes, assigning similarity score 1 to cell pairs sharing a hash code. Average these binary similarity scores across all L weak learners (n_wl = 1000 voters, seed = 2020) to obtain the final similarity matrix S per Equation 1. Rationale: massive randomized sketches let EDGE learn cell similarities accurately while keeping per-learner memory and compute small, and genes that contribute strongly across voters become feature-gene candidates.

## Related tools

- **EDGE** (R package whose affinity() weak learners build sketch vectors, apply modulo hashing, and average voter similarity scores) — https://github.com/shawnstat/EDGE
- **Rcpp** (C++ backend dependency (>= 1.0.1, C++11) for the performance-critical sketching/hashing loops in EDGE)

## Examples

```
custom_defs <- endr_defs; custom_defs$n_wl <- 1000; custom_defs$n_dm <- 10; custom_defs$H <- 101107; custom_defs$seed <- 2020; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- The saved similarity object is an n-by-n matrix (500 x 500 for the README dataset) with all entries bounded between 0 and 1.
- Each weak learner selects exactly n_dm = 10 genes, satisfying the heuristic B < 0.05*G for G = 200 genes.
- All n_wl = 1000 voter scores are averaged with the fixed seed (2020); rerunning with the same seed reproduces the same similarity matrix.
- Hash codes fall within the table of size H = 101107 (or 1,017,881 if using the optimal size) with no out-of-range mappings.
- Cells from the same simulated group (e.g. the 15-cell minor group with group.prob = c(0.98, 0.02)) share hash codes more often, yielding higher similarity entries.

## Limitations

- Exact per-dataset B is not stated by the article; only a heuristic (B typically < 0.05*G) is given.
- More weak learners and larger hash table sizes improve results but require large memory; the README reduces H to 101,107 from the optimal 1,017,881 to save memory.
- Detailed sketching/hashing pseudocode is deferred to Supplementary Algorithms 1 and 2 rather than the main text.

## Evidence

- [discussion] Compute a binary sketch vector V per cell by thresholding gene expression values: "For each cell, we calculate a sketch vector V ∈ {0, 1}B, which is a bit vector with B elements"
- [discussion] Modulo hashing assigns similarity 1 to hash-sharing cells: "We use modulo hashing technique to map V ⋅ W to one of the predefined hash codes"
- [discussion] Ensemble averaging yields the final similarity matrix: "The final similarity matrix S is calculated by averaging the corresponding similarity scores from all voters"
- [methods] affinity function signature for the similarity search: "`affinity(dat, n_wl, n_dm, nk, H, seed)`"
- [readme] Hash table size trade-off documented in the README: "The optimal hash table size is 1,017,881. We reduce the hash table size to 101,107 in the example."
- [readme] B genes per weak learner in the README configuration: "custom_defs$n_dm <- 10"
