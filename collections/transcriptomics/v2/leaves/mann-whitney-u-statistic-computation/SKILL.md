---
name: mann-whitney-u-statistic-computation
description: Use when computing per-cell gene signature scores from a gene-by-cell
  scRNA-seq matrix (sparse matrix, SingleCellExperiment, or Seurat object) via rank-based
  Mann-Whitney U statistics, or when reimplementing/porting UCell's scoring mechanism.
when_to_use_negative:
- You need scores that depend on dataset-wide normalization — UCell ranks are purely
  per-cell relative expression, unlike Seurat's AddModuleScore, which bins genes by
  average expression across the whole dataset and yields composition-dependent results.
- You want to compare absolute expression levels of genes across cells rather than
  within-cell relative ordering.
- Your reference UCell version differs in normalization (pre- vs post-2.7.6 U_max
  definition) — mixing the two normalizations will produce systematically offset scores
  for large gene sets.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: UCell
  role: Reference implementation providing ScoreSignatures_UCell, data_to_ranks, and
    u_stat used to validate the reconstructed scoring mechanism
  repo: https://github.com/carmonalab/UCell
- name: data.table
  role: Provides the frank function UCell uses internally for efficient per-cell rank
    computations (data_to_ranks_data_table)
- name: SingleCellExperiment
  role: Accepted input container for the gene-vs-cell matrix fed to the scoring routine
- name: future
  role: Enables parallel processing of rank/score batches for large datasets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:98f32ba87cf4a668dd9f03cbdda88f9f1a6551f06cbb46d34eb0f9fb90feffd6
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# mann-whitney-u-statistic-computation

## Summary

Reconstructs UCell's rank-based gene signature scoring: per-cell gene ranks are capped at maxRank (default 1500), the Mann-Whitney U statistic is computed from signature-gene rank sums, and the score is normalized to [0,1]. This enables faithful reimplementation or cross-validation of UCell scoring (data_to_ranks + u_stat) in R or Python (pyUCell).

## When to use

Use when computing per-cell gene signature scores from a gene-by-cell scRNA-seq matrix (sparse matrix, SingleCellExperiment, or Seurat object) via rank-based Mann-Whitney U statistics, or when reimplementing/porting UCell's scoring mechanism.

## When NOT to use

- You need scores that depend on dataset-wide normalization — UCell ranks are purely per-cell relative expression, unlike Seurat's AddModuleScore, which bins genes by average expression across the whole dataset and yields composition-dependent results.
- You want to compare absolute expression levels of genes across cells rather than within-cell relative ordering.
- Your reference UCell version differs in normalization (pre- vs post-2.7.6 U_max definition) — mixing the two normalizations will produce systematically offset scores for large gene sets.

## Inputs

- gene-by-cell single-cell expression matrix (sparse matrix or SingleCellExperiment object; Seurat object via AddModuleScore_UCell wrapper)
- gene signature list (named list of gene sets; supports +/- positive/negative gene annotations)
- maxRank parameter (default 1500)

## Outputs

- per-cell, per-signature UCell scores U' in [0, 1]
- relative-rank matrix R (optional intermediate; storeable via StoreRankings_UCell)

## How to apply

First build the relative-rank matrix R by sorting each column of the gene-by-cell expression matrix X (a ranked list of genes per cell), using data.table::frank as UCell does internally (data_to_ranks_data_table). Apply the rank cap: set r(g,c) = rmax + 1 for every rank above rmax, with rmax = 1500 by default, to collapse the uninformative tail of zero-expression genes. For each signature of n genes, compute per cell the Mann-Whitney U statistic U = sum(r_i) - n(n+1)/2 and normalize it to U' = 1 - U/U_max; match the normalization to the installed UCell version — U_max = n*maxRank - n(n+1)/2 for UCell >= 2.7.6, versus the older U_max = n*maxRank. Signatures may contain positive and negative gene sets by appending + or - signs to gene names (e.g. c("CD2+","CD8A+","CD4-")). For large datasets, batch cells and process serially (memory-minimizing) or in parallel via the future package, then validate by comparing reconstructed scores element-wise against ScoreSignatures_UCell on the same matrix and signature.

## Related tools

- **UCell** (Reference implementation providing ScoreSignatures_UCell, data_to_ranks, and u_stat used to validate the reconstructed scoring mechanism) — https://github.com/carmonalab/UCell
- **data.table** (Provides the frank function UCell uses internally for efficient per-cell rank computations (data_to_ranks_data_table))
- **SingleCellExperiment** (Accepted input container for the gene-vs-cell matrix fed to the scoring routine)
- **future** (Enables parallel processing of rank/score batches for large datasets)

## Examples

```
scores <- ScoreSignatures_UCell(sample.matrix, features=gene.sets)
```

## Evaluation signals

- Reconstructed scores match ScoreSignatures_UCell output element-wise with a maximum absolute difference within a small numerical tolerance.
- All scores fall within [0, 1], with U bounded by 0 and U_max.
- Ranks above rmax = 1500 collapse exactly to rmax + 1 in the rank matrix, leaving only the top-1500 genes informative.
- Score distributions for a given cell population are identical whether computed on the full heterogeneous dataset or on the population subset (robustness to dataset composition).
- For UCell >= 2.7.6, verify U_max = n*maxRank - n(n+1)/2 rather than n*maxRank, especially for large gene sets where the two normalizations diverge.

## Limitations

- UCell >= 2.7.6 changed the normalization from U_max = n*maxRank to U_max = n*maxRank - n(n+1)/2; results differ between implementations, particularly for large gene sets, so the reference version must be pinned.
- Ranks above maxRank (1500 by default) are collapsed to rmax + 1, so signatures whose genes fall almost entirely in the zero-expression tail receive near-floor scores.
- Scores depend only on within-cell relative ranks, capturing no absolute expression information.
- Single-cell data sparsity can bias scores; UCell offers SmoothKNN imputation by weighted average of k nearest-neighbor scores as a partial correction.

## Evidence

- [methods] Rank cap to rmax: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"
- [methods] data.table frank ranking: "Internally, UCell uses the frank function from the data.table package [9] for efficient ranks computations."
- [methods] Relative-rank matrix construction: "we first calculate the matrix of relative ranks R by sorting each column in X; in other words, we calculate a ranked list of genes for each cell in the dataset."
- [readme] Updated U_max normalization (>=2.7.6): [evidence span withheld — anchor status: external_doc]
- [readme] Legacy normalization caveat: [evidence span withheld — anchor status: external_doc]
