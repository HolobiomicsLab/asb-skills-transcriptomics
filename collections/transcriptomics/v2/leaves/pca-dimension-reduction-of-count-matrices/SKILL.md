---
name: pca-dimension-reduction-of-count-matrices
description: Use when a raw gene-by-cell count matrix needs dimension reduction before
  distance- or similarity-based steps such as scImpute's outlier-cell detection via
  nearest-neighbor distances.
when_to_use_negative:
- The count matrix has already been dimension-reduced or normalized into a feature/embedding
  table — PCA on the raw whole-genome count matrix is what scImpute recommends, since
  a gene filtering step is acceptable only if most genes are present
- The goal is imputation of dropout values without the outlier-detection pre-step,
  and a different distance definition (not nearest-neighbor in PC space) is required
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scImpute
  role: Supplies the scimpute() function whose count_path input loads the raw count
    matrix and performs PCA-based dimension reduction internally before nearest-neighbor
    outlier detection
  repo: https://github.com/Vivianstats/scImpute
- name: parallel
  role: Parallel back-end for the PCA/distance computation across multiple cores (e.g.,
    ncores = 10) to cut runtime
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:50b5da3f00a344c55f386fa8975bf6ae1ed4899ce2d4ddb51bf5421ebf3d2ddf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - parallel
  - scImpute
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pca-dimension-reduction-of-count-matrices

## Summary

Reduce a raw scRNA-seq count matrix (genes × cells) to its leading principal components within the scImpute workflow, retaining PCs that jointly explain a set fraction of total variance so that cell-cell distances — e.g., nearest-neighbor distances used for outlier detection — are computed in a denoised low-dimensional space.

## When to use

Use when a raw gene-by-cell count matrix needs dimension reduction before distance- or similarity-based steps such as scImpute's outlier-cell detection via nearest-neighbor distances.

## When NOT to use

- The count matrix has already been dimension-reduced or normalized into a feature/embedding table — PCA on the raw whole-genome count matrix is what scImpute recommends, since a gene filtering step is acceptable only if most genes are present
- The goal is imputation of dropout values without the outlier-detection pre-step, and a different distance definition (not nearest-neighbor in PC space) is required

## Inputs

- Raw scRNA-seq count matrix (genes as rows, cells as columns), e.g., a CSV such as raw_count.csv from scImpute extdata
- Variance-explained threshold (e.g., 40% cumulative variance) for retaining leading principal components

## Outputs

- Reduced cell-coordinate matrix in the retained leading principal components
- Cell-by-cell Euclidean nearest-neighbor distances computed in PC space (input to outlier flagging)

## How to apply

Load the raw count matrix via scImpute's count_path argument (e.g., system.file("extdata", "raw_count.csv", package = "scImpute")), with rows as genes and columns as cells. Apply PCA to this matrix and retain only the leading principal components that jointly explain at least 40% of the total variance. In the retained PC space, compute Euclidean nearest-neighbor distances between cells (as done in scImpute's outlier detection), so distances reflect transcriptomic structure rather than dropout noise. Run with parallel computation (ncores = 10) since scImpute recommends parallel computing to significantly reduce computation time. Validate by confirming the retained PCs' cumulative variance ≥ 40% and that downstream distance-based flags are consistent with the PCA space used.

## Related tools

- **scImpute** (Supplies the scimpute() function whose count_path input loads the raw count matrix and performs PCA-based dimension reduction internally before nearest-neighbor outlier detection) — https://github.com/Vivianstats/scImpute
- **parallel** (Parallel back-end for the PCA/distance computation across multiple cores (e.g., ncores = 10) to cut runtime)

## Examples

```
R: outlier_idx <- scimpute(count_path=system.file("extdata","raw_count.csv",package="scImpute"), infile="csv", outfile="csv", out_dir="./", labeled=FALSE, drop_thre=0.5, Kcluster=2, ncores=10)
```

## Evaluation signals

- Cumulative variance explained by the retained leading PCs is ≥ 40% of the total variance of the count matrix
- The reduced representation has one row per cell and column count equal to the number of retained PCs
- Nearest-neighbor Euclidean distances l_j computed in PC space match recomputed distances from the same retained PCs
- Cells flagged as outliers (l_j > Q3 + 1.5*(Q3 - Q1) of nearest-neighbor distances) correspond exactly to the column indices returned by scimpute
- The scimpute run with ncores set produces scimpute_count.csv in out_dir and returns outlier cell column indices

## Limitations

- No software version for scImpute is specified, and the article does not state a fixed rule for the number of components — the 40% variance criterion comes from the workflow card, not the paper text
- Computation efficiency degrades for extremely large cell numbers; scImpute suggests a filtering step on cells based on biological knowledge, which is not specified in the article
- The parameter K (number of cell populations) is left to the user without a definitive selection procedure, which indirectly affects the downstream cell-similarity computation
- Imputation efficiency degrades when dropout rates in raw data are severely high, e.g., droplet-based technologies

## Evidence

- [methods] dimension reduction of scRNA-seq data: "dimension reduction of scRNA-seq data"
- [methods] We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time.: "We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time."
- [abstract] scImpute also detects outlier cells and excludes them from imputation.: "scImpute also detects outlier cells and excludes them from imputation."
- [methods] When the number of cells is extremely large, a filtering step on the cells can save the computation time of `scImpute`.: "When the number of cells is extremely large, a filtering step on the cells can save the computation time of `scImpute`."
- [readme] This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir`: "This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir`"
