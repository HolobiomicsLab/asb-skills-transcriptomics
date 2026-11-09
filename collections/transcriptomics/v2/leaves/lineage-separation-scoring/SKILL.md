---
name: lineage-separation-scoring
description: Use when comparing dimension-reduction methods on temporal single-cell
  data to quantify which 2D embedding separates cells from distinct lineages across
  synchronized vs. asynchronized differentiation simulations.
when_to_use_negative:
- Data lack a complete cell time index, or the time index is not correlated with the
  underlying dynamic expression pattern (a stated prerequisite of TCM)
- The goal is immediate biological interpretation of cell clusters rather than visualization/separation
  benchmarking — the paper notes further pathway analysis is needed
- Data are not temporal scRNA-seq (no time-course structure or lineage labels), so
  no cross-lineage separation score can be defined
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: tcm
  role: R package providing sim.rnaseq.ts for simulating temporal scRNA-seq under
    sequential and asynchronized differentiation models, and tcm() for fitting the
    TCM embedding with optional stochastic variational inference
  repo: https://github.com/gongx030/tcm
- name: SummarizedExperiment
  role: Container from which the count matrix (assays()) and time.table (colData())
    are extracted
- name: Rtsne
  role: Baseline dimension reduction on scale(log(X + 1)) via Rtsne(t(X.log))$Y for
    the separation comparison
- name: diffusionMap
  role: Baseline dimension reduction via diffuse(dist(t(X.log)))$X[, c(1, 2)] for
    the separation comparison
- name: devtools
  role: Installs the tcm package via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:681ba2d757f1253d8c325bd9cd23c1a65d63b074ce5b2c6e473d80175bcb00f9
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# lineage-separation-scoring

## Summary

Benchmarks how well 2D latent-space embeddings (TCM, t-SNE, diffusion map) separate cells from different simulated lineages in temporal scRNA-seq data, by scoring cross-lineage separation at each simulated differentiation condition and aggregating results into lineage_separation_scores.csv.

## When to use

Use when comparing dimension-reduction methods on temporal single-cell data to quantify which 2D embedding best separates cells from distinct lineages across synchronized vs. asynchronized differentiation simulations.

## When NOT to use

- Data lack a complete cell time index, or the time index is not correlated with the underlying dynamic expression pattern (a stated prerequisite of TCM)
- The goal is immediate biological interpretation of cell clusters rather than visualization/separation benchmarking — the paper notes further pathway analysis is needed
- Data are not temporal scRNA-seq (no time-course structure or lineage labels), so no cross-lineage separation score can be defined

## Inputs

- Simulated temporal scRNA-seq count matrix (gene-by-cell, e.g. 2000 x 500) from tcm::sim.rnaseq.ts as a SummarizedExperiment assay
- Cell-by-time-point binary time.table (e.g. 500 x 5) from SummarizedExperiment colData
- Cell lineage labels from colData(sim)$lineage used for coloring and separation scoring
- Simulation conditions: differentiation model (sequential / forward-asynchronized / delayed-asynchronized), n.lineage = 2–10, n.time.points = 2–10

## Outputs

- 2D latent-space embeddings per method (TCM plot object, t-SNE Y matrix, diffusion map first two components)
- Cross-lineage cell-separation scores per method per simulation condition
- Aggregated lineage_separation_scores.csv covering TCM, t-SNE, and diffusion map across all conditions

## How to apply

Simulate temporal scRNA-seq benchmarks with tcm::sim.rnaseq.ts under sequential, forward-asynchronized, and delayed-asynchronized differentiation models (e.g., N = 2000 genes, M = 500 cells, n.lineage = 2–10, n.time.points = 2–10, exponential-decay dropout, fixed seed like set.seed(144)); extract the gene-by-cell count matrix via assays() and the cell-by-time-point binary matrix via colData() from the SummarizedExperiment object. Fit TCM with tcm(X, time.table = time.table), switching to stochastic variational inference (optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4) for large-scale data such as M = 10000. Preprocess baselines identically with scale(log(X + 1)), then embed with Rtsne(t(X.log))$Y and diffuse(dist(t(X.log)))$X[, c(1, 2)]. For each method's 2D latent space, compute a cross-lineage cell-separation score per condition, then aggregate scores for TCM, t-SNE, and diffusion map into lineage_separation_scores.csv. Correct application is confirmed when TCM attains the highest separation score under every simulated differentiation model, matching the paper's Fig. 2 comparison.

## Related tools

- **tcm** (R package providing sim.rnaseq.ts for simulating temporal scRNA-seq under sequential and asynchronized differentiation models, and tcm() for fitting the TCM embedding with optional stochastic variational inference) — https://github.com/gongx030/tcm
- **SummarizedExperiment** (Container from which the count matrix (assays()) and time.table (colData()) are extracted)
- **Rtsne** (Baseline dimension reduction on scale(log(X + 1)) via Rtsne(t(X.log))$Y for the separation comparison)
- **diffusionMap** (Baseline dimension reduction via diffuse(dist(t(X.log)))$X[, c(1, 2)] for the separation comparison)
- **devtools** (Installs the tcm package via install_github('gongx030/tcm')) — https://github.com/hadley/devtools

## Examples

```
sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5); mf <- tcm(X, time.table = time.table)
```

## Evaluation signals

- lineage_separation_scores.csv shows TCM with the best cross-lineage cell-separation score under every simulated condition, consistent with the paper's Fig. 2
- time.table is a binary cell-by-time-point matrix whose rows equal the number of columns of the count matrix (e.g. 500 cells x 5 time points)
- Baseline embeddings are computed on the identical scale(log(X + 1)) matrix used for both t-SNE and diffusion map, ensuring a fair comparison
- Large-scale runs (e.g. M = 10000) use optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4 and complete in a feasible time (~15 min on Intel Xeon 2.6GHz)
- Simulation is reproducible with a fixed seed (e.g. set.seed(144) for the 500-cell benchmark, set.seed(2) for the 10000-cell run)

## Limitations

- TCM requires scRNA-seq datasets with a complete time index that correlates with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is required
- For relatively easy simulated datasets, both t-SNE and diffusion map can separate lineages — TCM's advantage is improved lineage trajectories, so scores may be closer under easy conditions

## Evidence

- [methods] Simulation parameters: "sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5)"
- [methods] Baseline preprocessing and t-SNE embedding: [evidence span withheld — anchor status: not_found]
- [methods] Baseline diffusion map embedding: "y <- diffuse(dist(t(X.log)))$X[, c(1, 2)]"
- [readme] Stochastic inference for large scale: "optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4"
- [methods] Relative method performance on easy simulated data: "We find that for a relatively easy dataset, both t-SNE and diffusion map are able to separate three lineages. However, the visualization produced by TCM show much improved lineage trajectories."
