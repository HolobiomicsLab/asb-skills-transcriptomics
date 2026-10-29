---
name: temporal-scrnaseq-dimension-reduction
description: Use when the input is a gene-by-cell count matrix from a temporal scRNA-seq
  time course (e.g., guo, guo2, loh, trapnell, iPSC, petropoulos, deng datasets) with
  a complete cell-by-time-point binary time.
when_to_use_negative:
- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated
  with the underlying dynamic expression pattern
- The task needs immediate biological interpretation of cell clusters on the latent
  space without further pathway analysis
- No temporal information is available — for non-temporal single-cell data, standard
  dimension reduction (t-SNE, diffusion map) without a time.table is more appropriate
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: tcm
  role: Core R package that preprocesses counts, fits the TCM model on the count matrix
    and time.table, and plots the 2D latent space
  repo: https://github.com/gongx030/tcm
- name: scDatasets
  role: Provides the temporal scRNA-seq datasets (guo, guo2, loh, trapnell, iPSC,
    petropoulos, deng) and simulated data via sim.rnaseq.ts
- name: SummarizedExperiment
  role: Accesses count matrices (assays()) and cell metadata/time tables (colData())
    from the dataset objects
- name: devtools
  role: Installs the tcm package from GitHub via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
- name: Rtsne
  role: t-SNE baseline visualization on scale(log(X + 1)) inputs for comparison against
    TCM
- name: diffusionMap
  role: Diffusion map baseline via diffuse(dist(t(X.log)))$X[, c(1, 2)] for comparison
    against TCM
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:61ffaafaa1d91d6799c218107258d15da1809544bb8f1d0cafe914c580299abc
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
  - devtools
  - diffusionMap
  - diptest
  - scDatasets
  - tcm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# temporal-scrnaseq-dimension-reduction

## Summary

Apply TCM (Topographic Cell Map), a prototype-based dimension reduction method, to visualize temporal single-cell RNA-seq data on a single 2D latent space that preserves global developmental trajectories while resolving subpopulations within each time point.

## When to use

Use when the input is a gene-by-cell count matrix from a temporal scRNA-seq time course (e.g., guo, guo2, loh, trapnell, iPSC, petropoulos, deng datasets) with a complete cell-by-time-point binary time.table, and the goal is trajectory-preserving 2D visualization with within-time-point subpopulation separation.

## When NOT to use

- The scRNA-seq dataset lacks a complete time index, or the time index is not correlated with the underlying dynamic expression pattern
- The task needs immediate biological interpretation of cell clusters on the latent space without further pathway analysis
- No temporal information is available — for non-temporal single-cell data, standard dimension reduction (t-SNE, diffusion map) without a time.table is more appropriate

## Inputs

- Gene-by-cell raw count matrix extracted via assays(dataset)$count from a SummarizedExperiment object (e.g., scDatasets datasets guo, guo2, loh, trapnell, iPSC, petropoulos, deng, or sim.rnaseq.ts output)
- Cell-by-time-point binary time.table indicating each cell's timestamp, from colData(dataset)$time.table
- Per-cell metadata such as sample_alias, source.name, developmental.stage, or lineage for coloring cells

## Outputs

- Fitted TCM model object (mf) holding the 2D latent coordinates for all cells
- 2D scatter plot (e.g., tcm_loh.png, tcm_guo2.png) colored by sample labels, lineage, or time point
- Per-cell 2D latent coordinate CSV (e.g., tcm_loh_coordinates.csv) with one row per cell, two coordinate columns, and sample labels

## How to apply

