---
name: tsne-embedding-on-similarity-matrix
description: Use when a cell-to-cell similarity matrix (e.g. learned by SIMLR from
  scRNA-seq expression data) must be turned into low-dimensional coordinates for visualizing
  and interpreting subpopulations.
when_to_use_negative:
- The goal is clustering or de novo discovery rather than visualization — SIMLR's
  similarity learning itself (multi-kernel + graph diffusion) is the appropriate stage,
  not the downstream t-SNE embedding
- The input is raw high-throughput data at a scale beyond the standard implementation
  — use SIMLR_Large_Scale (with Rcpp, pracma, RcppAnnoy, RSpectra) instead of the
  standard pipeline
- Example datasets are needed as authoritative data — the README states the provided
  Test_*.RData files are preprocessed examples 'and should not be used in place of
  the ones provided in the respective publications'
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: SIMLR
  role: Learns the sample-to-sample similarity measure via multiple kernels and graph
    diffusion, then performs the downstream t-SNE dimension-reduction and 2-D/3-D
    visualization
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: R
  role: Implementation language for the standard SIMLR pipeline (SIMLR.R) and the
    t-SNE embedding demo
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: grDevices
  role: R package used to color the scatter plots of the resulting embedding
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:27d0e8f919388b99de09fef0ec5da16cee6d180cfdd9bb1b9c81e43aa6e1969f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/BatzoglouLabSU/SIMLR
  tools:
  - Matlab
  - R
  - SIMLR
  - grDevices
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# t-SNE embedding on a learned similarity matrix (SIMLR)

## Summary

Converts a learned cell-to-cell similarity matrix — such as the one produced by SIMLR's multi-kernel optimization — into 2-D (or 3-D) coordinates via stochastic neighbor embedding, enabling visualization and interpretation of single-cell populations.

## When to use

Use when a cell-to-cell similarity matrix (e.g. learned by SIMLR from scRNA-seq expression data) must be turned into low-dimensional coordinates for visualizing and interpreting subpopulations.

## When NOT to use

- The goal is clustering or de novo discovery rather than visualization — SIMLR's similarity learning itself (multi-kernel + graph diffusion) is the appropriate stage, not the downstream t-SNE embedding
- The input is raw high-throughput data at a scale beyond the standard implementation — use SIMLR_Large_Scale (with Rcpp, pracma, RcppAnnoy, RSpectra) instead of the standard pipeline
- Example datasets are needed as authoritative data — the README states the provided Test_*.RData files are preprocessed examples 'and should not be used in place of the ones provided in the respective publications'

## Inputs

- Single-cell RNA-seq expression matrix (e.g. Test_1_mECS.RData, Test_2_Kolod.RData, Test_3_Pollen.RData, Test_4_Usoskin.RData, Test_5_Zeisel.RData from the package data directory)
- Learned cell-to-cell similarity matrix from SIMLR's multi-kernel optimization

## Outputs

- 2-D (and optionally 3-D) embedding coordinates for every cell
- Colored scatter plot of the embedding for interpreting cell populations
- Embedding coordinates saved as simlr_tsne_embedding.csv

## How to apply

Load an example single-cell expression dataset (e.g. Test_1_mECS.RData from the package data directory) and run SIMLR's multi-kernel similarity-learning loop, which combines multiple kernels and applies graph diffusion to strengthen weak similarities caused by noise or dropout. Feed the resulting similarity matrix into a t-distributed stochastic neighbor embedding (t-SNE) step: convert similarities into conditional probabilities and minimize KL divergence to place each cell in a low-dimensional space. Extract the 2-D (and optionally 3-D) coordinates for every cell, then render the embedding as a colored scatter plot (the grDevices package is used to color the plots). Before running the R implementation, compile the external C code with `R CMD SHLIB -c projsplx_R.c`, which is required for the computations. Judgment of correctness: known subpopulations should separate more accurately in the resulting 2-D/3-D visualization than with existing dimension-reduction methods.

## Related tools

- **SIMLR** (Learns the sample-to-sample similarity measure via multiple kernels and graph diffusion, then performs the downstream t-SNE dimension-reduction and 2-D/3-D visualization) — https://github.com/BatzoglouLabSU/SIMLR
- **R** (Implementation language for the standard SIMLR pipeline (SIMLR.R) and the t-SNE embedding demo) — https://github.com/BatzoglouLabSU/SIMLR
- **grDevices** (R package used to color the scatter plots of the resulting embedding)

## Examples

```
R CMD SHLIB -c projsplx_R.c && Rscript R_main_demo_SIMLR.R
```

## Evaluation signals

- The embedding output is a coordinates matrix with one row per cell and 2 (or 3) columns, matching the number of cells in the input expression matrix
- The t-SNE step minimizes KL divergence between conditional probabilities in the high-dimensional similarity space and the low-dimensional embedding
- Known subpopulations in the example dataset (e.g. Test_1_mECS) appear visually separated in the colored scatter plot, consistent with SIMLR's claim of separating known subpopulations more accurately than existing dimension-reduction methods
- simlr_tsne_embedding.csv contains one coordinate record per cell and can be joined back to the input expression matrix by cell identifier
- The projsplx_R.c shared library compiled via R CMD SHLIB loads without error (no stale projsplx_R.o/projsplx_R.so files blocking compilation)

## Limitations

- Single-cell data carry high levels of noise, outliers and dropout events that weaken cell-to-cell similarities; SIMLR mitigates this via graph diffusion, but severe dropout can still degrade the embedding
- SIMLR is reported to produce more reliable clusters than common nonlinear methods such as t-SNE applied directly, so a t-SNE embedding alone is not a substitute for the similarity-learning stage
- The example Test_*.RData datasets are preprocessed examples and should not be used in place of the data from the original publications
- Compilation of projsplx_R.c can fail on some systems (e.g. OS X); stale pre-compiled files (projsplx_R.o, projsplx_R.so) may need to be removed
- Large-scale datasets require the SIMLR_Large_Scale variant and four additional R packages (Rcpp, pracma, RcppAnnoy, RSpectra)

## Evidence
