---
name: hartigan-dip-statistic-multimodality-testing
description: Use when comparing dimension-reduction methods on temporal scRNA-seq
  data and you need a distribution-level statistic quantifying whether cells from
  a single time point separate into multiple subpopulations on a 2D latent space.
when_to_use_negative:
- The dataset has no complete time index, or the time index is not correlated with
  the underlying dynamic expression pattern — TCM's own scope limitation applies to
  this benchmark design
- The goal is biological interpretation of clusters rather than quantitative multimodality
  scoring — the article notes further pathway analysis is needed beyond the latent
  space
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: diptest
  role: R package implementing Hartigan's dip test, applied to last-time-point latent
    coordinate distributions to quantify subpopulation multimodality
- name: tcm
  role: TCM method being benchmarked; fit via tcm(X, time.table = time.table) to produce
    the latent space whose last-time-point dip scores are computed
  repo: https://github.com/gongx030/tcm
- name: scDatasets
  role: Supplies the temporal single-cell count matrices and time.table indicators
    (guo, guo2, loh, trapnell, iPSC, petropoulos, deng) used as benchmark inputs
- name: Rtsne
  role: t-SNE baseline; produces 2D coordinates via Rtsne(t(X.log))$Y on scale(log(X
    + 1)) inputs for dip-test comparison
- name: diffusionMap
  role: Diffusion map baseline; produces 2D coordinates via diffuse(dist(t(X.log)))$X[,
    c(1, 2)] for dip-test comparison
- name: SummarizedExperiment
  role: Provides assay/count matrix and colData accessors (assays()$count, colData()$time.table)
    for loading benchmark datasets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:a2a7a0dc0066647b1243b77333ef2780dbf689e5eca11835772d529d7b8b6bc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gongx030/tcm
  tools:
  - Monocle
  - R
  - Rtsne
  - SummarizedExperiment
  - TSCAN
  - Wishbone
  - diffusionMap
  - diptest
  - scDatasets
  - tcm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# hartigan-dip-statistic-multimodality-testing

## Summary

Applies Hartigan's dip test (R diptest package) to the 2D latent coordinates of cells from the last time point, across methods like TCM, t-SNE, diffusion map, diffusion pseudotime, Wishbone, Monocle and TSCAN, to quantify subpopulation multimodality and benchmark separation performance on 11 temporal single-cell expression datasets.

## When to use

Use when comparing dimension-reduction methods on temporal scRNA-seq data and you need a distribution-level statistic quantifying whether cells from a single time point separate into multiple subpopulations on a 2D latent space.

## When NOT to use

- The dataset has no complete time index, or the time index is not correlated with the underlying dynamic expression pattern — TCM's own scope limitation applies to this benchmark design
- The goal is biological interpretation of clusters rather than quantitative multimodality scoring — the article notes further pathway analysis is needed beyond the latent space

## Inputs

- Gene-by-cell raw count matrices from 11 temporal scRNA-seq datasets (guo, guo2, loh, trapnell, iPSC, petropoulos, deng, simulated via sim.rnaseq.ts)
- Cell-by-time-point binary time.table indicator matrices from scDatasets colData
- 2D latent coordinate matrices produced by each dimension-reduction method

## Outputs

- Hartigan's dip statistic per method-dataset pair for last-time-point cell coordinates
- Method-by-dataset dip score matrix written as dip_scores.csv
- Benchmark verdict that TCM consistently separates last-time-point subpopulations best

## How to apply

Load the 11 temporal count matrices and their cell-by-time-point binary time.table matrices via the scDatasets package (guo, guo2, loh, trapnell, iPSC, petropoulos, deng, plus sim.rnaseq.ts simulations). For t-SNE and diffusion map baselines, scale and log-transform raw counts with X.log <- scale(log(X + 1)); fit each method (tcm(X, time.table = time.table), Rtsne(t(X.log))$Y, diffuse(dist(t(X.log)))$X[, c(1, 2)], plus diffusion pseudotime, Wishbone, Monocle and TSCAN). For each method-dataset pair, extract the 2D latent coordinates of cells assigned to the last time point using the time.table binary indicator matrix, then run Hartigan's dip test from the diptest package on each coordinate distribution to measure multimodality. Tabulate dip scores into a method-by-dataset matrix; TCM should attain the highest dip score on all 11 datasets, consistent with the reported finding. Write the table as dip_scores.csv.

## Related tools

- **diptest** (R package implementing Hartigan's dip test, applied to last-time-point latent coordinate distributions to quantify subpopulation multimodality)
- **tcm** (TCM method being benchmarked; fit via tcm(X, time.table = time.table) to produce the latent space whose last-time-point dip scores are computed) — https://github.com/gongx030/tcm
- **scDatasets** (Supplies the temporal single-cell count matrices and time.table indicators (guo, guo2, loh, trapnell, iPSC, petropoulos, deng) used as benchmark inputs)
- **Rtsne** (t-SNE baseline; produces 2D coordinates via Rtsne(t(X.log))$Y on scale(log(X + 1)) inputs for dip-test comparison)
- **diffusionMap** (Diffusion map baseline; produces 2D coordinates via diffuse(dist(t(X.log)))$X[, c(1, 2)] for dip-test comparison)
- **SummarizedExperiment** (Provides assay/count matrix and colData accessors (assays()$count, colData()$time.table) for loading benchmark datasets)

## Examples

```
library(diptest); dip.t <- dip.test(y[time.table[, ncol(time.table)] == 1, 1])
```

## Evaluation signals

- TCM attains the highest Hartigan dip statistic on all 11 datasets, matching the reported finding of significantly better last-time-point subpopulation separation
- dip_scores.csv contains a complete method-by-dataset matrix covering TCM, t-SNE, diffusion map, diffusion pseudotime, Wishbone, Monocle and TSCAN across 11 datasets
- The subset of cells scored per dataset matches the rows with indicator 1 in the last column of the time.table binary matrix
- t-SNE and diffusion map baselines were computed on the same X.log <- scale(log(X + 1)) transformed input, ensuring a fair comparison

## Limitations

- Subpopulations within a time point are usually indistinguishable on t-SNE and diffusion map latent spaces because minor expression differences are dominated by temporal differences, which motivates but also complicates this benchmark
- TCM requires datasets with a complete time index that correlates with the underlying dynamic expression pattern
- The dip test scores only coordinate distributions of the final time point; it does not provide biological interpretation of the separated clusters

## Evidence

- [results] TCM has consistently significantly better performance in separating subpopulations from the final or last time point on all 11 datasets, as measured by Hartigan's Dip statistics: [evidence span withheld — anchor status: not_found]
- [methods] scaling and log-transform workflow step: "X.log <- scale(log(X + 1))"
- [readme] t-SNE baseline invocation: "y <- Rtsne(t(X.log))$Y"
- [readme] diffusion map baseline invocation: "y <- diffuse(dist(t(X.log)))$X[, c(1, 2)]"
- [intro] subpopulations indistinguishable on baselines rationale: "The subpopulations of cells within each time point are usually indistinguishable, due to minor expression differences compared with the more dominant temporal differences."