Load the count matrix via SummarizedExperiment accessors (assays(dataset)$count) and the binary time.table from colData(dataset), then filter genes with preprocess(X) (optionally min.expressed.gene = 0 for sparser data such as trapnell). Fit the model with mf <- tcm(X, time.table = time.table) using the default landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2) settings, or use stochastic variational inference with control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4) for large datasets (e.g., 2,000 genes x 10,000 cells, ~15 min on Intel Xeon 2.6GHz). Set a seed (e.g., set.seed(1)) for reproducibility, assign per-cell colors from sample metadata (e.g., lab2name mapping of loh sample_alias, or gold/purple overrides for 17-week female and 19-week male PGCs in guo2), and plot with plot(mf, pch = 21, bg = bg.cell, cex = 2.25). Extract the 2D latent coordinates per cell for downstream trajectory inference, clustering, or differential expression analysis.

## Related tools

- **tcm** (Core R package that preprocesses counts, fits the TCM model on the count matrix and time.table, and plots the 2D latent space) — https://github.com/gongx030/tcm
- **scDatasets** (Provides the temporal scRNA-seq datasets (guo, guo2, loh, trapnell, iPSC, petropoulos, deng) and simulated data via sim.rnaseq.ts)
- **SummarizedExperiment** (Accesses count matrices (assays()) and cell metadata/time tables (colData()) from the dataset objects)
- **devtools** (Installs the tcm package from GitHub via install_github('gongx030/tcm')) — https://github.com/hadley/devtools
- **Rtsne** (t-SNE baseline visualization on scale(log(X + 1)) inputs for comparison against TCM)
- **diffusionMap** (Diffusion map baseline via diffuse(dist(t(X.log)))$X[, c(1, 2)] for comparison against TCM)

## Examples

```
library(tcm); library(SummarizedExperiment); data(loh); X <- preprocess(assays(loh)$count); mf <- tcm(X, time.table = colData(loh)$time.table); plot(mf, pch = 21, bg = bg.cell, cex = 2.25)
```

## Evaluation signals

- Output CSV contains one row per cell of the input dataset with exactly two latent-coordinate columns plus sample labels
- Cells from sequential differentiation stages (e.g., hESC(D0) through Cardiac Mesoderm(D3-D4) in the loh dataset) are laid out along continuous trajectories matching the published tcm_loh.png
- Known bifurcations are visible on the latent space, e.g., somite (cyan) vs cardiomyocyte (pink) mesoderm lineages in loh, or somatic-vs-PGC lineages with the female (17 week, gold) vs male (19 week, purple) PGC bifurcation in guo2
- Hartigan's dip test on the last-time-point cells' latent coordinates should yield a higher dip score than t-SNE, diffusion map, diffusion pseudotime, Wishbone, Monocle, and TSCAN baselines
- For large datasets, stochastic variational inference completes in the reported runtime (~15 mins for 2,000 genes x 10,000 cells with batch.size = 2000 and mc.cores = 4 on Intel Xeon 2.6GHz)

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed to elucidate the meaning of each trajectory
- Extension to incorporate spatial expression patterns and other-omics data is proposed but not implemented
- Baseline methods (t-SNE, diffusion map) render subpopulations within each time point indistinguishable because minor expression differences are dominated by temporal differences — comparisons must use the last-time-point dip statistic rather than qualitative visual inspection alone

## Evidence

- [readme] TCM fitting on temporal scRNA-seq data with default landscape settings: "mf <- tcm(X, time.table = time.table)"
- [methods] Stochastic variational inference configuration for large-scale datasets: "mf <- tcm(X, time.table = time.table, control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4))"
- [methods] Temporal convolving landscape specification with K = 15, 15 prototypes, 10 circles: "ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)"
- [discussion] Complete time index requirement for TCM: "TCM requires the scRNA-seq datasets with complete time index and the time index needs to be correlated with the underlying dynamic expression pattern."
- [results] Superior separation of last-time-point subpopulations measured by Hartigan's dip statistic: [evidence span withheld — anchor status: not_found]
- [intro] Loss of within-time-point resolution in baseline methods: "The subpopulations of cells within each time point are usually indistinguishable, due to minor expression differences compared with the more dominant temporal differences."
