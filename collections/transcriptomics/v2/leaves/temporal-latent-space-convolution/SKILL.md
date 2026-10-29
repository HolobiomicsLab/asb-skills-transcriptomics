---
name: temporal-latent-space-convolution
description: Use when scRNA-seq data span multiple time points (a cell-by-time-point
  binary time.table is available) and you need one 2D latent space that preserves
  global developmental trajectories and separates subpopulations within each stage.
when_to_use_negative:
- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated
  with the underlying dynamic expression pattern
- The goal is immediate biological interpretation of cell clusters without further
  pathway analysis — TCM does not provide this directly
- Data are not temporal (no developmental time-course structure); TCM is a prototype-based
  temporal dimension reduction tool, not a generic embedding
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3308
tools:
- name: tcm
  role: R package implementing the TCM prototype-based generative model, sim.rnaseq.ts
    simulator, landscape() prototype grid, and preprocess() filter; installed via
    install_github('gongx030/tcm')
  repo: https://github.com/gongx030/tcm
- name: SummarizedExperiment
  role: Bioconductor container from which the count matrix (assays(sim)$count) and
    time.table (colData(sim)$time.table) are extracted
- name: devtools
  role: Provides install_github() to install the tcm package from GitHub
  repo: https://github.com/hadley/devtools
- name: R
  role: Runtime language for the TCM pipeline (R version 3.4.3 used in the paper)
- name: scDatasets
  role: Supplies case-study temporal datasets such as guo, guo2, loh, and trapnell
    with count assays and time.table metadata
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:2a313f7c9003144e346eadaf551fd88e3e6134c7cdb5a2e7b71a2c0f8db569a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gongx030/tcm
  tools:
  - R
  - SummarizedExperiment
  - TCM
  - devtools
  - scDatasets
  - tcm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# temporal-latent-space-convolution

## Summary

Reconstruct a single global developmental latent space from temporal single-cell RNA-seq by convolving time-point-specific latent spaces using the TCM prototype-based Bayesian generative model, placing early time points at the center and later ones at the periphery while preserving subpopulations within each time point.

## When to use

Use when scRNA-seq data span multiple time points (a cell-by-time-point binary time.table is available) and you need one 2D latent space that preserves global developmental trajectories and separates subpopulations within each stage.

## When NOT to use

- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated with the underlying dynamic expression pattern
- The goal is immediate biological interpretation of cell clusters without further pathway analysis — TCM does not provide this directly
- Data are not temporal (no developmental time-course structure); TCM is a prototype-based temporal dimension reduction tool, not a generic embedding

## Inputs

- Gene-by-cell raw read count matrix X (e.g. 2000 genes by 500 cells from sim.rnaseq.ts)
- Cell-by-time-point binary time.table (e.g. 500 by 5) indicating the timestamp of each cell
- Optional landscape() prototype grid specification (temporal.convolving, K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)

## Outputs

- Fitted 2D cell latent-space coordinates (embedding.csv)
- Prototype positions per time-point sub-space (prototype_layout.csv), H = R×S prototypes per time point
- Trajectory visualization (plot(mf, ...)) showing cells from early time points at the center and later time points at the periphery

## How to apply

Simulate or load a temporal count matrix X (genes by cells) plus a binary time.table, e.g. sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5) under set.seed(144). Optionally define the radial prototype grid with landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2), which places early time points at the center and later ones at the periphery. Fit the TCM generative model — Poisson likelihood for read counts driven by prototype metagenes, Gamma priors on rates, and an RBF-kernel Gaussian-process prior over prototype positions — via variational EM (GTM-style responsibility updates) using tcm(X, time.table = time.table); for large datasets (e.g. 10,000 cells) switch to stochastic variational inference with control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4). Preprocess experimental counts with preprocess(X) (optionally min.expressed.gene = 0) before fitting. Export the fitted 2D cell coordinates (embedding.csv) and prototype positions (prototype_layout.csv), and validate that each per-time-point sub-space contains exactly H = R×S prototypes and that mean time index increases monotonically with radial distance from the center.

## Related tools

- **tcm** (R package implementing the TCM prototype-based generative model, sim.rnaseq.ts simulator, landscape() prototype grid, and preprocess() filter; installed via install_github('gongx030/tcm')) — https://github.com/gongx030/tcm
- **SummarizedExperiment** (Bioconductor container from which the count matrix (assays(sim)$count) and time.table (colData(sim)$time.table) are extracted)
- **devtools** (Provides install_github() to install the tcm package from GitHub) — https://github.com/hadley/devtools
- **R** (Runtime language for the TCM pipeline (R version 3.4.3 used in the paper))
- **scDatasets** (Supplies case-study temporal datasets such as guo, guo2, loh, and trapnell with count assays and time.table metadata)

## Examples

```
library(tcm); set.seed(144); sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5); mf <- tcm(assays(sim)$count, time.table = colData(sim)$time.table)
```

## Evaluation signals

- Exported embedding.csv and prototype_layout.csv exist and are valid CSV
- Each per-time-point latent space contains exactly H = R×S prototypes (e.g. 15×15 with default landscape parameters)
- Mean time index increases monotonically with radial distance from the center of the latent space
- On the simulated 5-lineage dataset, TCM visualization shows much improved lineage trajectories compared with t-SNE and diffusion map run on scale(log(X + 1))
- Cell subpopulations within each time point remain distinguishable, consistent with the paper's report that TCM increases the average percent of variance explained by non-temporal factors to 78.6% across 11 datasets

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- Spatial expression pattern and other-omics integration are proposed future extensions but not implemented
- Large datasets require stochastic variational inference (~15 mins for 10,000 cells on 4 CPU cores of an Intel Xeon 2.6GHz machine)

## Evidence

- [abstract] TCM reconstructs the global developmental trajectory by convolving time-point-specific latent spaces: "Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point"
- [methods] Temporal convolving landscape specification places prototypes radially: "ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)"
- [methods] TCM model fitting interface: "mf <- tcm(X, time.table = time.table, control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4))"
- [readme] Simulated dataset generation grounded in README: "sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5)"
- [results] Superior visualization over t-SNE and diffusion map: "We find that for a relatively easy dataset, both t-SNE and diffusion map are able to separate three lineages. However, the visualization produced by TCM show much improved lineage trajectories."
- [readme] Preprocessing requirement for experimental counts: "X <- preprocess(X, min.expressed.gene = 0)"
