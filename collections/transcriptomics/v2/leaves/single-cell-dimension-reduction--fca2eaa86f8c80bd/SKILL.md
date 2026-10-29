---
name: single-cell-dimension-reduction
description: Use when you have a temporal scRNA-seq gene-by-cell count matrix with
  a cell-by-time-point binary time table and need a 2D visualization that preserves
  developmental trajectories and resolves intra-time-point subpopulations.
when_to_use_negative:
- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated
  with the underlying dynamic expression pattern
- The data are not temporal single-cell expression data and no time.table exists (e.g.
  a feature table or bulk omics matrix)
- You need immediate biological interpretation of cell clusters on the latent space
  — TCM requires further pathway analysis to elucidate the meaning of each trajectory
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: tcm
  role: Core R package implementing TCM dimension reduction (tcm(), preprocess(),
    landscape())
  repo: https://github.com/gongx030/tcm
- name: devtools
  role: Used to install tcm from GitHub via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
- name: scDatasets
  role: Provides temporal scRNA-seq datasets (guo, guo2, loh, trapnell, iPSC, etc.)
    as inputs
- name: SummarizedExperiment
  role: Used to access count assays and colData time tables from dataset objects
- name: Rtsne
  role: 'Comparison dimension reduction: t-SNE on the scaled log-transformed matrix
    via Rtsne(t(X.log))'
- name: diffusionMap
  role: 'Comparison dimension reduction: diffusion map via diffuse(dist(t(X.log)))$X[,
    c(1, 2)]'
- name: Monocle
  role: Commonly used comparison method in the benchmark of seven dimension reduction
    approaches
- name: TSCAN
  role: Commonly used comparison method in the benchmark of seven dimension reduction
    approaches
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:4cedcc23fb92c3cdd8f2ba4f6b73ac8c22ca25aba33e709ed016580f47784b85
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 21
---

# single-cell-dimension-reduction

## Summary

Reduces temporal single-cell expression count matrices to a 2D latent space using TCM (Topographic Cell Map), which preserves global developmental trajectories over time points while separating subpopulations within each time point, benchmarked against t-SNE, diffusion map, diffusion pseudotime, Wishbone, Monocle and TSCAN via percent of non-temporal variance explained.

## When to use

Use when you have a temporal scRNA-seq gene-by-cell count matrix with a cell-by-time-point binary time table and need a 2D visualization that preserves developmental trajectories and resolves intra-time-point subpopulations.

## When NOT to use

- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated with the underlying dynamic expression pattern
- The data are not temporal single-cell expression data and no time.table exists (e.g. a feature table or bulk omics matrix)
- You need immediate biological interpretation of cell clusters on the latent space — TCM requires further pathway analysis to elucidate the meaning of each trajectory

## Inputs

- Gene-by-cell raw read count matrix X (e.g. 2,000 genes by 500 cells) as assays(dataset)$count
- Cell-by-time-point binary time table (e.g. 500 cells by 5 time points) as colData(dataset)$time.table
- Datasets from scDatasets: guo, guo2, loh, trapnell, deng, petropoulos, iPSC
- Optional cell metadata for coloring (e.g. lineage, source.name, developmental stage)

## Outputs

- A TCM model object mf containing the 2D latent coordinates for each cell
- 2D scatter plots of the latent space (plot(mf, pch = 21, bg = bg.cell, cex = 2.25)) colored by time point or lineage
- Comparison latent spaces from Rtsne(t(X.log)) and diffuse(dist(t(X.log)))$X[, c(1, 2)]
- Per-method per-dataset percent of variance explained by non-temporal factors

## How to apply

Install the tcm package via devtools::install_github('gongx030/tcm') and load tcm, scDatasets and SummarizedExperiment. Load a dataset (e.g. data(guo), data(trapnell), data(iPSC)) to get a gene-by-cell count matrix X and a cell-by-time-point binary time.table, then filter genes with preprocess(X) or preprocess(X, min.expressed.gene = 0). Fit the model with set.seed(1) and mf <- tcm(X, time.table = time.table); for datasets needing an explicit topology use landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2) and init = list(method = 'backward', update.beta = TRUE). For large datasets (e.g. 10,000 cells) switch to stochastic variational inference with control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4). Render the 2D latent map with plot(mf, pch = 21, bg = bg.cell, cex = 2.25), coloring cells by time point or lineage; for comparison methods use t-SNE on Rtsne(t(X.log)) and diffusion map on diffuse(dist(t(X.log)))$X[, c(1, 2)] computed on the scaled log-transformed matrix X.log <- scale(log(X + 1)). Judge success by the percent of variance explained by non-temporal factors: TCM reached the highest value on 9 of 11 temporal datasets with an average of 78.6%, reducing the crowding problem where diffusion map and t-SNE make subpopulations within a time point indistinguishable.

## Related tools

- **tcm** (Core R package implementing TCM dimension reduction (tcm(), preprocess(), landscape())) — https://github.com/gongx030/tcm
- **devtools** (Used to install tcm from GitHub via install_github('gongx030/tcm')) — https://github.com/hadley/devtools
- **scDatasets** (Provides temporal scRNA-seq datasets (guo, guo2, loh, trapnell, iPSC, etc.) as inputs)
- **SummarizedExperiment** (Used to access count assays and colData time tables from dataset objects)
- **Rtsne** (Comparison dimension reduction: t-SNE on the scaled log-transformed matrix via Rtsne(t(X.log)))
- **diffusionMap** (Comparison dimension reduction: diffusion map via diffuse(dist(t(X.log)))$X[, c(1, 2)])
- **Monocle** (Commonly used comparison method in the benchmark of seven dimension reduction approaches)
- **TSCAN** (Commonly used comparison method in the benchmark of seven dimension reduction approaches)

## Examples

```
mf <- tcm(X, time.table = time.table); plot(mf, pch = 21, bg = bg.cell, cex = 2.25)
```

## Evaluation signals

- TCM attains the highest percent of variance explained by non-temporal factors on 9 of 11 temporal datasets, matching Fig. 1c
- Average percent of non-temporal variance explained across the 11 datasets is ~78.6% for TCM
- Latent plot shows cells ordered along global developmental trajectory with subpopulations separable within each time point (unlike diffusion map/t-SNE where temporal differences dominate)
- On the simulated 5-lineage dataset (sim.rnaseq.ts(N = 2000, M = 500)), TCM visualization shows improved lineage trajectories versus t-SNE and diffusion map
- Reproducibility check: fitting with set.seed(1) yields a stable 2D latent map

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- On large datasets, batch stochastic variational inference with batch.size = 2000 and mc.cores = 4 takes ~15 minutes on an Intel Xeon 2.6GHz computer
- Spatial expression patterns and other-omics integration are proposed future extensions but not implemented

## Evidence

- [discussion] TCM increases average non-temporal variance to 78.6%: "the average percent of variance explained by non-temporal factors on 11 examined temporal expression datasets increases to 78.6% by using TCM"
- [abstract] Superior visualization over six methods: "demonstrating superior visualization performance over six commonly used methods"
- [methods] Model fitting call: "mf <- tcm(X, time.table = time.table)"
- [methods] Preprocessing filter: "X <- preprocess(X, min.expressed.gene = 0)"
- [readme] t-SNE comparison invocation: "y <- Rtsne(t(X.log))$Y"
- [intro] Subpopulation indistinguishability rationale: "The subpopulations of cells within each time point are usually indistinguishable, due to minor expression differences compared with the more dominant temporal differences."
