---
name: nontemporal-variance-decomposition
description: Use when benchmarking dimension reduction visualizations of temporal
  scRNA-seq data by measuring percent of non-temporal variance explained, e.g. reproducing
  Fig. 1c comparisons of TCM vs six tools across 11 time-course datasets.
when_to_use_negative:
- The single-cell dataset lacks a complete time index, or the time index is not correlated
  with the underlying dynamic expression pattern — TCM requires both
- The goal is immediate biological interpretation of cell clusters on the latent space
  — further pathway analysis is needed beyond what this decomposition provides
- The input is non-temporal single-cell data with no time-course labeling, so non-temporal
  variance cannot be distinguished from temporal variance
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: tcm
  role: Primary R implementation of TCM; fits the topographic model tcm(X, time.table
    = time.table) whose latent space is scored for non-temporal variance
  repo: https://github.com/gongx030/tcm
- name: devtools
  role: Installs tcm via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
- name: scDatasets
  role: Provides the temporal benchmark datasets (guo, guo2, loh, trapnell, etc.)
    with count matrices and time.table metadata
- name: SummarizedExperiment
  role: Container for the count assays and colData (time.table) loaded from scDatasets
- name: Rtsne
  role: 't-SNE comparison method: Rtsne(t(X.log))$Y on the scaled log-transformed
    counts'
- name: diffusionMap
  role: 'Diffusion map comparison method: diffuse(dist(t(X.log)))$X[, c(1, 2)]'
- name: Monocle
  role: Comparison trajectory/dimension reduction method benchmarked in Fig. 1c
- name: TSCAN
  role: Comparison trajectory/dimension reduction method benchmarked in Fig. 1c
- name: Wishbone
  role: Comparison trajectory/dimension reduction method benchmarked in Fig. 1c
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:666a771abcade31a7cb0a49402baffb440dda606aa4eee7774cd041de3ffc575
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gongx030/tcm
  tools:
  - Monocle
  - R
  - Rtsne
  - SummarizedExperiment
  - TCM
  - TSCAN
  - Wishbone
  - devtools
  - diffusionMap
  - scDatasets
  - tcm
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# nontemporal-variance-decomposition

## Summary

Quantifies the percent of variance explained by non-temporal factors in 2D latent spaces produced by dimension reduction methods (TCM, t-SNE, diffusion map, diffusion pseudotime, Wishbone, Monocle, TSCAN) on temporal single-cell datasets, to benchmark visualization quality against Fig. 1c of the TCM paper.

## When to use

Use when benchmarking dimension reduction visualizations of temporal scRNA-seq data by measuring percent of non-temporal variance explained, e.g. reproducing Fig. 1c comparisons of TCM vs six tools across 11 time-course datasets.

## When NOT to use

- The single-cell dataset lacks a complete time index, or the time index is not correlated with the underlying dynamic expression pattern — TCM requires both
- The goal is immediate biological interpretation of cell clusters on the latent space — further pathway analysis is needed beyond what this decomposition provides
- The input is non-temporal single-cell data with no time-course labeling, so non-temporal variance cannot be distinguished from temporal variance

## Inputs

- Gene-by-cell read count matrix X from temporal single-cell datasets (guo, guo2, petropoulos, deng, loh, trapnell, iPSC and remaining scDatasets/SRA PRJNA438778 datasets)
- Cell-by-time-point binary time.table matrix giving each cell's timestamp
- Scaled log-transformed matrix X.log <- scale(log(X + 1)) used for t-SNE and diffusion map

## Outputs

- Per-method, per-dataset percent of variance explained by non-temporal factors
- Method-by-dataset summary table saved as fig1c_nontemporal_variance.csv
- 2D latent space plots of each method/dataset pair (e.g. plot(mf, pch = 21, bg = bg.cell, cex = 2.25))

## How to apply

