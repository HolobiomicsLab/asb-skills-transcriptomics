---
name: variational-em-approximate-inference
description: Use when fitting a probabilistic latent-space model (e.g., TCM) to temporal
  scRNA-seq count matrices with a cell-by-time-point binary time table, especially
  when exact inference of prototype-based generative models is intractable.
when_to_use_negative:
- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated
  with the underlying dynamic expression pattern
- The goal is immediate biological interpretation of cell clusters — TCM's latent
  space requires further pathway analysis and does not provide it directly
- The data are nontemporal single-cell data with no cell-by-time-point annotation,
  so the temporal convolving landscape cannot be defined
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: tcm
  role: R package implementing the prototype-based Bayesian generative model and variational
    EM / stochastic variational inference fitting via tcm(X, time.table = ...)
  repo: gongx030/tcm
- name: devtools
  role: Used to install the tcm package from GitHub via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
- name: SummarizedExperiment
  role: Provides the container classes from which the count matrix (assays(sim)$count)
    and time.table (colData(sim)$time.table) are extracted before fitting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:295b650a874278d9c01a30c8eaf5b0ac208db1cd0670361a6f4db8597ae7ecd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gongx030/tcm
  tools:
  - R
  - SummarizedExperiment
  - TCM
  - devtools
  - tcm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# variational-em-approximate-inference

## Summary

Fits TCM's prototype-based Bayesian generative model for temporal single-cell data by variational EM: Poisson read-count likelihood driven by radial prototype metagenes, Gamma priors on rates, and an RBF-kernel Gaussian-process prior over prototype positions, optimized via variational Bayes / GTM-style responsibility updates.

## When to use

Use when fitting a probabilistic latent-space model (e.g., TCM) to temporal scRNA-seq count matrices with a cell-by-time-point binary time table, especially when exact inference of prototype-based generative models is intractable.

## When NOT to use

- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated with the underlying dynamic expression pattern
- The goal is immediate biological interpretation of cell clusters — TCM's latent space requires further pathway analysis and does not provide it directly
- The data are nontemporal single-cell data with no cell-by-time-point annotation, so the temporal convolving landscape cannot be defined

## Inputs

- Gene-by-cell raw read count matrix (e.g., 2000 genes × 500 cells from sim.rnaseq.ts, or assays(sim)$count)
- Cell-by-time-point binary matrix time.table (e.g., 500 × 5) indicating each cell's timestamp
- Radial prototype landscape specification from landscape(type = 'temporal.convolving', ...)

## Outputs

- Fitted 2D cell embedding coordinates (embedding.csv)
- Prototype positions on the latent space (prototype_layout.csv)
- Fitted TCM model object (mf) plottable with plot(mf, pch = 21, bg = bg.cell)

## How to apply

Simulate or load a temporal count matrix (e.g., sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5) under set.seed(144)) plus its 500-by-5 binary time.table, then build the R×S radial prototype grid with landscape(type = 'temporal.convolving', time.points = 5, K = 15, n.prototype = 15, n.circle = 10, n.prev = 2), which places early time points at the center and later ones at the periphery. Fit the generative model with tcm(X, time.table = time.table): a Poisson likelihood for read counts driven by prototype metagenes, Gamma priors on the rates, and an RBF-kernel Gaussian-process prior over prototype positions, with variational Bayes / GTM-style responsibility updates alternating with parameter (M-step) updates. For large datasets (e.g., 2,000 genes × 10,000 cells), switch to stochastic variational inference via control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4), which takes ~15 min on an Intel Xeon 2.6 GHz machine. Export the fitted 2D cell coordinates and prototype layout, then verify that each per-time-point latent space contains H = R×S prototypes and that mean time index increases monotonically with radial distance from center.

## Related tools

- **tcm** (R package implementing the prototype-based Bayesian generative model and variational EM / stochastic variational inference fitting via tcm(X, time.table = ...)) — gongx030/tcm
- **devtools** (Used to install the tcm package from GitHub via install_github('gongx030/tcm')) — https://github.com/hadley/devtools
- **SummarizedExperiment** (Provides the container classes from which the count matrix (assays(sim)$count) and time.table (colData(sim)$time.table) are extracted before fitting)

## Examples

```
mf <- tcm(X, time.table = time.table, control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4))
```

## Evaluation signals

- Exported embedding.csv and prototype_layout.csv exist and parse as valid CSV files
- Each per-time-point latent space contains exactly H = R×S prototypes (e.g., H = 225 for K = 15, n.prototype = 15)
- Mean time index increases monotonically with radial distance from the center of the latent space (early stages central, later stages peripheral)
- Qualitative check: TCM visualization shows much improved lineage trajectories versus t-SNE and diffusion map on the same simulated dataset
- Large-scale run (M = 10,000) converges within ~15 minutes under stochastic variational inference with batch.size = 2000 and mc.cores = 4

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- Stochastic variational inference on large data (10,000 cells) takes ~15 minutes even on an Intel Xeon 2.6 GHz machine with 4 cores

## Evidence

- [methods] Variational EM fitting of TCM generative model: "mf <- tcm(X, time.table = time.table, control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4))"
- [methods] Landscape specification for temporal convolving: "ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)"
- [readme] Simulated temporal scRNA-seq dataset generation: "sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5)"
- [abstract] TCM preserves global trajectories over time course: "Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point"
- [discussion] Data requirement limitation: "TCM requires the scRNA-seq datasets with complete time index and the time index needs to be correlated with the underlying dynamic expression pattern."
- [readme] Stochastic variational inference runtime note: "We will use stochastic variational inference for optimizaing TCM, with batch size of 2,000 cells and using four CPU cores. It will take ~15 mins on an Intel Xeon 2.6GHz computer."
