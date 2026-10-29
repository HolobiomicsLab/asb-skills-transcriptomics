---
name: dimension-reduction-visualization-comparison
description: Use when you must benchmark a dimension-reduction method for temporal
  single-cell data against t-SNE and diffusion map, evaluating how well each 2D embedding
  separates cells from different lineages across simulated differentiation conditions.
when_to_use_negative:
- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated
  with the underlying dynamic expression pattern (a stated requirement for TCM)
- The goal is immediate biological interpretation of cell clusters rather than visualization
  or trajectory separation — further pathway analysis is needed for interpretation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: tcm
  role: Fits TCM and simulates temporal scRNA-seq via sim.rnaseq.ts under sequential/async
    differentiation models
  repo: https://github.com/gongx030/tcm
- name: devtools
  role: Installs the tcm package from GitHub via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
- name: SummarizedExperiment
  role: Container from which the count matrix (assays) and time.table (colData) are
    extracted
- name: Rtsne
  role: 't-SNE baseline embedding of the log-scaled count matrix: Rtsne(t(X.log))$Y'
- name: diffusionMap
  role: 'Diffusion map baseline embedding: diffuse(dist(t(X.log)))$X[, c(1, 2)]'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:90ad4bfc0ce7e851884b9437b48fd3cf172ee348276ac65f8dfebab695cb9b48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gongx030/tcm
  tools:
  - R
  - Rtsne
  - SummarizedExperiment
  - TCM
  - devtools
  - diffusionMap
  - tcm
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# dimension-reduction-visualization-comparison

## Summary

Benchmark pipeline that simulates temporal scRNA-seq data under sequential and asynchronized differentiation models, fits Topographic Cell Map (TCM), and compares its 2D latent space against t-SNE and diffusion map using cross-lineage cell-separation scores.

## When to use

Use when you must benchmark a dimension-reduction method for temporal single-cell data against t-SNE and diffusion map, evaluating how well each 2D embedding separates cells from different lineages across simulated differentiation conditions.

## When NOT to use

- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated with the underlying dynamic expression pattern (a stated requirement for TCM)
- The goal is immediate biological interpretation of cell clusters rather than visualization or trajectory separation — further pathway analysis is needed for interpretation

## Inputs

- Simulated temporal scRNA-seq data from tcm::sim.rnaseq.ts (SummarizedExperiment with count assay and cell-by-time-point binary time.table)
- Parameters of the differentiation models: n.lineage (2–10), n.time.points (2–10), N = 2000 genes, M = 500–10000 cells
- Log-scaled gene-by-cell matrix X.log <- scale(log(X + 1)) for baseline methods

## Outputs

- 2D latent-space embeddings from TCM, t-SNE (Rtsne), and diffusion map (diffuse, first two components)
- Cross-lineage cell-separation scores per method per simulation condition
- Aggregated lineage_separation_scores.csv showing TCM best under every condition

## How to apply

Simulate temporal scRNA-seq count matrices with tcm::sim.rnaseq.ts (e.g. N = 2000 genes, M = 500 cells, n.lineage = 2–10, n.time.points = 2–10) under the sequential, forward-asynchronized, and delayed-asynchronized differentiation models, adding exponential-decay dropout noise and fixing the seed (e.g. set.seed(144)). Extract the gene-by-cell count matrix via assays(sim)$count and the cell-by-time-point binary matrix via colData(sim)$time.table from the SummarizedExperiment object, then fit TCM with tcm(X, time.table = time.table); for large runs (M = 10000) switch to stochastic variational inference with optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4. Preprocess baselines identically with X.log <- scale(log(X + 1)), embed with Rtsne(t(X.log))$Y and diffuse(dist(t(X.log)))$X[, c(1, 2)], and compute a cross-lineage cell-separation score for each 2D latent space at every simulated condition. Aggregate the scores into lineage_separation_scores.csv and verify TCM attains the highest score under all conditions, mirroring the paper's Fig. 2.

## Related tools

- **tcm** (Fits TCM and simulates temporal scRNA-seq via sim.rnaseq.ts under sequential/async differentiation models) — https://github.com/gongx030/tcm
- **devtools** (Installs the tcm package from GitHub via install_github('gongx030/tcm')) — https://github.com/hadley/devtools
- **SummarizedExperiment** (Container from which the count matrix (assays) and time.table (colData) are extracted)
- **Rtsne** (t-SNE baseline embedding of the log-scaled count matrix: Rtsne(t(X.log))$Y)
- **diffusionMap** (Diffusion map baseline embedding: diffuse(dist(t(X.log)))$X[, c(1, 2)])

## Examples

```
set.seed(144); sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5); mf <- tcm(assays(sim)$count, time.table = colData(sim)$time.table)
```

## Evaluation signals

- lineage_separation_scores.csv must show TCM attaining the highest cross-lineage cell-separation score under every simulated differentiation model (sequential, forward-async, delayed-async)
- TCM output must preserve global developmental trajectories while separating subpopulations within each time point, per the plot(mf, pch = 21, bg = bg.cell) visualization
- Baseline embeddings must be computed on the identically preprocessed matrix scale(log(X + 1)) to ensure a fair comparison
- Across 11 temporal expression datasets, TCM should increase the average percent of variance explained by non-temporal factors to 78.6%, indicating reduction of the crowding problem

## Limitations

- TCM requires scRNA-seq datasets with a complete time index that correlates with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; pathway analysis is required
- t-SNE and diffusion map can still separate lineages on easy datasets (e.g. three lineages); the TCM advantage is improved lineage trajectories
- Spatial expression patterns and other-omics integration are proposed but not implemented

## Evidence

- [methods] Simulation of temporal scRNA-seq data using sim.rnaseq.ts: "sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5)"
- [methods] Baseline preprocessing for t-SNE and diffusion map: "X.log <- scale(log(X + 1))"
- [methods] Baseline comparison finding: "We find that for a relatively easy dataset, both t-SNE and diffusion map are able to separate three lineages. However, the visualization produced by TCM show much improved"
- [readme] Large-scale stochastic variational inference settings: "mf <- tcm(X, time.table = time.table, control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4))"
- [discussion] Quantitative improvement of TCM over baselines: "the average percent of variance explained by non-temporal factors on 11 examined temporal expression datasets increases to 78.6% by using TCM"
- [methods] t-SNE embedding step: "y <- Rtsne(t(X.log))$Y"
