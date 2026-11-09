---
name: iqr-based-anomaly-thresholding
description: Use when a raw gene-by-cell scRNA-seq count matrix may contain outlier
  cells that should be flagged and removed before dropout imputation, based on their
  distance to the nearest neighboring cell in PC space.
when_to_use_negative:
- The input is not a raw gene-by-cell count matrix (e.g. already normalized or imputed
  data) — scImpute is meant to be applied to the raw read count matrix before downstream
  analyses such as dimension reduction, normalization, clustering, or differential
  expression analysis.
- The analysis goal is to impute dropouts for every cell in a dataset with known,
  legitimate rare subpopulations — outlier exclusion would remove those cells from
  imputation rather than correct them.
- The task is clustering or annotating cell types (e.g. with SNN-Cliq, CIDR, or Seurat)
  rather than screening cells prior to dropout imputation.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scImpute
  role: Provides the scimpute() function that performs the nearest-neighbor/IQR outlier
    detection in PCA-reduced space and returns the column indices of outlier cells,
    excluding them from imputation
  repo: https://github.com/Vivianstats/scImpute
- name: parallel
  role: Used via the ncores argument of scimpute() to parallelize computation of cell
    distances and speed up outlier detection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:cd205570ac2e489578cd6ad5511cb42e3a08cb35a870dad97b5b2eb2f8b84eb2
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# iqr-based-anomaly-thresholding

## Summary

Detects outlier cells in a raw scRNA-seq count matrix by measuring each cell's nearest-neighbor Euclidean distance in a PCA-reduced space and thresholding at the Tukey bound Q3 + 1.5*(Q3 - Q1), so that scImpute can exclude these cells from dropout imputation.

## When to use

Use when a raw gene-by-cell scRNA-seq count matrix may contain outlier cells that should be flagged and removed before dropout imputation, based on their distance to the nearest neighboring cell in PC space.

## When NOT to use

- The input is not a raw gene-by-cell count matrix (e.g. already normalized or imputed data) — scImpute is meant to be applied to the raw read count matrix before downstream analyses such as dimension reduction, normalization, clustering, or differential expression analysis.
- The analysis goal is to impute dropouts for every cell in a dataset with known, legitimate rare subpopulations — outlier exclusion would remove those cells from imputation rather than correct them.
- The task is clustering or annotating cell types (e.g. with SNN-Cliq, CIDR, or Seurat) rather than screening cells prior to dropout imputation.

## Inputs

- raw scRNA-seq count matrix (CSV, genes as rows, cells as columns), e.g. raw_count.csv
- scimpute() parameters: infile/outfile format ('csv'), out_dir, labeled = FALSE, drop_thre = 0.5, Kcluster (number of cell subpopulations), ncores

## Outputs

- column indices of outlier cells (returned by scimpute(); written to outlier_cells.csv in the detection workflow)
- imputed count matrix scimpute_count.csv in out_dir (same dimensions as the input)

## How to apply

Load the raw count matrix (rows = genes, columns = cells) — e.g. system.file("extdata", "raw_count.csv", package = "scImpute") — and apply PCA, retaining only leading principal components that jointly explain at least 40% of the total variance. For each cell j in the retained PC space, compute the Euclidean distance l_j to its nearest neighboring cell. Flag cell j as an outlier when l_j exceeds Q3 + 1.5*(Q3 - Q1) computed over the distribution of all nearest-neighbor distances, and record the flagged cells' column indices. Run the detection inside scimpute() with parallel computation (e.g. ncores = 10) since the function returns the column indices of outlier cells while writing the imputed matrix scimpute_count.csv to out_dir. The rationale is that scImpute excludes detected outlier cells from imputation, so dropout probabilities are learned only from typical cells; validate by checking that the recorded indices correspond exactly to cells whose l_j exceeds the IQR bound.

## Related tools

- **scImpute** (Provides the scimpute() function that performs the nearest-neighbor/IQR outlier detection in PCA-reduced space and returns the column indices of outlier cells, excluding them from imputation) — https://github.com/Vivianstats/scImpute
- **parallel** (Used via the ncores argument of scimpute() to parallelize computation of cell distances and speed up outlier detection)

## Examples

```
scimpute(count_path = system.file("extdata", "raw_count.csv", package = "scImpute"), infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 2, ncores = 10)
```

## Evaluation signals

- outlier_cells.csv exists and its listed column indices correspond exactly to the cells whose nearest-neighbor distances l_j exceed Q3 + 1.5*(Q3 - Q1) of the nearest-neighbor distance distribution.
- The value returned by scimpute() is a set of integer column indices, each within [1, ncol] of the input count matrix.
- The number of flagged outlier cells is small relative to the total number of cells, consistent with scImpute's design of excluding atypical cells while imputing the rest.
- scimpute_count.csv is created in out_dir and has the same dimensions (genes x cells) as the raw input matrix.
- Retained principal components jointly explain at least 40% of the total variance before distances are computed.

## Limitations

- Detected outlier cells are excluded from imputation, not corrected — rare but biological cell states risk being flagged as outliers.
- When the number of cells is extremely large, a prior filtering step on cells (based on biological knowledge) is needed to keep computation tractable.
- The choice of parameter K (number of cell subpopulations) is left to the user without a definitive selection procedure, which can affect cell-similarity estimates underlying the distances.
- Imputation efficiency degrades when dropout rates in the raw data are severely high, e.g. in droplet microfluidics data, which is noted as a future direction.

## Evidence

- [abstract] scImpute also detects outlier cells and excludes them from imputation.: "scImpute also detects outlier cells and excludes them from imputation."
- [readme] This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir`: "This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir` to store the imputed count matrix."
- [methods] We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time.: "We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time."
