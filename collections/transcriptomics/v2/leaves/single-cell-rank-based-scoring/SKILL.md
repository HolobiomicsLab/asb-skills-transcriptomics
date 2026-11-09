---
name: single-cell-rank-based-scoring
description: Use when you need per-cell enrichment scores for gene signatures (e.g.,
  cell-type or T-cell/myeloid marker sets) on a gene-by-cell scRNA-seq matrix, SingleCellExperiment,
  or Seurat object, and want scores invariant to dataset composition.
when_to_use_negative:
- You need absolute expression-based scores tied to dataset-wide gene binning, as
  in Seurat's AddModuleScore (UCell is deliberately rank-based and dataset-composition-invariant).
- The input is not single-cell expression data (e.g., already aggregated feature tables
  or bulk signatures without per-cell rank structure).
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UCell
  role: Core R package implementing rank-based Mann-Whitney U signature scoring (ScoreSignatures_UCell,
    AddModuleScore_UCell, StoreRankings_UCell, SmoothKNN)
  repo: https://github.com/carmonalab/UCell
- name: data.table
  role: Provides the frank function used internally by UCell for efficient per-cell
    rank computations (data_to_ranks_data_table)
- name: future
  role: Enables parallel processing of rank/score batches to minimize execution time
- name: SingleCellExperiment
  role: Supported input container for gene-vs-cell matrices scored by UCell
- name: Seurat
  role: Object framework that UCell interacts with directly through AddModuleScore_UCell
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:2fa3ba83e4ad755785984cb89977d20276904ccba0c26d365d86f5c844201e77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/UCell
  tools:
  - R
  - Seurat
  - SingleCellExperiment
  - UCell
  - data.table
  - future
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# single-cell-rank-based-scoring

## Summary

Score gene signatures in single-cell RNA-seq data using UCell, a rank-based method built on the Mann-Whitney U statistic whose scores depend only on relative gene expression within each cell and are thus robust to dataset size and composition.

## When to use

Use when you need per-cell enrichment scores for gene signatures (e.g., cell-type or T-cell/myeloid marker sets) on a gene-by-cell scRNA-seq matrix, SingleCellExperiment, or Seurat object, and want scores invariant to dataset composition.

## When NOT to use

- You need absolute expression-based scores tied to dataset-wide gene binning, as in Seurat's AddModuleScore (UCell is deliberately rank-based and dataset-composition-invariant).
- The input is not single-cell expression data (e.g., already aggregated feature tables or bulk signatures without per-cell rank structure).

## Inputs

- gene-vs-cell expression matrix (dense or sparse)
- SingleCellExperiment object
- Seurat object (via AddModuleScore_UCell)
- named list of gene signatures (character vectors of gene symbols; +/- suffixes denote positive/negative sets)

## Outputs

- data frame of per-cell signature scores (one row per cell, one column per signature) with normalized U statistics (AUC-equivalent) in [0, 1]
- stored rank matrix via StoreRankings_UCell
- kNN-smoothed scores added to object metadata via SmoothKNN

## How to apply

Load a gene-vs-cell expression matrix (dense or sparse, or a SingleCellExperiment/Seurat object) and define the signature as a named list of gene sets, e.g. list(Tcell_signature=c('CD2','CD3E','CD3D'), Myeloid_signature=c('SPI1','FCER1G','CSF1R')). Call ScoreSignatures_UCell, which internally computes a matrix of relative ranks R per cell with the data.table frank function, then caps ranks: r = rmax + 1 for all ranks above rmax (rmax = 1500 by default) to collapse the uninformative tail of zero-expression genes. For each signature of n genes, compute the Mann-Whitney U = sum(r_i) - n(n+1)/2 and normalize via UCell = 1 - U/Umax with Umax = n*maxRank - n(n+1)/2 (versions >= 2.7.6; earlier used n*maxRank). Large datasets are automatically split into batches processed serially (minimizing memory) or in parallel via future. Optionally smooth sparse scores with SmoothKNN over k nearest neighbors in a dimensionality reduction.

## Related tools

- **UCell** (Core R package implementing rank-based Mann-Whitney U signature scoring (ScoreSignatures_UCell, AddModuleScore_UCell, StoreRankings_UCell, SmoothKNN)) — https://github.com/carmonalab/UCell
- **data.table** (Provides the frank function used internally by UCell for efficient per-cell rank computations (data_to_ranks_data_table))
- **future** (Enables parallel processing of rank/score batches to minimize execution time)
- **SingleCellExperiment** (Supported input container for gene-vs-cell matrices scored by UCell)
- **Seurat** (Object framework that UCell interacts with directly through AddModuleScore_UCell) — https://github.com/satijalab/seurat

## Examples

```
library(UCell); gene.sets <- list(Tcell_signature=c('CD2','CD3E','CD3D'), Myeloid_signature=c('SPI1','FCER1G','CSF1R')); scores <- ScoreSignatures_UCell(sample.matrix, features=gene.sets); head(scores)
```

## Evaluation signals

- Output table contains exactly one row per cell and one column per signature.
- All scores are normalized U statistics (AUC-equivalent) bounded in [0, 1].
- A from-scratch reconstruction (data.table frank ranking + rmax=1500 cap + u_stat) matches ScoreSignatures_UCell output within a small numerical tolerance (max absolute difference near machine precision).
- CD8 T cell signature score distributions are identical whether computed on the full dataset or on CD8 T cells only (composition invariance).
- head(scores) on the bundled sample.matrix reproduces the quick-start vignette output.

## Limitations

- Scores depend only on relative gene ranks, so they do not reflect absolute expression levels; genes beyond the rank cap (rmax=1500) are treated as equivalent at the lowest ranking value, diluting signal in low-expression signatures.
- Single-cell data sparsity can yield noisy scores; SmoothKNN imputation over k nearest neighbors is recommended but adds a smoothing step.
- Normalization changed in version >= 2.7.6 (Umax = n*maxRank - n(n+1)/2 vs earlier n*maxRank), so scores are not strictly comparable across versions, though differences are small for typical applications.

## Evidence

- [methods] UCell calculates gene signature scores for scRNA-seq data based on the Mann-Whitney U statistic: "UCell calculates gene signature scores for scRNA-seq data based on the Mann-Whitney U statistic"
- [methods] we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"
- [methods] Internally, UCell uses the frank function from the data.table package [9] for efficient ranks computations.: "Internally, UCell uses the frank function from the data.table package [9] for efficient ranks computations."
- [readme] scores <- ScoreSignatures_UCell(sample.matrix, features=gene.sets): "scores <- ScoreSignatures_UCell(sample.matrix, features=gene.sets)"
- [results] UCell scores depend only on the relative ranking of genes for individual cells, therefore they are robust to dataset composition.: "UCell scores are based on the relative ranking of genes for individual cells, therefore they are robust to dataset composition."