Install tcm from github.com/gongx030/tcm via devtools::install_github, load tcm, scDatasets, SummarizedExperiment, Rtsne and diffusionMap. For each of the 11 temporal datasets (guo, guo2, petropoulos, deng, loh, trapnell, iPSC and the scDatasets/SRA PRJNA438778 datasets), load the gene-by-cell count matrix X and cell-by-time-point binary time.table, then filter with preprocess(X) (using min.expressed.gene = 0 where the vignette specifies, e.g. trapnell). Run TCM with mf <- tcm(X, time.table = time.table) under set.seed(1); where specified use a temporal-convolving landscape (landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)) and for large datasets stochastic optimization with batch.size = 2000 and mc.cores = 4. Run the six comparison methods on the scaled log-transformed matrix X.log <- scale(log(X + 1)), e.g. Rtsne(t(X.log))$Y for t-SNE and diffuse(dist(t(X.log)))$X[, c(1, 2)] for diffusion map, plus diffusion pseudotime, Wishbone, Monocle and TSCAN. For each method/dataset pair, visualize the 2D latent space (plot(mf, pch = 21, bg = bg.cell, cex = 2.25)) and compute the percent of variance explained by non-temporal factors, then assemble results into a method-by-dataset table (e.g. fig1c_nontemporal_variance.csv).

## Related tools

- **tcm** (Primary R implementation of TCM; fits the topographic model tcm(X, time.table = time.table) whose latent space is scored for non-temporal variance) — https://github.com/gongx030/tcm
- **devtools** (Installs tcm via install_github('gongx030/tcm')) — https://github.com/hadley/devtools
- **scDatasets** (Provides the temporal benchmark datasets (guo, guo2, loh, trapnell, etc.) with count matrices and time.table metadata)
- **SummarizedExperiment** (Container for the count assays and colData (time.table) loaded from scDatasets)
- **Rtsne** (t-SNE comparison method: Rtsne(t(X.log))$Y on the scaled log-transformed counts)
- **diffusionMap** (Diffusion map comparison method: diffuse(dist(t(X.log)))$X[, c(1, 2)])
- **Monocle** (Comparison trajectory/dimension reduction method benchmarked in Fig. 1c)
- **TSCAN** (Comparison trajectory/dimension reduction method benchmarked in Fig. 1c)
- **Wishbone** (Comparison trajectory/dimension reduction method benchmarked in Fig. 1c)

## Examples

```
library(devtools); install_github('gongx030/tcm'); library(tcm); library(scDatasets); data(guo); X <- preprocess(assays(guo)$count); set.seed(1); mf <- tcm(X, time.table = colData(guo)$time.table)
```

## Evaluation signals

- Reproduced table should show TCM attaining the highest percent of non-temporal variance on 9 of 11 datasets, matching Fig. 1c
- TCM's average percent of variance explained by non-temporal factors across the 11 temporal expression datasets should reach ~78.6%
- t-SNE and diffusion map baselines are computed on the same preprocessed matrix X.log <- scale(log(X + 1)) as a consistency check
- Each latent space is a 2D embedding (e.g. first two diffusion map coordinates X[, c(1, 2)]) and renders via plot(mf, pch = 21, bg = bg.cell, cex = 2.25) without error
- set.seed(1) is applied before tcm() so results are reproducible

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- TCM was validated on the 11 examined temporal datasets; the benchmark reports 9 of 11 datasets with highest non-temporal variance, so 2 datasets do not favor TCM

## Evidence

- [discussion] TCM increases the average percent of variance explained by non-temporal factors to 78.6% across 11 temporal expression datasets: "the average percent of variance explained by non-temporal factors on 11 examined temporal expression datasets increases to 78.6% by using TCM"
- [methods] TCM run command: "mf <- tcm(X, time.table = time.table)"
- [methods] log-transform scaling for comparison methods: "X.log <- scale(log(X + 1))"
- [methods] landscape specification: "ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)"
- [methods] preprocessing with gene filter: "X <- preprocess(X, min.expressed.gene = 0)"
- [readme] stochastic optimization for large datasets: "mf <- tcm(X, time.table = time.table, control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4))"
- [discussion] temporal index requirement: "TCM requires the scRNA-seq datasets with complete time index and the time index needs to be correlated with the underlying dynamic expression pattern."
