---
name: temporal-trajectory-visualization
description: Use when input is a temporal scRNA-seq count matrix with a complete cell-by-time-point
  binary time.table and the goal is to preserve lineage trajectories and resolve subpopulations
  within stages that t-SNE or diffusion maps crowd together.
when_to_use_negative:
- The scRNA-seq dataset lacks a complete time index or the time index is not correlated
  with the underlying dynamic expression pattern
- The data are single-condition (non-temporal) expression profiles where no time.table
  can be constructed
- Immediate biological interpretation of clusters is required without additional pathway
  analysis — TCM itself does not provide it
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: tcm
  role: Core R package implementing TCM model fitting (tcm()), preprocessing (preprocess()),
    landscape specification, and 2D visualization of temporal single-cell data
  repo: https://github.com/gongx030/tcm
- name: devtools
  role: Used via install_github to install the tcm package from GitHub
  repo: https://github.com/hadley/devtools
- name: SummarizedExperiment
  role: Provides assays()$count and colData()$time.table accessors for the input count
    matrix and time annotations
- name: scDatasets
  role: Supplies exemplar temporal datasets (guo, guo2, loh, trapnell, iPSC) for loading
    X and time.table
- name: Rtsne
  role: Baseline t-SNE visualization for comparison against TCM on the same log-transformed
    data
- name: diffusionMap
  role: Baseline diffusion-map visualization for comparison against TCM
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:c41e128740ca0d72d12565460f975b39241d35f072a7cf233164ef30abf6e66d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gongx030/tcm
  tools:
  - R
  - Rtsne
  - SummarizedExperiment
  - devtools
  - diffusionMap
  - scDatasets
  - tcm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# temporal-trajectory-visualization

## Summary

Apply Topographic Cell Map (TCM), a prototype-based dimension-reduction method for temporal single-cell RNA-seq, to visualize global developmental trajectories on a single 2D latent space while separating subpopulations within each time point.

## When to use

Use when input is a temporal scRNA-seq count matrix with a complete cell-by-time-point binary time.table and the goal is to preserve lineage trajectories and resolve subpopulations within stages that t-SNE or diffusion maps crowd together.

## When NOT to use

- The scRNA-seq dataset lacks a complete time index or the time index is not correlated with the underlying dynamic expression pattern
- The data are single-condition (non-temporal) expression profiles where no time.table can be constructed
- Immediate biological interpretation of clusters is required without additional pathway analysis — TCM itself does not provide it

## Inputs

- Gene-by-cell raw read count matrix (X) from temporal scRNA-seq or single-cell PCR data
- Binary cell-by-time-point time.table indicating each cell's timestamp
- SummarizedExperiment object (e.g. from scDatasets: guo, guo2, loh, trapnell, iPSC)
- Optional gene marker list for expression overlay (e.g. TNNT2, MYL2, NPPA, NPPB, NR2F2, CDH11)

## Outputs

- Fitted TCM model (mf) with 2D latent-space coordinates preserving global developmental trajectories
- 2D scatter plot of cells colored by time point or lineage (plot(mf, pch = 21, bg = bg.cell))
- Marker-expression overlays via num2color(log(X + 1)) on the latent map
- Cell-to-subpopulation assignment tables (e.g. CSV of C1–C4 labels per cell)

## How to apply

Install the R package with devtools::install_github('gongx030/tcm'), load tcm and SummarizedExperiment, then extract the gene-by-cell count matrix X and the binary cell-by-time-point time.table from a SummarizedExperiment (e.g. scDatasets data such as guo, guo2, loh, trapnell, or iPSC). Filter low-quality cells with preprocess(X), optionally with min.expressed.gene = 0 for sparse datasets like trapnell. Fit the model with tcm(X, time.table = time.table) after set.seed for reproducibility; for large datasets (e.g. 10,000 cells) use stochastic variational inference via control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4), and optionally supply a custom landscape via landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2) with init = list(method = 'backward', update.beta = TRUE). Visualize with plot(mf, pch = 21, bg = bg.cell) coloring cells by time point or lineage, and overlay marker expression using num2color(log(X[gene, ] + 1)). Downstream subpopulation clustering and differential marker checks (e.g. Hand1/Fgfr1 vs Gata1/Hba-x in mesoderm C1, TNNT2/MYL2/NPPA/NPPB, NR2F2, CDH11 in hiPSC-CM C1–C4) are then performed on the 2D latent coordinates.

## Related tools

- **tcm** (Core R package implementing TCM model fitting (tcm()), preprocessing (preprocess()), landscape specification, and 2D visualization of temporal single-cell data) — https://github.com/gongx030/tcm
- **devtools** (Used via install_github to install the tcm package from GitHub) — https://github.com/hadley/devtools
- **SummarizedExperiment** (Provides assays()$count and colData()$time.table accessors for the input count matrix and time annotations)
- **scDatasets** (Supplies exemplar temporal datasets (guo, guo2, loh, trapnell, iPSC) for loading X and time.table)
- **Rtsne** (Baseline t-SNE visualization for comparison against TCM on the same log-transformed data)
- **diffusionMap** (Baseline diffusion-map visualization for comparison against TCM)

## Examples

```
library(tcm); X <- preprocess(X); set.seed(1); mf <- tcm(X, time.table = time.table); plot(mf, pch = 21, bg = bg.cell)
```

## Evaluation signals

- The TCM plot preserves sequential ordering of time points/lineages better than t-SNE and diffusion map run on scale(log(X + 1)) of the same data
- Subpopulations within a single time point are visually separable on the TCM latent space (e.g. four C1–C4 clusters among E7.75 Cd41+ cells or day-60 hiPSC-CM cells)
- Marker overlays match expected biology: Hand1/Fgfr1 high and Gata1/Hba-x low in mesoderm C1; Zfpm1/Gata2 in C2–C3; Hbb-bh1/Hba-x high in C4; TNNT2/MYL2 in hiPSC-CM C1–C2, NPPA/NPPB in C1, NR2F2 in C3, CDH11 in C4
- Assignment CSV contains one row per target-stage cell with a valid subpopulation label
- Re-running with the same random seed (e.g. set.seed(1)) reproduces consistent cluster structure on the latent map

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- Large datasets require stochastic variational inference and multi-core execution (~15 mins for 10,000 cells on an Intel Xeon 2.6GHz computer)

## Evidence

- [abstract] Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point: "Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point"
- [readme] preprocess before fitting: "X <- preprocess(X)

set.seed(1) 
mf <- tcm(X, time.table = time.table)"
- [methods] landscape specification parameters: "ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)"
- [readme] stochastic variational inference for large data: "mf <- tcm(X, time.table = time.table, control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4))"
- [discussion] time-index requirement: "TCM requires the scRNA-seq datasets with complete time index and the time index needs to be correlated with the underlying dynamic expression pattern."
