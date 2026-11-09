---
name: nearest-neighbor-distance-computation
description: Use when preparing a raw count matrix (genes x cells, e.g. raw_count.csv)
  for scImpute dropout imputation and aberrant/low-quality cells must be detected
  and excluded from the imputation procedure.
when_to_use_negative:
- Input is not a raw read count matrix — the README recommends applying scImpute on
  the whole-genome count matrix before downstream analyses such as dimension reduction,
  normalization, or clustering.
- A gene-level pre-filter has removed most genes — 'A filtering step on genes is acceptable
  but most genes should be present to ensure robust identification of dropouts', which
  the outlier rule implicitly relies on.
- The goal is imputation of dropouts for all cells without exclusion — outlier detection
  here is specifically to exclude outlier cells from the imputation procedure, not
  a general QC report.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scImpute
  role: Provides the scimpute() function whose PCA plus nearest-neighbor IQR rule
    flags outlier cells and returns their column indices during dropout imputation
  repo: https://github.com/Vivianstats/scImpute
- name: parallel
  role: Used via ncores for parallel computation to significantly reduce the computation
    time of the outlier detection and imputation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:70b68ee65b4cbce30058718c4ea51a7b57720514400003ba5793aaa4d9aab7c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - parallel
  - scImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nearest-neighbor-distance-computation

## Summary

Flags outlier cells in a scRNA-seq count matrix by computing each cell's Euclidean distance to its nearest neighbor in a PCA-reduced space and applying the IQR-based bound Q3 + 1.5*(Q3 - Q1). This is the outlier-detection step scImpute uses to exclude aberrant cells before imputation of dropout values.

## When to use

Use when preparing a raw count matrix (genes x cells, e.g. raw_count.csv) for scImpute dropout imputation and aberrant/low-quality cells must be detected and excluded from the imputation procedure.

## When NOT to use

- Input is not a raw read count matrix — the README recommends applying scImpute on the whole-genome count matrix before downstream analyses such as dimension reduction, normalization, or clustering.
- A gene-level pre-filter has removed most genes — 'A filtering step on genes is acceptable but most genes should be present to ensure robust identification of dropouts', which the outlier rule implicitly relies on.
- The goal is imputation of dropouts for all cells without exclusion — outlier detection here is specifically to exclude outlier cells from the imputation procedure, not a general QC report.

## Inputs

- raw scRNA-seq count matrix (rows = genes, columns = cells), e.g. raw_count.csv loaded via count_path
- PCA-reduced cell representation retaining leading PCs explaining at least 40% of total variance

## Outputs

- column indices of outlier cells returned by the scimpute() function
- outlier_cells.csv listing the flagged outlier cell column indices

## How to apply

Load the raw count matrix via scimpute's count_path argument (system.file("extdata", "raw_count.csv", package = "scImpute")) in R, then apply PCA retaining only the leading principal components that jointly explain at least 40% of the total variance. In the retained PC space, compute for each cell j the Euclidean distance l_j to its nearest neighboring cell. Flag cell j as an outlier when l_j exceeds Q3 + 1.5*(Q3 - Q1) of the distribution of nearest-neighbor distances; the scimpute() call returns the column indices of these outlier cells. Use parallel computation (e.g. ncores = 10) since scImpute's authors recommend it to significantly reduce computation time, then write the flagged column indices to outlier_cells.csv.

## Related tools

- **scImpute** (Provides the scimpute() function whose PCA plus nearest-neighbor IQR rule flags outlier cells and returns their column indices during dropout imputation) — https://github.com/Vivianstats/scImpute
- **parallel** (Used via ncores for parallel computation to significantly reduce the computation time of the outlier detection and imputation)

## Examples

```
scimpute(count_path = system.file("extdata", "raw_count.csv", package = "scImpute"), infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 2, ncores = 10)
```

## Evaluation signals

- outlier_cells.csv exists in the output directory after the scimpute() call.
- Each column index listed in outlier_cells.csv corresponds exactly to a cell whose nearest-neighbor Euclidean distance in the retained PC space exceeds Q3 + 1.5*(Q3 - Q1) of the nearest-neighbor distance distribution.
- The retained principal components jointly explain at least 40% of the total variance before distance computation.
- The set of returned outlier indices is a subset of the input matrix's cell columns and is reproducible under the same PCA and distance settings.
- The imputed output scimpute_count.csv is produced with the same dimensions as the input count matrix, with outlier cells excluded from imputation.

## Limitations

- Choice of parameter K (number of cell populations) is left to the user without a definitive selection procedure, which affects the neighborhood structure used.
- No software version for scImpute is specified in the article.
- Computation efficiency depends on a prior filtering step on cells based on biological knowledge, which is not specified.

## Evidence

- [readme] This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir` to store the imputed count matrix.: "This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir` to store the imputed count matrix."
- [abstract] scImpute also detects outlier cells and excludes them from imputation.: "scImpute also detects outlier cells and excludes them from imputation."
- [methods] We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time.: "We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time."
