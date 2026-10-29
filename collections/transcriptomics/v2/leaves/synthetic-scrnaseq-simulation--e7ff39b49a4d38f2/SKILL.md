---
name: synthetic-scrnaseq-simulation
description: Use when benchmarking temporal scRNA-seq visualization or trajectory
  methods on data with known ground truth — you need synthetic gene-by-cell count
  matrices with defined lineage and time-point labels.
when_to_use_negative:
- The input is real experimental scRNA-seq data with an observed time index — use
  TCM directly on the count matrix and time.table instead of simulating
- The analysis goal lacks a temporal or lineage component (e.g. static clustering
  of untimed single-cell data), since the simulator generates time-stamped multi-lineage
  trajectories by design
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2229
tools:
- name: tcm
  role: R package providing sim.rnaseq.ts for simulating temporal scRNA-seq data with
    dropout noise, and tcm() for the downstream method under test
  repo: https://github.com/gongx030/tcm
- name: SummarizedExperiment
  role: Container from which the simulated count matrix (assays) and time.table/lineage
    metadata (colData) are extracted
- name: Rtsne
  role: t-SNE baseline embedding of scale(log(X + 1)) for comparison of lineage separation
    on simulated data
- name: diffusionMap
  role: Diffusion map baseline (diffuse(dist(t(X.log)))$X[, c(1, 2)]) for comparison
    of lineage separation on simulated data
- name: devtools
  role: Used to install the tcm package from GitHub via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:aae60704763b1ff0609eb1f4560525f83a13c187332b3c240d2235f929f77bbe
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# synthetic-scrnaseq-simulation

## Summary

Simulate temporal single-cell RNA-seq count matrices with known lineages and time points using tcm::sim.rnaseq.ts, enabling controlled benchmarking of trajectory-visualization methods such as TCM, t-SNE, and diffusion map against ground-truth lineage structure.

## When to use

Use when benchmarking temporal scRNA-seq visualization or trajectory methods on data with known ground truth — you need synthetic gene-by-cell count matrices with defined lineage and time-point labels.

## When NOT to use

- The input is real experimental scRNA-seq data with an observed time index — use TCM directly on the count matrix and time.table instead of simulating
- The analysis goal lacks a temporal or lineage component (e.g. static clustering of untimed single-cell data), since the simulator generates time-stamped multi-lineage trajectories by design

## Inputs

- Simulation parameters: number of genes (N), cells (M), lineages (n.lineage), time points (n.time.points), random seed
- Differentiation model choice: sequential, forward-asynchronized, or delayed-asynchronized
- Dropout noise specification (exponential decay model)

## Outputs

- SummarizedExperiment object containing a gene-by-cell count matrix (assays(sim)$count), cell-by-time-point binary time.table (colData(sim)$time.table), and lineage labels
- Aggregated cross-lineage separation scores per method and condition (e.g. lineage_separation_scores.csv)

## How to apply

Call tcm::sim.rnaseq.ts under a fixed seed (e.g. set.seed(144)) with parameters such as N = 2000 genes, M = 500 cells, n.lineage = 2–10, and n.time.points = 2–10 to generate data under sequential, forward-asynchronized, or delayed-asynchronized differentiation models, with dropout noise added via an exponential decay model. Extract the gene-by-cell count matrix from assays(sim)$count and the cell-by-time-point binary matrix from colData(sim)$time.table of the returned SummarizedExperiment object. For large-scale variants (e.g. M = 10000), increase cells while keeping gene count fixed and adjust downstream fitting (stochastic variational inference with batch.size = 2000, mc.cores = 4). Feed the simulated matrix and time.table into each method under comparison (tcm(X, time.table), Rtsne on scale(log(X+1)), diffuse(dist(t(X.log)))) and compute cross-lineage cell-separation scores per condition.

## Related tools

- **tcm** (R package providing sim.rnaseq.ts for simulating temporal scRNA-seq data with dropout noise, and tcm() for the downstream method under test) — https://github.com/gongx030/tcm
- **SummarizedExperiment** (Container from which the simulated count matrix (assays) and time.table/lineage metadata (colData) are extracted)
- **Rtsne** (t-SNE baseline embedding of scale(log(X + 1)) for comparison of lineage separation on simulated data)
- **diffusionMap** (Diffusion map baseline (diffuse(dist(t(X.log)))$X[, c(1, 2)]) for comparison of lineage separation on simulated data)
- **devtools** (Used to install the tcm package from GitHub via install_github('gongx030/tcm')) — https://github.com/hadley/devtools

## Examples

```
set.seed(144); sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5); X <- assays(sim)$count; time.table <- colData(sim)$time.table
```

## Evaluation signals

- The simulated object exposes a 2000-gene by 500-cell count matrix and a 500-by-5 binary time.table when called with N = 2000, M = 500, n.time.points = 5
- Fixed seed reproducibility: set.seed(144) before sim.rnaseq.ts yields identical count matrices across runs
- TCM fitted on the simulated data attains the highest cross-lineage cell-separation score under all simulated differentiation models, consistent with the paper's Fig. 2 comparison
- On an easy simulated dataset (5 lineages, 5 time points), t-SNE and diffusion map separate lineages but the TCM latent space shows the radial early-center/late-periphery ordering
- Large-scale simulation (M = 10000) runs in ~15 mins on an Intel Xeon 2.6GHz computer when using stochastic variational inference

## Limitations

- The simulator implements only sequential, forward-asynchronized, and delayed-asynchronized differentiation models with exponential-decay dropout noise; other biological noise or asynchrony regimes are not covered
- Simulated ground truth may not reflect real-world data complexity, so benchmark conclusions are only qualitative agreement criteria for the visualization methods

## Evidence

- [methods] Simulation of temporal scRNA-seq data using sim.rnaseq.ts: "sim <- sim.rnaseq.ts(N = 2000, M = 500, n.lineage = 5, n.time.points = 5)"
- [readme] Dropout noise and differentiation models: "The single cell data are sampled across five time points following a sequentail differentiation model. The dropout noise was added using an exponential decay model."
- [methods] Baseline comparison outcome on simulated data: "We find that for a relatively easy dataset, both t-SNE and diffusion map are able to separate three lineages. However, the visualization produced by TCM show much improved lineage trajectories."
- [readme] Stochastic variational inference for large simulated datasets: "mf <- tcm(X, time.table = time.table, control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4))"
