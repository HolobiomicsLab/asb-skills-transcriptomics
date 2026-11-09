---
name: sparse-single-cell-matrix-handling
description: Use when scoring gene signatures on sparse gene-by-cell scRNA-seq matrices
  (sparse matrix or SingleCellExperiment) and per-cell rank-based, dataset-composition-independent
  scores are required, or when reproducing ScoreSignatures_UCell numerically.
when_to_use_negative:
- When a dataset-composition-dependent but statistically averaged score is acceptable
  — note Seurat's AddModuleScore bins genes by average expression and gives inconsistent
  results for the same cell depending on dataset composition, but is a different method,
  not a reproduction target here
- When input is not a gene-by-cell expression matrix (e.g. already-computed signature
  scores or a feature table) — reranking is meaningless
- When comparing against UCell versions < 2.7.6 while using the new U_max = n*maxRank
  - n(n+1)/2 normalization, which will systematically shift scores for large gene
  sets
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: UCell
  role: R package providing ScoreSignatures_UCell (reference implementation), data_to_ranks,
    and u_stat routines being reconstructed
  repo: https://github.com/carmonalab/UCell
- name: data.table
  role: frank function used internally by UCell for efficient per-cell rank computations
- name: SingleCellExperiment
  role: accepted input container for the gene vs. cell matrix (sparse or SingleCellExperiment
    object)
- name: future
  role: parallel backend for batch processing of large datasets during rank/score
    computation
- name: R
  role: implementation language for the reconstructed data_to_ranks and u_stat routines
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:4a8ad2393e799bc589af444e89528a1bd845a6318be93235feaaaef5fc7b9d45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/UCell
  tools:
  - R
  - SingleCellExperiment
  - UCell
  - data.table
  - future
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sparse-single-cell-matrix-handling

## Summary

Reconstructs UCell's rank-based gene signature scoring: per-cell gene ranks computed with data.table::frank are capped at maxRank (1500 by default) to handle sparse zero-expression tails, then a Mann-Whitney U statistic is normalized to a [0,1] score per cell.

## When to use

Use when scoring gene signatures on sparse gene-by-cell scRNA-seq matrices (sparse matrix or SingleCellExperiment) and per-cell rank-based, dataset-composition-independent scores are required, or when reproducing ScoreSignatures_UCell numerically.

## When NOT to use

- When a dataset-composition-dependent but statistically averaged score is acceptable — note Seurat's AddModuleScore bins genes by average expression and gives inconsistent results for the same cell depending on dataset composition, but is a different method, not a reproduction target here
- When input is not a gene-by-cell expression matrix (e.g. already-computed signature scores or a feature table) — reranking is meaningless
- When comparing against UCell versions < 2.7.6 while using the new U_max = n*maxRank - n(n+1)/2 normalization, which will systematically shift scores for large gene sets

## Inputs

- gene-by-cell single-cell expression matrix (sparse matrix or SingleCellExperiment object)
- gene signature list (named list of gene symbols; supports +/- signed gene sets)
- maxRank parameter (default 1500)

## Outputs

- per-cell signature score matrix (cells x signatures) with values in [0, 1]
- intermediate relative-rank matrix R (genes x cells) if ranks are stored via StoreRankings_UCell

## How to apply

Given a gene-by-cell expression matrix X and a signature s of n genes, first compute the matrix of relative ranks R by sorting each column of X, using data.table::frank as UCell does internally. Apply the sparsity cap: set r(g,c) = rmax + 1 for all r(g,c) > rmax, with rmax = 1500 by default, collapsing the uninformative tail of zero-expression genes. Subset R to the signature genes and compute the Mann-Whitney U statistic per cell as U = sum(r') - n(n+1)/2, then normalize: score U' = 1 - U/U_max with U_max = n*maxRank - n(n+1)/2 (UCell >= 2.7.6; earlier versions used U_max = n*maxRank). Because scores depend only on relative gene ranks within individual cells, they are robust to dataset size and composition. Large datasets are automatically split into batches processed serially or in parallel via the future package. Validate by running the reference implementation ScoreSignatures_UCell on the same input and confirming element-wise agreement within small numerical tolerance, with all scores in [0, 1].

## Related tools

- **UCell** (R package providing ScoreSignatures_UCell (reference implementation), data_to_ranks, and u_stat routines being reconstructed) — https://github.com/carmonalab/UCell
- **data.table** (frank function used internally by UCell for efficient per-cell rank computations)
- **SingleCellExperiment** (accepted input container for the gene vs. cell matrix (sparse or SingleCellExperiment object))
- **future** (parallel backend for batch processing of large datasets during rank/score computation)
- **R** (implementation language for the reconstructed data_to_ranks and u_stat routines)

## Examples

```
scores <- ScoreSignatures_UCell(sample.matrix, features=gene.sets)  # reference; reconstruct via R <- apply(X, 2, data.table::frank); U <- colSums(R[sig,]) - n*(n+1)/2; s <- 1 - U/(n*rmax - n*(n+1)/2)
```

## Evaluation signals

- Reconstructed scores match ScoreSignatures_UCell output element-wise within a small numerical tolerance (report maximum absolute difference)
- All normalized scores fall within [0, 1], since U is bounded by 0 and U_max
- Rank matrix R contains no rank above rmax + 1 (capped tail), with rmax = 1500
- Evaluating a signature (e.g. CD8 T cell signature) on the full dataset vs. a subset yields identical score distributions for shared cells — the composition-invariance check
- Scores for a cell are unchanged if that cell's column is permuted into a differently composed dataset

## Limitations

- Sparsity: scores are computed on capped ranks, so genes with zero expression all share the rmax + 1 rank and dilute the U statistic for large signatures; kNN smoothing (SmoothKNN) is suggested to partially correct this
- Normalization changed in UCell >= 2.7.6 from U_max = n*maxRank to U_max = n*maxRank - n(n+1)/2; results differ somewhat for large gene sets
- Very large gene sets give more homogeneous but compressed score distributions under the new normalization

## Evidence

- [methods] rank matrix computation: "we first calculate the matrix of relative ranks R by sorting each column in X; in other words, we calculate a ranked list of genes for each cell in the dataset."
- [methods] rank capping for sparse data: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"
- [methods] frank implementation detail: "Internally, UCell uses the frank function from the data.table package [9] for efficient ranks computations."
- [readme] score normalization formula: [evidence span withheld — anchor status: external_doc]
- [methods] robustness rationale: "This score depends only on the gene expression ranks of individual cell, and therefore is robust across datasets."
