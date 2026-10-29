---
name: rank-capping-zero-inflation-tail-correction
description: Use when scoring gene signatures on sparse single-cell RNA-seq matrices
  where thousands of zero-expression genes per cell would dominate per-cell gene rankings,
  and scores must be robust to dataset size and composition.
when_to_use_negative:
- Input is already a scored feature table or enrichment score matrix — rank capping
  applies to raw expression matrices only
- The analysis requires scores sensitive to dataset composition or absolute expression
  levels (e.g. comparing cells across deliberately different expression scales)
- Using Seurat's AddModuleScore-style binning is intended instead, since UCell's rank-based
  approach is specifically an alternative whose scores depend only on per-cell relative
  ranks
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UCell
  role: reference implementation providing ScoreSignatures_UCell, data_to_ranks_data_table,
    and u_stat routines for rank capping and U-statistic scoring
  repo: https://github.com/carmonalab/UCell
- name: data.table
  role: provides the frank function used internally for efficient per-cell rank computations
- name: SingleCellExperiment
  role: supported input container for the gene vs. cell matrix passed to rank computation
- name: future
  role: enables parallel processing of rank batches for large datasets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:348210245e5a842968e9df6032e40e39cec5dcb7c452533a475b85df9d41f468
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# rank-capping-zero-inflation-tail-correction

## Summary

UCell's sparsity-handling mechanism: per-cell gene expression ranks are computed (via data.table frank) and capped at a maxRank parameter (default 1500), setting all genes ranked above rmax to rmax+1 so the uninformative zero-expression tail does not distort Mann-Whitney U-based signature scores.

## When to use

Use when scoring gene signatures on sparse single-cell RNA-seq matrices where thousands of zero-expression genes per cell would dominate per-cell gene rankings, and scores must be robust to dataset size and composition.

## When NOT to use

- Input is already a scored feature table or enrichment score matrix — rank capping applies to raw expression matrices only
- The analysis requires scores sensitive to dataset composition or absolute expression levels (e.g. comparing cells across deliberately different expression scales)
- Using Seurat's AddModuleScore-style binning is intended instead, since UCell's rank-based approach is specifically an alternative whose scores depend only on per-cell relative ranks

## Inputs

- gene-by-cell single-cell expression matrix (sparse matrix or SingleCellExperiment object)
- gene signature list (e.g. list(Tcell_signature=c("CD2","CD3E","CD3D")))
- maxRank parameter (default 1500)

## Outputs

- per-cell, per-signature normalized UCell scores in [0, 1]
- optional: stored relative-rank matrix R (via StoreRankings_UCell)

## How to apply

Given a gene-by-cell matrix (sparse matrix or SingleCellExperiment) and a gene signature of n genes, first compute the relative-rank matrix R by sorting each column with data.table's frank function as done internally by UCell. Then apply the cap: set r(g,c) = rmax + 1 for every rank above rmax (rmax = 1500 by default), collapsing the uninformative tail of zero-expression genes. For each cell, subset R to the signature genes and compute the Mann-Whitney U statistic U = sum(r_i) - n(n+1)/2, then normalize to U' = 1 - U/U_max with U_max = n*maxRank - n(n+1)/2 (UCell >= 2.7.6; earlier versions used U_max = n*maxRank). Rationale: scores depend only on relative gene expression ranks within individual cells, making them unaffected by dataset composition unlike Seurat's AddModuleScore. Validate by comparing reconstructed scores against ScoreSignatures_UCell output within a small numerical tolerance and checking all scores fall in [0, 1].

## Related tools

- **UCell** (reference implementation providing ScoreSignatures_UCell, data_to_ranks_data_table, and u_stat routines for rank capping and U-statistic scoring) — https://github.com/carmonalab/UCell
- **data.table** (provides the frank function used internally for efficient per-cell rank computations)
- **SingleCellExperiment** (supported input container for the gene vs. cell matrix passed to rank computation)
- **future** (enables parallel processing of rank batches for large datasets)

## Examples

```
scores <- ScoreSignatures_UCell(sample.matrix, features=gene.sets, maxRank=1500)
```

## Evaluation signals

- Reconstructed scores match ScoreSignatures_UCell output element-wise within a small numerical tolerance (maximum absolute difference near machine precision)
- All signature scores fall within the interval [0, 1]
- Ranks above rmax equal exactly rmax + 1 (1501 with default maxRank), confirming the cap collapsed the zero-expression tail
- U statistic for a cell never exceeds U_max = n*maxRank - n(n+1)/2
- Score distributions for a given cell type are identical whether the signature is evaluated on the full dataset or on that cell type alone (dataset-composition invariance)

## Limitations

- Genes ranked above maxRank are assumed equivalent at the lowest ranking value, so the cap discards rank information in the uninformative tail and can affect scores for signatures containing very lowly expressed genes
- UCell >= 2.7.6 changed the normalization from U_max = n*maxRank to n*maxRank - n(n+1)/2; results from versions mixing these normalizations will differ, particularly for large gene sets
- Scores depend only on relative ranks, so they do not reflect absolute expression differences across datasets

## Evidence

- [methods] rank capping threshold and rationale: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"
- [methods] rank computation tooling: "Internally, UCell uses the frank function from the data.table package [9] for efficient ranks computations."
- [readme] new U-statistic normalization (README): [evidence span withheld — anchor status: external_doc]
- [readme] default maxRank capping: [evidence span withheld — anchor status: not_found]
- [abstract] robustness rationale: "UCell signature scores, based on the Mann-Whitney U statistic, are robust to dataset size and heterogeneity"
