---
name: precomputed-rankings-score-acceleration
description: Use when you need to score multiple or evolving gene signatures on the
  same single-cell dataset and want to avoid recomputing the expensive gene-ranking
  step for every new signature.
when_to_use_negative:
- 'When the expression matrix changes (new cells or re-normalized counts): stored
  ranks no longer reflect relative gene expression and must be recomputed'
- When scoring only a single signature once — direct ScoreSignatures_UCell on the
  expression matrix is simpler and incurs no extra storage
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: UCell
  role: Provides StoreRankings_UCell for one-time gene-rank calculation and ScoreSignatures_UCell
    for fast scoring over pre-calculated ranks
  repo: https://github.com/carmonalab/UCell
- name: SingleCellExperiment
  role: Accepted input container for the gene-vs-cell matrix passed to StoreRankings_UCell
- name: data.table
  role: Underlying frank function used by UCell for efficient rank computations
- name: future
  role: Parallel backend to minimize execution time of ranking/scoring batches
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:89dc59329a05069adbc95b2c47ccda6dd4098d5ccc9c0563d596fd816e44c928
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# precomputed-rankings-score-acceleration

## Summary

Accelerate UCell gene signature scoring by pre-calculating per-cell gene ranks once with StoreRankings_UCell and reusing the stored rank matrix with ScoreSignatures_UCell to rapidly score both original and newly defined signatures (e.g., Mast.cell, Lymphoid).

## When to use

Use when you need to score multiple or evolving gene signatures on the same single-cell dataset and want to avoid recomputing the expensive gene-ranking step for every new signature.

## When NOT to use

- When the expression matrix changes (new cells or re-normalized counts): stored ranks no longer reflect relative gene expression and must be recomputed
- When scoring only a single signature once — direct ScoreSignatures_UCell on the expression matrix is simpler and incurs no extra storage

## Inputs

- Gene-vs-cell expression matrix (sparse matrix or SingleCellExperiment object, e.g., downsampled Zilionis gene-vs-cell matrix)
- Gene signature list (named list of character vectors, extendable with new signatures such as Mast.cell and Lymphoid)

## Outputs

- Stored gene-rank matrix (precalc.ranks) from StoreRankings_UCell
- Per-cell signature score table (cells x signatures Mann-Whitney U enrichment scores, e.g., written to ucell_scores.csv)

## How to apply

Load the gene-vs-cell expression matrix (e.g., the downsampled Zilionis matrix, sparse matrix or SingleCellExperiment) into R and call StoreRankings_UCell once, keeping the default rmax = 1500 cap so genes ranked above rmax are assigned rank rmax + 1, which mitigates the uninformative tail of zero-expression genes. Extend the signature list with any new signatures such as Mast.cell = c("TPSAB1","TPSB2","CPA3","MS4A2") and Lymphoid = c("LCK"). Run ScoreSignatures_UCell directly on the stored rankings (precalc.ranks) to compute the Mann-Whitney U-based, AUC-equivalent enrichment scores; because scores depend only on relative gene ranks per cell, results match direct scoring while skipping the ranking computation. Optionally parallelize with the future package or batch large datasets. Verify score distributions and ranges against those from scoring the expression matrix directly (or the basic UCell vignette values) before exporting, e.g., via write.csv to ucell_scores.csv.

## Related tools

- **UCell** (Provides StoreRankings_UCell for one-time gene-rank calculation and ScoreSignatures_UCell for fast scoring over pre-calculated ranks) — https://github.com/carmonalab/UCell
- **SingleCellExperiment** (Accepted input container for the gene-vs-cell matrix passed to StoreRankings_UCell)
- **data.table** (Underlying frank function used by UCell for efficient rank computations)
- **future** (Parallel backend to minimize execution time of ranking/scoring batches)

## Examples

```
library(UCell); precalc.ranks <- StoreRankings_UCell(expr.matrix); sigs$Mast.cell <- c("TPSAB1","TPSB2","CPA3","MS4A2"); sigs$Lymphoid <- c("LCK"); scores <- ScoreSignatures_UCell(precalc.ranks, features=sigs)
```

## Evaluation signals

- Score distributions and ranges from ScoreSignatures_UCell on precalc.ranks match those from scoring the raw expression matrix directly and agree with values in the basic UCell vignette
- Stored rank matrix caps at rmax: all ranks > 1500 are set to 1501 (rmax + 1)
- Every cell has a non-NA score for every signature in the extended list (original signatures plus Mast.cell and Lymphoid)
- Ranks are computed only once; re-running ScoreSignatures_UCell with additional signatures does not recompute rankings and completes considerably faster
- Exported ucell_scores.csv has one row per cell and one column per signature, with scores in the expected 0–1 range

## Limitations

- UCell normalization changed in version >= 2.7.6 (U_max = n·maxRank − n(n+1)/2), so scores may differ slightly from earlier releases, especially for large gene sets
- Pre-calculated ranks are dataset-specific; the rmax = 1500 cap discards information from the tail of zero-expression genes
- For very large datasets, ranking is memory-heavy unless split into batches processed serially or in parallel

## Evidence

- [methods] rmax cap quote: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"
- [methods] StoreRankings function: "`StoreRankings_UCell`   Calculate and store gene rankings for a single-cell dataset."
- [methods] ScoreSignatures function: "`ScoreSignatures_UCell`    Calculate module enrichment scores from single-cell data."
- [readme] robustness rationale: "UCell scores, based on the Mann-Whitney U statistic, are robust to dataset size and heterogeneity"
