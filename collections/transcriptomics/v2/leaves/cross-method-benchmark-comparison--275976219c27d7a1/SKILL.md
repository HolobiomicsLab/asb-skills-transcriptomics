---
name: cross-method-benchmark-comparison
description: Use when a new temporal single-cell embedding method must be benchmarked
  head-to-head against established dimension-reduction tools on the same gene-by-cell
  count matrices, with a quantitative per-dataset comparison metric and a method-by-dataset
  results table.
when_to_use_negative:
- The datasets lack a complete time index, or the time index is not correlated with
  the underlying dynamic expression pattern — TCM explicitly requires a complete,
  biologically meaningful time index
- The goal is biological interpretation of cell clusters rather than method comparison
  — TCM does not provide immediate biological interpretation and further pathway analysis
  is needed
- Data are not temporal single-cell expression matrices (no gene-by-cell counts plus
  cell-by-time-point annotation)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tcm
  role: 'Candidate method under benchmark: fits the TCM latent space from count matrix
    X and time.table, with optional temporal.convolving landscape and stochastic variational
    inference'
  repo: github.com/gongx030/tcm
- name: Rtsne
  role: 'Baseline dimension-reduction method: embeds the scaled log-transformed matrix
    via Rtsne(t(X.log))$Y'
- name: diffusionMap
  role: 'Baseline dimension-reduction method: embeds via diffuse(dist(t(X.log)))$X[,
    c(1, 2)]'
- name: scDatasets
  role: Provides the real temporal datasets (guo, guo2, loh, trapnell, etc.) with
    count assays and time.table colData
- name: devtools
  role: Installs the tcm package from GitHub via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
- name: SummarizedExperiment
  role: Container for count assays and colData extracted via assays() and colData()
    before fitting each method
- name: Monocle
  role: Baseline comparison tool in the six-tool benchmark of Fig. 1c
- name: TSCAN
  role: Baseline comparison tool in the six-tool benchmark of Fig. 1c
- name: Wishbone
  role: Baseline comparison tool in the six-tool benchmark of Fig. 1c
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:2059f3e8e592e8c86715027564923a476f2f581b97133c2a2b2a06100f08cee3
schema_version: 0.2.0
metadata:
  license_tier: open
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-method-benchmark-comparison

## Summary

A benchmarking procedure from the TCM (Topographic Cell Map) paper that compares a proposed dimension-reduction method against commonly used tools (t-SNE, diffusion map, diffusion pseudotime, Wishbone, Monocle, TSCAN) on identical temporal single-cell datasets, scoring each by the percent of variance explained by non-temporal factors or by cross-lineage cell separation.

## When to use

Use when a new temporal single-cell embedding method must be benchmarked head-to-head against established dimension-reduction tools on the same gene-by-cell count matrices, with a quantitative per-dataset comparison metric and a method-by-dataset results table.

## When NOT to use

- The datasets lack a complete time index, or the time index is not correlated with the underlying dynamic expression pattern — TCM explicitly requires a complete, biologically meaningful time index
- The goal is biological interpretation of cell clusters rather than method comparison — TCM does not provide immediate biological interpretation and further pathway analysis is needed
- Data are not temporal single-cell expression matrices (no gene-by-cell counts plus cell-by-time-point annotation)

## Inputs

- Gene-by-cell raw count matrices X from temporal single-cell datasets (e.g. scDatasets objects guo, guo2, petropoulos, deng, loh, trapnell, iPSC)
- Cell-by-time-point binary time.table matrices assigning each cell a timestamp
- Simulated temporal scRNA-seq data from tcm::sim.rnaseq.ts (N = 2000 genes, M = 500–10000 cells, n.lineage = 2–10, n.time.points = 2–10) under sequential, forward-async and delayed-async models

## Outputs

- Method-by-dataset table of percent variance explained by non-temporal factors (e.g. fig1c_nontemporal_variance.csv)
- Per-condition lineage separation scores for TCM, t-SNE and diffusion map (lineage_separation_scores.csv)
- 2D latent-space embeddings and plots per method/dataset pair (plot(mf, pch = 21, bg = bg.cell, cex = 2.25))

## How to apply

