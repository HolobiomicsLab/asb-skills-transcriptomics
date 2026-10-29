---
name: top-variable-gene-selection
description: Use when preparing raw scRNA-seq count matrices (cells x genes) as input
  to EDGE or a similar ensemble feature-extraction method, and you need log-normalization
  plus a top-variable-gene filter to reduce dimensionality of the candidate gene space.
when_to_use_negative:
- Input is already log-normalized and gene-filtered (e.g., a cells-by-genes log matrix
  from a prior pipeline step) — re-filtering could discard signal genes
- The dataset is simulated with Splatter and fewer than 500 genes exist — the 500-gene
  top-variable filter is unnecessary and the real-dataset 1000-gene rule does not
  apply
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: EDGE
  role: R package whose endr() function consumes the log-normalized top-variable-gene
    matrix to perform ensemble dimensionality reduction and feature gene extraction
  repo: https://github.com/shawnstat/EDGE
- name: Seurat
  role: Provides the FindVariableFeatures function used to identify the top variable
    genes
- name: Splatter
  role: Simulates scRNA-seq count matrices used to validate the filtering settings
    on simulated data
- name: RSpectra
  role: Dependency used by EDGE for large-scale eigenvalue decomposition during spectral
    embedding
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:e48e648559991346035893bcc9ef674642bb8806c5309711c50138bf2fe8f9fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shawnstat/EDGE
  tools:
  - EDGE
  - Monocle3
  - R
  - RSpectra
  - Rcpp
  - Rtsne
  - SCORPIUS
  - Seurat
  - Splatter
  - mclust
  - scran
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- highly-variable-gene-selection
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# top-variable-gene-selection

## Summary

Preprocesses scRNA-seq count matrices by library-size normalization (scaling to median totals, log2(x+1) transformation) and restricting to the most variable genes (1000 for real datasets, 500 for simulated) before running EDGE feature gene identification.

## When to use

Use when preparing raw scRNA-seq count matrices (cells x genes) as input to EDGE or a similar ensemble feature-extraction method, and you need log-normalization plus a top-variable-gene filter to reduce dimensionality of the candidate gene space.

## When NOT to use

- Input is already log-normalized and gene-filtered (e.g., a cells-by-genes log matrix from a prior pipeline step) — re-filtering could discard signal genes
- The dataset is simulated with Splatter and fewer than 500 genes exist — the 500-gene top-variable filter is unnecessary and the real-dataset 1000-gene rule does not apply

## Inputs

- Raw scRNA-seq count matrix (genes x cells, e.g., from Splatter counts() or a real dataset count table)
- Per-cell library sizes (colSums of the count matrix)

## Outputs

- Cells-by-genes log-normalized matrix dat_log (log2(x+1) after median library-size scaling)
- Top variable gene set (1000 genes for real datasets, 500 for simulated datasets)
- Ranked feature gene list with averaged entropy scores, filtered at mu - 1.5 x sigma

## How to apply

Load the raw count matrix and scale each cell by its library size ratio to the median total count using sweep (e.g., sweep(counts_dt,2,rs/rs_med,"/")), then transform with log2(x+1) and transpose to a cells-by-genes matrix dat_log. Compute highly variable genes with the Seurat FindVariableFeatures approach and keep the top 1000 variable genes for real datasets (Jurkat, PBMC, mouse brain) or 500 for Splatter-simulated datasets. Feed the resulting log-normalized, gene-filtered matrix into EDGE's endr(dat_log, custom_defs) with the ensemble configuration (n_wl=1000, n_dm=10, n_neigs=15, opt=TRUE, seed=2020, H=101107). Rank genes by averaged entropy contribution across weak learners and select feature genes at the mu - 1.5 x sigma cutoff.

## Related tools

- **EDGE** (R package whose endr() function consumes the log-normalized top-variable-gene matrix to perform ensemble dimensionality reduction and feature gene extraction) — https://github.com/shawnstat/EDGE
- **Seurat** (Provides the FindVariableFeatures function used to identify the top variable genes)
- **Splatter** (Simulates scRNA-seq count matrices used to validate the filtering settings on simulated data)
- **RSpectra** (Dependency used by EDGE for large-scale eigenvalue decomposition during spectral embedding)

## Examples

```
dat_norm <- sweep(counts_dt,2,colSums(counts_dt)/median(colSums(counts_dt)),"/"); dat_log <- t(log2(dat_norm+1)); simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- On real datasets (Jurkat, PBMC, mouse brain), the reproduced feature gene lists must contain exactly 17, 35, and 43 genes respectively, matching Supplementary Data 1
- The PBMC feature gene list must include PPBP, LYZ, CST3, and NKG7 (markers for platelet, CD14+ monocyte, dendritic, and NK cells), with S100A9 having the highest importance score
- Check dat_log is a cells-by-genes matrix where each cell's total equals the median library size of the original counts after scaling
- Confirm the gene filter retained the correct number of top variable genes: 1000 for real datasets, 500 for simulated ones
- Feature gene counts should match the mu - 1.5 x sigma cutoff applied to averaged entropy values across 1000 weak learners

## Limitations

- The optimal hash table size 1,017,881 requires large memory; the example reduces it to 101,107, which works most of the time but may degrade similarity search accuracy
- The exact per-dataset number of genes per weak learner (B) is not stated, only a heuristic that values for B are typically <0.05*G
- No version numbers are given for R, Splatter, Seurat, or RSpectra, so reproducibility depends on package versions not documented here

## Evidence

- [readme] Library-size normalization and log transform: "dat_norm <- sweep(counts_dt,2,rs/rs_med,"/") ; dat_log <- t(log2(dat_norm+1))"
- [discussion] Top variable gene filter: "We selected 1000 top variable genes for real datasets and 500 for simulated datasets"
- [discussion] HVG tool: "calculated by the FindVariableFeatures function in the R package Seurat"
- [discussion] Feature gene cutoff: "We choose μ −  1.5 × σ as the cutoff value to select top feature genes"
- [readme] endr invocation with ensemble hyperparameters: "simu.endr <- endr(dat_log,custom_defs)"
