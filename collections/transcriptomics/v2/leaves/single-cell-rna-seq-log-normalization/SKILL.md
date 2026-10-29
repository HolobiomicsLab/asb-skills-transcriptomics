---
name: single-cell-rna-seq-log-normalization
description: Use when the input is a raw scRNA-seq count matrix (genes × cells) that
  must be scaled by cell library size and log-transformed before embedding, similarity
  search, or feature gene selection.
when_to_use_negative:
- Input is already log-normalized or is a feature/summary table rather than raw counts
  — re-normalizing would distort expression values.
- The downstream method expects raw counts or applies its own internal size-factor
  normalization.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EDGE
  role: Consumes the log-normalized cells-by-genes matrix as input to endr() for ensemble
    dimensionality reduction and feature gene extraction
  repo: https://github.com/shawnstat/EDGE
- name: splatter
  role: Supplies the raw simulated count matrix (counts(simu_dt)) that is normalized
    with this procedure in the EDGE example workflow
- name: Seurat
  role: Provides FindVariableFeatures used after normalization to select 1000 top
    variable genes for real datasets
- name: R
  role: Implementation language; the normalization is a base-R sweep and log2 operation
    on the count matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:bcb1507731949ff0d4f1d78294e6ef850101ca42b0214ff56577ccc6e22f86ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shawnstat/EDGE
  tools:
  - EDGE
  - R
  - RSpectra
  - Rcpp
  - Seurat
  - splatter
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-rna-seq-log-normalization

## Summary

Library-size normalization to the median total count followed by log2(x+1) transformation of a scRNA-seq count matrix, producing a cells-by-genes log-normalized matrix suitable for downstream methods such as EDGE dimensionality reduction and feature gene extraction.

## When to use

Use when the input is a raw scRNA-seq count matrix (genes × cells) that must be scaled by cell library size and log-transformed before embedding, similarity search, or feature gene selection.

## When NOT to use

- Input is already log-normalized or is a feature/summary table rather than raw counts — re-normalizing would distort expression values.
- The downstream method expects raw counts or applies its own internal size-factor normalization.

## Inputs

- Raw scRNA-seq count matrix (genes × cells), e.g. counts(simu_dt) from Splatter or a real dataset count matrix
- Per-cell library sizes (column sums) computed from the count matrix

## Outputs

- Cells-by-genes log-normalized matrix dat_log = t(log2(dat_norm+1)) with each cell scaled to the median library size

## How to apply

Start from the raw count matrix (e.g., from splatSimulate or a real dataset count matrix). Compute per-cell library sizes as column sums, take the median library size rs_med, and scale each cell by sweep(counts_dt, 2, rs/rs_med, '/') so every cell totals the median count. Then apply the log2(x+1) transform, log2(dat_norm+1), and transpose so the matrix is cells-by-genes (dat_log). This log-normalized matrix is the direct input to the EDGE endr(dat_log, custom_defs) call and to comparison embeddings (UMAP with n_neigs=15, t-SNE with seed 2020). The same normalization was applied to the three real datasets (Jurkat, PBMC, mouse brain) before restricting to the 1000 top variable genes.

## Related tools

- **EDGE** (Consumes the log-normalized cells-by-genes matrix as input to endr() for ensemble dimensionality reduction and feature gene extraction) — https://github.com/shawnstat/EDGE
- **splatter** (Supplies the raw simulated count matrix (counts(simu_dt)) that is normalized with this procedure in the EDGE example workflow)
- **Seurat** (Provides FindVariableFeatures used after normalization to select 1000 top variable genes for real datasets)
- **R** (Implementation language; the normalization is a base-R sweep and log2 operation on the count matrix)

## Examples

```
rs <- colSums(counts_dt); dat_norm <- sweep(counts_dt,2,rs/median(rs),"/"); dat_log <- t(log2(dat_norm+1))
```

## Evaluation signals

- After sweep(), each cell's column sum in dat_norm equals the median library size rs_med (within rounding).
- All values in dat_log are non-negative, since log2(x+1) ≥ 0 for x ≥ 0.
- Matrix orientation is cells-by-genes after the transpose: nrow(dat_log) equals the original number of cells (e.g. BC = 500 in the simulated example).
- Downstream EDGE run on dat_log reproduces the expected feature gene counts (17, 35, 43 for Jurkat, PBMC, mouse brain) and recovers PBMC markers PPBP, LYZ, CST3, NKG7 with top-scored S100A9.
- Normalization preserves gene/cell dimensions and does not reorder rows or columns unexpectedly.

## Limitations

- Version numbers for R, Splatter, Seurat, and RSpectra are not specified, so exact reproducibility of the normalization context is uncertain.
- The median-scaling choice is library-size-based only; it does not model cell-level detection rates or other technical covariates.

## Evidence

- [readme] Median library-size scaling recipe: "rs <- colSums(counts_dt); rs_med <- median(rs); dat_norm <- sweep(counts_dt,2,rs/rs_med,"/"); dat_log <- t(log2(dat_norm+1))"
- [readme] Normalization feeds the EDGE endr call: "simu.endr <- endr(dat_log,custom_defs)"
- [discussion] Gene filtering applies after normalization for real datasets: "We selected 1000 top variable genes for real datasets and 500 for simulated datasets"
- [discussion] Variable features computed with Seurat: "calculated by the FindVariableFeatures function in the R package Seurat"
- [other] Real-dataset validation via feature gene counts and markers: [evidence span withheld — anchor status: not_found]
