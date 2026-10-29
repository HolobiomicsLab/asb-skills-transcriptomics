---
name: scrna-seq-outlier-cell-detection
description: Use when a raw scRNA-seq count matrix (genes × cells) is being imputed
  with scImpute and aberrant cells should be flagged and excluded before dropout imputation.
when_to_use_negative:
- The input is already normalized, dimension-reduced, or a feature/cell-type table
  rather than a raw whole-genome count matrix — scImpute recommends applying it on
  the whole-genome count matrix with most genes present
- The goal is general-purpose outlier detection independent of dropout imputation
  — this rule is coupled to scImpute's imputation pipeline and serves to exclude outlier
  cells from imputation
- The number of cells is extremely large without a prior cell filtering step — computational
  efficiency is limited in that setting per the article's discussion
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: scImpute
  role: 'Implements the outlier detection: PCA-based nearest-neighbor distance with
    the Q3 + 1.5*(Q3 - Q1) bound; scimpute returns the column indices of outlier cells'
  repo: https://github.com/Vivianstats/scImpute
- name: parallel
  role: Parallel computation across multiple cores (ncores) to reduce scImpute's computation
    time
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:f1592c5b857e3a8203305b5a37ad11c2c80859e471e8fc4052d577e9abb5e629
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - parallel
  - scImpute
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scrna-seq-outlier-cell-detection

## Summary

Detects outlier cells in a raw scRNA-seq count matrix using scImpute's PCA-based nearest-neighbor distance rule, so that aberrant cells are excluded from dropout imputation rather than distorting it.

## When to use

Use when a raw scRNA-seq count matrix (genes × cells) is being imputed with scImpute and aberrant cells should be flagged and excluded before dropout imputation.

## When NOT to use

- The input is already normalized, dimension-reduced, or a feature/cell-type table rather than a raw whole-genome count matrix — scImpute recommends applying it on the whole-genome count matrix with most genes present
- The goal is general-purpose outlier detection independent of dropout imputation — this rule is coupled to scImpute's imputation pipeline and serves to exclude outlier cells from imputation
- The number of cells is extremely large without a prior cell filtering step — computational efficiency is limited in that setting per the article's discussion

## Inputs

- Raw count matrix (genes × cells), e.g. raw_count.csv with genes as rows and cells as columns
- Number of principal components retained (leading PCs jointly explaining ≥ 40% of total variance)
- Number of cores for parallel computation (ncores, e.g. 10)

## Outputs

- Column indices of outlier cells (returned by scimpute and recorded, e.g. outlier_cells.csv)
- Imputed count matrix scimpute_count.csv with outlier cells excluded from imputation

## How to apply

Load the raw count matrix (rows = genes, columns = cells) and pass it to scimpute via the count_path argument (e.g. system.file("extdata", "raw_count.csv", package = "scImpute")). Apply PCA to the count matrix, retaining leading principal components that jointly explain at least 40% of total variance. In the retained PC space, compute for each cell j the Euclidean distance l_j to its nearest neighboring cell, and flag cell j as an outlier when l_j exceeds the boxplot upper bound Q3 + 1.5*(Q3 - Q1) of the nearest-neighbor distance distribution. Run scimpute with parallel computation (e.g. ncores = 10) — the function returns the column indices of outlier cells, which should be recorded (e.g. written to outlier_cells.csv). The rationale is that outlier cells would otherwise be treated as neighbors when borrowing information for dropout imputation, biasing estimates for nearby cells.

## Related tools

- **scImpute** (Implements the outlier detection: PCA-based nearest-neighbor distance with the Q3 + 1.5*(Q3 - Q1) bound; scimpute returns the column indices of outlier cells) — https://github.com/Vivianstats/scImpute
- **parallel** (Parallel computation across multiple cores (ncores) to reduce scImpute's computation time)

## Examples

```
scimpute(count_path = system.file("extdata", "raw_count.csv", package = "scImpute"), infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 2, ncores = 10)
```

## Evaluation signals

- outlier_cells.csv exists and its listed column indices correspond exactly to cells whose nearest-neighbor distances exceed Q3 + 1.5*(Q3 - Q1)
- Recompute nearest-neighbor distances in the retained PC space (PCs explaining ≥ 40% variance) and confirm the flagged indices match the boxplot-bound outliers
- scimpute returns a non-empty integer vector of column indices (or an empty vector when no cell exceeds the bound) consistent with the distance distribution
- scimpute_count.csv is created in out_dir with the same dimensions (genes × cells) as the input count matrix
- Outlier cell indices point to actual columns of the input count matrix (indices within 1:ncells)

## Limitations

- Imputation (and the embedded outlier exclusion) degrades when dropout rates in the raw data are severely high, as with droplet-based technologies — noted as a future direction
- When the number of cells is extremely large, a prior filtering step on cells (based on unspecified biological knowledge) is needed to improve computational efficiency
- Choice of parameter K (number of cell populations) is left to the user without a definitive selection procedure

## Evidence

- [abstract] scImpute also detects outlier cells and excludes them from imputation.: "scImpute also detects outlier cells and excludes them from imputation."
- [readme] outlier indices returned by scimpute: "This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir` to store the imputed count matrix."
- [methods] parallel computing recommendation: "We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time."
- [readme] whole-genome count matrix recommendation: [evidence span withheld — anchor status: external_doc]
