---
name: single-cell-log-normalization-preprocessing
description: Use when preparing a sparse scRNA-seq count matrix (rows=cells, columns=genes,
  CSV/JLD2 from 10x data) for RMT-based signal/noise separation, where raw log normalization
  would inflate detected signals (e.g., 84 vs 42 signals on ZhengMix) due to non-uniform
  cell vector lengths.
when_to_use_negative:
- Input is already normalized (e.g., Pearson residuals or count-based factor analysis
  outputs) — the paper notes these alternatives underperform relative to corrected
  log normalization and double-normalizing distorts signals
- The goal is feature selection of highly variable genes or zero imputation — the
  paper warns high-variance gene selection can compromise local structure in low-variance
  genes and imputation can bias data and generate false signals
- Data is not sparse single-cell count data — the preprocessing thresholds (200 genes/cell,
  15 cells/gene) are tuned for scRNA-seq sparsity
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scLENS
  role: Provides the preprocess function performing QC (cells <200 genes, genes <15
    cells) and the log+L2 normalization scheme feeding RMT-based signal detection
  repo: https://github.com/Mathbiomed/scLENS
- name: Julia
  role: Implementation environment in which scLENS preprocessing and embedding run
    (Julia 1.6+ recommended)
  repo: https://github.com/Mathbiomed/scLENS
- name: CUDA
  role: Optional GPU acceleration for the preprocessing and similarity-matrix computations;
    falls back to CPU automatically
  repo: https://github.com/Mathbiomed/scLENS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:c2ac679e66c2d145d88e28548dc23393d309a38aa2c9869f7ec60661026b3770
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
  tools:
  - CUDA
  - Julia
  - scLENS
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# single-cell-log-normalization-preprocessing

## Summary

Preprocess scRNA-seq count matrices for dimensionality reduction using log normalization with scaling factor L=1 followed by L2 normalization of cell vector lengths, addressing the signal distortion that conventional log normalization alone introduces before RMT-based signal detection (e.g., in scLENS on ZhengMix PBMC data).

## When to use

Use when preparing a sparse scRNA-seq count matrix (rows=cells, columns=genes, CSV/JLD2 from 10x data) for RMT-based signal/noise separation, where raw log normalization would inflate detected signals (e.g., 84 vs 42 signals on ZhengMix) due to non-uniform cell vector lengths.

## When NOT to use

- Input is already normalized (e.g., Pearson residuals or count-based factor analysis outputs) — the paper notes these alternatives underperform relative to corrected log normalization and double-normalizing distorts signals
- The goal is feature selection of highly variable genes or zero imputation — the paper warns high-variance gene selection can compromise local structure in low-variance genes and imputation can bias data and generate false signals
- Data is not sparse single-cell count data — the preprocessing thresholds (200 genes/cell, 15 cells/gene) are tuned for scRNA-seq sparsity

## Inputs

- scRNA-seq raw count matrix as CSV (cells as rows, genes as columns, first column cell IDs) or 10x gz files (matrix.mtx.gz, features.tsv.gz, barcodes.tsv.gz)
- JLD2 file containing a DataFrame under key "data" with a :cell column of cell IDs
- Optional gene dictionary CSV with columns "gene" and "gene_ID" for renaming

## Outputs

- QC-filtered, log-normalized (L=1), L2-normalized cell-by-gene expression matrix
- Preprocessed matrix ready for RMT-based noise filtering (cell similarity matrix G=XX^T/N) and downstream scLENS embedding

## How to apply

Load the count matrix (CSV with cells as rows and genes as columns, or 10x gz files converted via tenx2jld2 to JLD2), apply QC by filtering cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells (optionally cells with mitochondrial fraction above 5% for ZhengMix). Then apply library-size normalization plus gene scaling using log normalization with scaling factor set to 1, as scLENS does to reduce memory demand (median scaling replaces conventional z-score gene scaling). Critically, follow log normalization with L2 normalization of cell vector lengths, because the paper identifies the primary cause of signal distortion during log normalization as non-uniform cell vector lengths; on ZhengMix this drops detected signals from 84 to 42 before the signal robustness test further reduces them to 13. The L2-normalized matrix feeds construction of the cell similarity matrix G=XX^T/N for Marchenko-Pastur/Tracy-Widom (p=0.05) thresholding.

## Related tools

- **scLENS** (Provides the preprocess function performing QC (cells <200 genes, genes <15 cells) and the log+L2 normalization scheme feeding RMT-based signal detection) — https://github.com/Mathbiomed/scLENS
- **Julia** (Implementation environment in which scLENS preprocessing and embedding run (Julia 1.6+ recommended)) — https://github.com/Mathbiomed/scLENS
- **CUDA** (Optional GPU acceleration for the preprocessing and similarity-matrix computations; falls back to CPU automatically) — https://github.com/Mathbiomed/scLENS

## Examples

```
using scLENS; ndf = scLENS.read_file("data/Z8eq.csv.gz"); pre_df = scLENS.preprocess(ndf); sclens_embedding = scLENS.sclens(pre_df, device_="gpu")
```

## Evaluation signals

- Signal count invariant: log-only normalization yields 84 signals on ZhengMix, log+L2 yields 42 — a large drop after adding L2 normalization confirms the distortion was addressed
- QC check: cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are removed before normalization
- Robustness cascade completes: full pipeline with signal robustness test (sparsity >0.97 perturbations, robustness >0.5) reduces signals to 13, and only this embedding separates T-cell subtypes under UMAP (n_neighbors=15, min_dist=0.01, cosine metric)
- Output files written (e.g., zhengmix_signal_counts.csv, pca.csv, umap.csv) contain per-condition counts matching 84/42/13 and valid embedding coordinates

## Limitations

- RMT-based filtering requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources; scLENS on large data with whole genes still requires a computer with high specifications
- Conventional log normalization alone amplifies gaps between zero and non-zero values and does not uniformly normalize genes across expression levels, introducing false variability if L2 normalization is omitted
- UMAP metric changes are not recommended by the README, so the cosine metric should be retained when regenerating embeddings

## Evidence

- [abstract] L2 normalization identified as fix for log-normalization distortion: "we identify the primary cause of signal distortion during log normalization and effectively address it by uniformizing cell vector lengths with L2 normalization"
- [discussion] Scaling factor of 1 during log normalization: "we set a scaling factor of 1 during the first step of log normalization"
- [discussion] Median scaling replaces z-score to reduce memory demand: "we considerably reduced the memory demand of scLENS by replacing conventional gene scaling (z-score scaling) with median scaling"
- [readme] README QC thresholds in preprocess function: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality"
- [readme] README CSV input format specification: "The file should have rows representing cells and columns representing genes. The first row must contain gene names or IDs, and the first column must contain cell IDs."
