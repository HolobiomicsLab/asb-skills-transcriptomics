---
name: single-cell-dropout-probability-estimation
description: Use when an scRNA-seq raw count matrix contains dropout-driven false
  zeros and you need to estimate which gene-cell entries are affected by dropout before
  imputation, rather than altering all expression values.
when_to_use_negative:
- If the goal is to impute every entry in the matrix regardless of dropout status
  — MAGIC and SAVER alter all gene expression levels including those unaffected by
  dropouts, which scImpute deliberately avoids
- If cell type labels or cluster structure are unknown and no reasonable Kcluster
  can be chosen, since the number of cell populations is left to the user without
  a definitive selection procedure
- If the raw data come from droplet-based protocols with severely high dropout rates,
  where imputation efficiency is noted as degrading
tools:
- name: scImpute
  role: Fits the per-gene Gamma-Normal mixture model by EM to compute posterior dropout
    probabilities and imputes only entries above drop_thre
  repo: https://github.com/Vivianstats/scImpute
- name: parallel
  role: Provides multi-core parallel computation (ncores = 10) to speed up mixture
    model fitting across genes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:c540c57e41b8332fb7c2b2f3d8dca1d9ed34c5aaf8732e6d9268df360f89171a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - parallel
  - scImpute
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-dropout-probability-estimation

## Summary

Estimate per-gene, per-cell dropout probabilities in scRNA-seq data by fitting a two-component Gamma-Normal mixture model per gene within each cell subpopulation via expectation-maximization, as implemented in scImpute. Only entries whose posterior dropout probability exceeds a threshold (e.g. drop_thre = 0.5) are targeted for imputation.

## When to use

Use when an scRNA-seq raw count matrix contains dropout-driven false zeros and you need to estimate which gene-cell entries are affected by dropout before imputation, rather than altering all expression values.

## When NOT to use

- If the goal is to impute every entry in the matrix regardless of dropout status — MAGIC and SAVER alter all gene expression levels including those unaffected by dropouts, which scImpute deliberately avoids
- If cell type labels or cluster structure are unknown and no reasonable Kcluster can be chosen, since the number of cell populations is left to the user without a definitive selection procedure
- If the raw data come from droplet-based protocols with severely high dropout rates, where imputation efficiency is noted as degrading

## Inputs

- Raw scRNA-seq read/UMI count matrix as CSV (rows = genes, columns = cells), e.g. raw_count.csv passed via count_path
- Number of cell subpopulations (Kcluster) for candidate neighbor grouping
- Dropout probability threshold (drop_thre, e.g. 0.5)
- Number of cores for parallel computation (ncores)

## Outputs

- Per gene-cell dropout probability matrix (one row per gene, one column per cell, e.g. dropout_probabilities.csv)
- Imputed count matrix with the same dimension as the input (scimpute_count.csv)
- Column indices of outlier cells excluded from imputation

## How to apply

Load the raw count matrix (rows = genes, columns = cells, e.g. raw_count.csv via count_path) and normalize counts by each cell's library size with a log10 transformation. Partition cells into candidate subpopulations using Kcluster (e.g. Kcluster = 2) to define neighbor groups for each cell. For each gene within each subpopulation, fit a two-component mixture model — a Gamma component for dropout (zero/near-zero) values and a Normal component for true expression — by expectation-maximization, using parallel computation (ncores = 10) to reduce runtime. Compute the posterior dropout probability for every gene-cell pair from the fitted mixture weights, and apply the drop_thre threshold (default 0.5) so that only values with dropout probability larger than drop_thre are imputed. scImpute also detects outlier cells and excludes them from imputation, returning their column indices.

## Related tools

- **scImpute** (Fits the per-gene Gamma-Normal mixture model by EM to compute posterior dropout probabilities and imputes only entries above drop_thre) — https://github.com/Vivianstats/scImpute
- **parallel** (Provides multi-core parallel computation (ncores = 10) to speed up mixture model fitting across genes)

## Examples

```
scimpute(count_path = system.file("extdata", "raw_count.csv", package = "scImpute"), infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 2, ncores = 10)
```

## Evaluation signals

- The dropout probability matrix has one row per gene and one column per cell, matching the input count matrix dimensions
- The set of entries with dropout probability > drop_thre = 0.5 is exactly the set scImpute targets for imputation
- The imputed output matrix (scimpute_count.csv) has the same dimension as the input raw count matrix
- Non-targeted entries (dropout probability <= drop_thre) remain unchanged, so no new biases are introduced into the rest of the data
- The function returns column indices of detected outlier cells that were excluded from imputation

## Limitations

- Imputation efficiency degrades when dropout rates in the raw data are severely high, e.g. droplet-based technologies; this is noted as a future direction
- The choice of K (number of cell populations) is left to the user without a definitive selection procedure; it can be chosen based on the clustering result of the raw data and the desired resolution
- Computational efficiency depends on a prior filtering step on cells based on biological knowledge, which is not specified
- Models accounting for gene similarities may need additional prior knowledge on similar genes to assist modeling
- No software version for scImpute is specified in the article

## Evidence

- [abstract] Selectively impute only high-dropout-probability entries: "scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data."
- [methods] drop_thre threshold governs which values are imputed: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [readme] Input/output format of the scimpute function: "Its only input is the raw count matrix with rows representing genes and columns representing cells. It will output an imputed count matrix with the same dimension."
- [methods] Parallel computation recommended: "We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time."
