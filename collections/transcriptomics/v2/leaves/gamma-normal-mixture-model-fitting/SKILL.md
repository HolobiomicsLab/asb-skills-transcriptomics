---
name: gamma-normal-mixture-model-fitting
description: Use when a raw scRNA-seq count matrix (genes × cells) contains suspected
  dropout zeros and you need per-entry dropout probabilities to decide which values
  to impute rather than imputing everything.
when_to_use_negative:
- The matrix is not a raw whole-genome scRNA-seq count matrix — scImpute recommends
  whole-genome counts and warns that a gene filtering step is acceptable only if most
  genes are present, to ensure robust identification of dropouts
- Dropout rates in the raw data are severely high (e.g. droplet-based technologies),
  where the article notes imputation efficiency degrades and improvement is a future
  direction
- The goal is to alter all gene expression levels (e.g. smoothing all entries as MAGIC/SAVER
  do) rather than selectively imputing only likely dropouts
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scImpute
  role: Implements the Gamma-Normal mixture model EM fitting and dropout probability
    estimation via the scimpute() function; installed from GitHub
  repo: https://github.com/Vivianstats/scImpute
- name: parallel
  role: Provides multi-core parallel computation to speed up the per-gene EM fitting
    (ncores argument)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:e0ce1ce1215bf79a0effccae0ccecdd9f6a525722a2b14a911cd59d200ec0458
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - parallel
  - scImpute
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gamma-normal-mixture-model-fitting

## Summary

Fit a two-component Gamma–Normal mixture model per gene (within each cell subpopulation) via expectation-maximization to estimate posterior dropout probabilities for every gene–cell pair in an scRNA-seq count matrix, as implemented in scImpute.

## When to use

Use when a raw scRNA-seq count matrix (genes × cells) contains suspected dropout zeros and you need per-entry dropout probabilities to decide which values to impute rather than imputing everything.

## When NOT to use

- The matrix is not a raw whole-genome scRNA-seq count matrix — scImpute recommends whole-genome counts and warns that a gene filtering step is acceptable only if most genes are present, to ensure robust identification of dropouts
- Dropout rates in the raw data are severely high (e.g. droplet-based technologies), where the article notes imputation efficiency degrades and improvement is a future direction
- The goal is to alter all gene expression levels (e.g. smoothing all entries as MAGIC/SAVER do) rather than selectively imputing only likely dropouts

## Inputs

- raw scRNA-seq count matrix (CSV; rows = genes, columns = cells), e.g. raw_count.csv
- number of cell subpopulations (Kcluster)
- dropout probability threshold (drop_thre)
- number of cores for parallel computation (ncores)

## Outputs

- posterior dropout probability matrix d_ij (one row per gene, one column per cell, e.g. dropout_probabilities.csv)
- imputed count matrix with same dimensions as input (scimpute_count.csv)
- column indices of outlier cells returned by scimpute()

## How to apply

Load the raw count matrix (rows = genes, columns = cells, e.g. raw_count.csv) and normalize by each cell's library size followed by a log10 transformation. Partition cells into candidate subpopulations with Kcluster (e.g. Kcluster = 2) so each cell has neighbor groups for borrowing information. For each gene within each subpopulation, fit a two-component mixture model by EM — a Gamma component capturing dropout (zero/near-zero) values and a Normal component capturing true expression — using parallel computation (ncores, e.g. 10). Compute the posterior dropout probability d_ij for every gene–cell pair from the fitted mixture weights; only entries with dropout probability above drop_thre = 0.5 are targeted for imputation, which is the key rationale distinguishing scImpute from MAGIC and SAVER, which alter all expression levels.

## Related tools

- **scImpute** (Implements the Gamma-Normal mixture model EM fitting and dropout probability estimation via the scimpute() function; installed from GitHub) — https://github.com/Vivianstats/scImpute
- **parallel** (Provides multi-core parallel computation to speed up the per-gene EM fitting (ncores argument))

## Examples

```
scimpute(count_path = system.file("extdata", "raw_count.csv", package = "scImpute"), infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 2, ncores = 10)
```

## Evaluation signals

- The dropout probability matrix must have one row per gene and one column per cell, matching the input count matrix dimensions
- The set of entries with dropout probability > drop_thre = 0.5 must be identifiable as exactly the values scImpute targets for imputation
- The imputed output matrix (scimpute_count.csv) must have the same gene × cell dimensions as the raw input
- Entries with dropout probability ≤ drop_thre must remain unchanged — scImpute should not introduce new biases to the rest of the data
- Outlier cells must be detected and reported as column indices and excluded from imputation

## Limitations

- Choice of K (number of cell populations) is left to the user without a definitive selection procedure; it can be chosen based on the clustering result of the raw data and desired resolution
- Computation efficiency can be largely improved by a prior filtering step on cells based on biological knowledge, which must be supplied by the user
- Imputation efficiency degrades when dropout rates in the raw data are severely high (droplet-based technologies); improving this is a future direction
- scImpute detects outlier cells and excludes them from imputation, so those cells receive no corrected values

## Evidence

- [methods] Only the values that have dropout probability larger than drop_thre are imputed by scImpute.: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [abstract] scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data.: "scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data."
- [intro] Both MAGIC and SAVER would alter all gene expression levels including those unaffected by dropouts: "Both MAGIC and SAVER would alter all gene expression levels including those unaffected by dropouts"
- [methods] We highly recommend using parallel computing with scImpute, which will significantly reduce the computation time.: "We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time."
- [readme] It will output an imputed count matrix with the same dimension.: "Its only input is the raw count matrix with rows representing genes and columns representing cells. It will output an imputed count matrix with the same dimension."
