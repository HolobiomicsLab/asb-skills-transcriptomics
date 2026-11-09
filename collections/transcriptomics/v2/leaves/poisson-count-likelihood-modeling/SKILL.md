---
name: poisson-count-likelihood-modeling
description: Use when fitting a latent-space model to raw scRNA-seq count matrices
  with per-cell time labels, so the generative likelihood matches integer counts (Poisson)
  rather than Gaussian assumptions on log-transformed data.
when_to_use_negative:
- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated
  with the underlying dynamic expression pattern
- The goal is a purely Gaussian latent-variable model on log-transformed data (e.g.
  scaling log(X+1) then t-SNE/diffusion maps) rather than count-based likelihood fitting
- You need immediate biological interpretation of cell clusters without additional
  pathway analysis
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: tcm
  role: Implements the Poisson/Gamma/GP prototype-based generative model and variational
    EM fitting (tcm() function with optional stochastic optimization)
  repo: https://github.com/gongx030/tcm
- name: SummarizedExperiment
  role: Container for the simulated/loaded count assay (assays(sim)$count) and time.table
    colData consumed by tcm()
  repo: https://github.com/gongx030/tcm
- name: devtools
  role: Installs the tcm package from GitHub via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:37979d04f80e2f05f8bc766602efa869a8f2443348ec0618d89aa79ded07ca69
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# poisson-count-likelihood-modeling

## Summary

Prototype-based Bayesian generative modeling of temporal single-cell RNA-seq counts: read counts are generated from prototype metagenes under a Poisson likelihood with Gamma priors on rates and an RBF-kernel Gaussian-process prior over prototype positions, fit by variational EM as implemented in the TCM R package.

## When to use

Use when fitting a latent-space model to raw scRNA-seq count matrices with per-cell time labels, so the generative likelihood matches integer counts (Poisson) rather than Gaussian assumptions on log-transformed data.

## When NOT to use

- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated with the underlying dynamic expression pattern
- The goal is a purely Gaussian latent-variable model on log-transformed data (e.g. scaling log(X+1) then t-SNE/diffusion maps) rather than count-based likelihood fitting
- You need immediate biological interpretation of cell clusters without additional pathway analysis

## Inputs

- Gene-by-cell raw read count matrix X (integer counts, e.g. 2000 genes × 500 cells from sim.rnaseq.ts)
- Cell-by-time-point binary matrix time.table (e.g. 500 × 5) assigning each cell a timestamp
- Landscape specification object from landscape(type = 'temporal.convolving', ...)

## Outputs

- Fitted 2D latent-space cell coordinates (e.g. exported to embedding.csv)
- Prototype positions on the convolved latent space (e.g. prototype_layout.csv)
- Per-time-point latent spaces each containing H = R×S prototypes

## How to apply

Simulate or load a genes-by-cells count matrix X (e.g. sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5) under set.seed(144)) and a cell-by-time-point binary time.table. Define the latent grid via landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2), which convolves time-point-specific latent spaces into one map with early time points at the center and later ones at the periphery; each per-time-point latent space contains H = R×S prototypes (here 15×15). Implement the generative model as a Poisson likelihood for read counts driven by prototype metagenes, with Gamma priors on rates and an RBF-kernel Gaussian-process prior over prototype positions, then fit by variational EM (GTN/GTM-style responsibility updates) using tcm(X, time.table = time.table). For large datasets (e.g. 2,000 genes × 10,000 cells) switch to stochastic variational inference with list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4), which takes ~15 min on an Intel Xeon 2.6GHz machine. Export fitted 2D cell coordinates and prototype positions and validate radial ordering of time points.

## Related tools

- **tcm** (Implements the Poisson/Gamma/GP prototype-based generative model and variational EM fitting (tcm() function with optional stochastic optimization)) — https://github.com/gongx030/tcm
- **SummarizedExperiment** (Container for the simulated/loaded count assay (assays(sim)$count) and time.table colData consumed by tcm()) — https://github.com/gongx030/tcm
- **devtools** (Installs the tcm package from GitHub via install_github('gongx030/tcm')) — https://github.com/hadley/devtools

## Examples

```
mf <- tcm(X, time.table = time.table)
```

## Evaluation signals

- Each per-time-point latent space contains exactly H = R×S prototypes (e.g. 15×15 for K = 15, n.prototype = 15)
- Mean time index increases monotonically with radial distance from the latent-space center (early time points near center, later at periphery)
- Fitted cell embeddings and prototype positions export as valid CSV (embedding.csv, prototype_layout.csv)
- Qualitative check: TCM visualization shows much improved lineage trajectories versus t-SNE and diffusion map on the simulated 5-lineage dataset
- On large simulated data, stochastic variational inference (batch.size = 2000, mc.cores = 4) converges in roughly ~15 minutes on an Intel Xeon 2.6GHz computer

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis will need to be conducted
- Extension to spatial expression patterns and other-omics data is proposed but not implemented

## Evidence

- [methods] Simulation of temporal scRNA-seq data using sim.rnaseq.ts: "sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5)"
- [methods] Landscape specification for temporal convolving: "ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)"
- [methods] Stochastic variational inference optimization: "mf <- tcm(X, time.table = time.table, control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4))"
- [abstract] TCM preserves global trajectories and identifies subpopulations: "Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point"
- [readme] README: simulated data with dropout noise model: "The single cell data are sampled across five time points following a sequentail differentiation model. The dropout noise was added using an exponential decay model."
- [results] TCM visualization improves lineage trajectories: "However, the visualization produced by TCM show much improved lineage trajectories."
