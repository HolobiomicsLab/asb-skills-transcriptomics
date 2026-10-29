---
name: sc-rnaseq-library-size-normalization
description: Use when a raw scRNA-seq count matrix (e.g., counts_dt from splatSimulate,
  or the 3005-cell mouse cortex/hippocampus matrix) must be depth-normalized and log2-transformed
  before running endr, UMAP, or t-SNE.
when_to_use_negative:
- The input is already a log-normalized matrix (dat_log), a similarity matrix, or
  a 2D embedding — re-normalizing would corrupt values meant for endr/UMAP/t-SNE
- The downstream task requires raw counts (e.g., count-based modeling in Splatter
  params) rather than log-normalized expression
- You only need feature-gene selection; normalization here does not replace the separate
  top-variable-gene filter (1000 genes real / 500 simulated)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EDGE
  role: Downstream consumer of the log-normalized matrix dat_log passed to endr(dat_log,
    custom_defs)
  repo: github.com/shawnstat/EDGE
- name: Splatter
  role: Simulates the raw count matrix (splatSimulate, 500 cells, 200 genes, two groups
    with a 15-cell minor group) whose counts_dt feeds this normalization step
- name: R
  role: Base R functions colSums, median, sweep, log2, and t implement the size-factor
    scaling and log2(x+1) transform
- name: umap
  role: Baseline embedding run on the same normalized dat_log with 15 nearest neighbors
    (umap(dat_log, random_state=2020))
- name: Rtsne
  role: Baseline t-SNE embedding run on dat_log with seed 2020
merged_aliases:
- scrnaseq-library-size-normalization
merged_alias_records:
- alias: scrna-seq-library-size-normalization
  slug: scrna-seq-library-size-normalization
  cosine_score: 0.9203
  method: embedding-cosine
  decision: auto
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:35da691a4c98fd4af0c09fdc42487f409d059a7554776497b24980272ad1e7b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - R
  - RSpectra
  - Rcpp
  - Rtsne
  - Splatter
  - umap
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# sc-rnaseq-library-size-normalization

## Summary

Median library-size normalization followed by log2(x+1) transformation of a raw gene-by-cell scRNA-seq count matrix, producing the log-normalized cell-by-gene matrix (dat_log) that EDGE's endr pipeline, UMAP, and t-SNE consume.

## When to use

Use when a raw scRNA-seq count matrix (e.g., counts_dt from splatSimulate, or the 3005-cell mouse cortex/hippocampus matrix) must be depth-normalized and log2-transformed before running endr, UMAP, or t-SNE.

## When NOT to use

- The input is already a log-normalized matrix (dat_log), a similarity matrix, or a 2D embedding — re-normalizing would corrupt values meant for endr/UMAP/t-SNE
- The downstream task requires raw counts (e.g., count-based modeling in Splatter params) rather than log-normalized expression
- You only need feature-gene selection; normalization here does not replace the separate top-variable-gene filter (1000 genes real / 500 simulated)

## Inputs

- Raw gene-by-cell scRNA-seq count matrix (counts_dt), e.g., from splatSimulate with 500 cells and 200 genes, or the 3005-cell mouse cortex/hippocampus count matrix
- Per-cell library sizes (column sums rs) and their median (rs_med), derived from the count matrix

## Outputs

- dat_log: log-normalized cell-by-gene matrix (rows = cells, columns = genes), values >= 0 and finite
- Optional dat_log.rds artifact saved for reproducible downstream embedding

## How to apply

Compute each cell's library size as the column sums (rs) of the gene-by-cell count matrix, take their median (rs_med), and scale each cell by its size factor rs/rs_med via column-wise sweep to obtain dat_norm. Apply log2(x+1), then transpose so rows are cells and columns are genes, yielding dat_log; optionally save as dat_log.rds. This dat_log matrix is the mandated input to endr(dat_log, custom_defs), umap(dat_log, random_state=2020), and Rtsne(dat_log) with seed 2020. Per the paper's broader workflow, follow with selection of 1000 top variable genes for real datasets (500 for simulated ones) before embedding. The log2(x+1) transform with a +1 pseudo-count is what the Methods (Data Preprocessing) specify, so do not substitute a different normalization unless the downstream method requires it.

## Related tools

- **EDGE** (Downstream consumer of the log-normalized matrix dat_log passed to endr(dat_log, custom_defs)) — github.com/shawnstat/EDGE
- **Splatter** (Simulates the raw count matrix (splatSimulate, 500 cells, 200 genes, two groups with a 15-cell minor group) whose counts_dt feeds this normalization step)
- **R** (Base R functions colSums, median, sweep, log2, and t implement the size-factor scaling and log2(x+1) transform)
- **umap** (Baseline embedding run on the same normalized dat_log with 15 nearest neighbors (umap(dat_log, random_state=2020)))
- **Rtsne** (Baseline t-SNE embedding run on dat_log with seed 2020)

## Examples

```
dat_norm <- sweep(counts_dt,2,colSums(counts_dt)/median(colSums(counts_dt)),"/"); dat_log <- t(log2(dat_norm+1))
```

## Evaluation signals

- Output matrix dimensions are correct: 500 rows (cells) x 200 columns (genes) for the README simulation example, and 3005 cells for the mouse brain dataset
- All entries of dat_log are non-negative and finite after log2(x+1)
- After sweep normalization, each cell's scaled library size equals the median library size rs_med of the original columns
- Rows correspond to cells and columns to genes after transposition — orientation errors surface as dimension failures in endr(dat_log, custom_defs)
- Downstream endr/UMAP/t-SNE runs with seed 2020 complete without error, and 100-replication silhouette averages are reproducible against reported values (EDGE 0.621, UMAP 0.597, t-SNE 0.501) within Monte Carlo tolerance

## Limitations

- Median normalization assumes library-size differences reflect sequencing depth only; it does not correct for batch effects or other technical covariates
- The step must be paired with the paper's subsequent gene filter (top 1000 variable genes for real data, 500 for simulated) — normalization alone does not perform feature selection
- Normalization is defined on the gene-by-cell orientation (cells as columns) via column-wise sweep; forgetting the transpose to cell-by-gene rows before endr breaks downstream calls

## Evidence

- [readme] rs <- colSums(counts_dt)
    rs_med <- median(rs)
    dat_norm <- sweep(counts_dt,2,rs/rs_med,"/")
    dat_log <- t(log2(dat_norm+1)): "rs <- colSums(counts_dt)
    rs_med <- median(rs)
    dat_norm <- sweep(counts_dt,2,rs/rs_med,"/")
    dat_log <- t(log2(dat_norm+1))"
- [readme] We will use the simulator, Splatter, to simulate a single cell RNA-seq dataset with 500 cells and 200 genes.: "We will use the simulator, Splatter, to simulate a single cell RNA-seq dataset with 500 cells and 200 genes."
- [methods] simu.endr <- endr(dat_log,custom_defs): "simu.endr <- endr(dat_log,custom_defs)"
- [discussion] We selected 1000 top variable genes for real datasets and 500 for simulated datasets: "We selected 1000 top variable genes for real datasets and 500 for simulated datasets"