Install the candidate method (e.g. devtools::install_github('gongx030/tcm')) and load comparison packages (tcm, scDatasets, SummarizedExperiment, Rtsne, diffusionMap). For each dataset (guo, guo2, petropoulos, deng, loh, trapnell, iPSC, plus SRA PRJNA438778 datasets), load the gene-by-cell count matrix X and the cell-by-time-point binary time.table, and apply preprocess(X) (with min.expressed.gene = 0 where specified) so every method receives the identical filtered input. Fit the candidate with tcm(X, time.table = time.table) using set.seed(1), optionally with landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2) and init = list(method = 'backward', update.beta = TRUE); use stochastic optimization (optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4) for large data. Fit all baselines on the same shared representation: build X.log <- scale(log(X + 1)), embed with Rtsne(t(X.log))$Y and diffuse(dist(t(X.log)))$X[, c(1, 2)], plus diffusion pseudotime, Wishbone, Monocle and TSCAN. Score each method/dataset pair (percent of variance explained by non-temporal factors for the real-data benchmark, cross-lineage separation score for the sim.rnaseq.ts simulation benchmark), assemble a method-by-dataset table (e.g. fig1c_nontemporal_variance.csv), and validate against the expected outcome: TCM attains the highest non-temporal variance on 9 of 11 datasets with an average of ~78.6%, and the best lineage separation under all simulated differentiation models.

## Related tools

- **tcm** (Candidate method under benchmark: fits the TCM latent space from count matrix X and time.table, with optional temporal.convolving landscape and stochastic variational inference) — github.com/gongx030/tcm
- **Rtsne** (Baseline dimension-reduction method: embeds the scaled log-transformed matrix via Rtsne(t(X.log))$Y)
- **diffusionMap** (Baseline dimension-reduction method: embeds via diffuse(dist(t(X.log)))$X[, c(1, 2)])
- **scDatasets** (Provides the real temporal datasets (guo, guo2, loh, trapnell, etc.) with count assays and time.table colData)
- **devtools** (Installs the tcm package from GitHub via install_github('gongx030/tcm')) — https://github.com/hadley/devtools
- **SummarizedExperiment** (Container for count assays and colData extracted via assays() and colData() before fitting each method)
- **Monocle** (Baseline comparison tool in the six-tool benchmark of Fig. 1c)
- **TSCAN** (Baseline comparison tool in the six-tool benchmark of Fig. 1c)
- **Wishbone** (Baseline comparison tool in the six-tool benchmark of Fig. 1c)

## Examples

```
library(tcm); X <- preprocess(assays(guo)$count); set.seed(1); mf <- tcm(X, time.table = colData(guo)$time.table)
```

## Evaluation signals

- TCM achieves the highest percent of variance explained by non-temporal factors on 9 of 11 datasets, with an average of ~78.6%, matching Fig. 1c
- The simulation CSV shows TCM with the best cross-lineage cell separation under every simulated differentiation model (sequential, forward-async, delayed-async), consistent with Fig. 2
- Every method is evaluated on the identical preprocessed matrix (preprocess(X) applied once; baselines all use the shared X.log <- scale(log(X + 1)))
- Reproduced runs use set.seed(1) (real data) or set.seed(144) (simulations) so embeddings are reproducible
- The output table has one row per method and one column per dataset, covering all seven methods (TCM, t-SNE, diffusion map, diffusion pseudotime, Wishbone, Monocle, TSCAN) across 11 datasets

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- Large datasets (e.g. M = 10000 cells) require stochastic variational inference (~15 mins on an Intel Xeon 2.6GHz with batch.size = 2000 and mc.cores = 4)

## Evidence

- [discussion] Superior non-temporal variance across 11 datasets: "the average percent of variance explained by non-temporal factors on 11 examined temporal expression datasets increases to 78.6% by using TCM, suggesting a significant reduction of the crowding"
- [methods] Shared baseline preprocessing: "X.log <- scale(log(X + 1))"
- [methods] Stochastic optimization for large data: "mf <- tcm(X, time.table = time.table, control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4))"
- [readme] Baseline comparison behavior on simulations: "We find that for a relatively easy dataset, both t-SNE and diffusion map are able to separate three lineages. However, the visualization produced by TCM show much improved lineage trajectories."
- [abstract] Superiority claim over six methods: "demonstrating superior visualization performance over six commonly used methods"
