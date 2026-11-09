---
name: single-cell-latent-space-clustering
description: Use when temporal scRNA-seq data must be visualized on a single 2D latent
  space and subpopulations within one time point (e.g. E7.75 Cd41+ cells) need to
  be identified and marker-validated.
when_to_use_negative:
- The scRNA-seq data lack a complete time index, or the time index is not correlated
  with the underlying dynamic expression pattern, since TCM requires both
- The goal is direct biological interpretation of clusters — TCM does not provide
  immediate biological interpretation and further pathway analysis is needed
- The input is not temporal single-cell data (e.g. a static feature table or bulk
  expression matrix), where TCM's temporal convolving landscape is inapplicable
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: tcm
  role: R package that fits the TCM model on temporal scRNA-seq counts and produces
    the 2D latent space used for clustering and subpopulation identification
  repo: https://github.com/gongx030/tcm
- name: SummarizedExperiment
  role: Provides the count matrix (assays)$count and time.table (colData) containers
    fed into preprocess() and tcm()
- name: devtools
  role: Used to install the tcm package from GitHub via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
- name: R
  role: Runtime environment (R version 3.4.3 reported in methods) in which the tcm
    workflow, plotting and marker overlays are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:80966a86c2b66d2343e086427754b700121192b16e7c98b94f54c82a4edfc52b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gongx030/tcm
  tools:
  - R
  - SummarizedExperiment
  - devtools
  - tcm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-latent-space-clustering

## Summary

Cluster cells within a developmental time point using the 2D latent space produced by TCM (Topographic Cell Map), a dimension-reduction method that preserves global temporal trajectories while separating subpopulations, then validate clusters against marker gene signatures.

## When to use

Use when temporal scRNA-seq data must be visualized on a single 2D latent space and subpopulations within one time point (e.g. E7.75 Cd41+ cells) need to be identified and marker-validated.

## When NOT to use

- The scRNA-seq data lack a complete time index, or the time index is not correlated with the underlying dynamic expression pattern, since TCM requires both
- The goal is direct biological interpretation of clusters — TCM does not provide immediate biological interpretation and further pathway analysis is needed
- The input is not temporal single-cell data (e.g. a static feature table or bulk expression matrix), where TCM's temporal convolving landscape is inapplicable

## Inputs

- Gene-by-cell scRNA-seq read count matrix (X, e.g. SummarizedExperiment assays)$count from a temporal dataset such as the E7.5/E7.75 mouse mesoderm data
- Cell-by-time-point binary time.table indicating each cell's time point
- Marker gene list per expected subpopulation (e.g. Hand1, Fgfr1, Gata1, Hba-x, Zfpm1, Gata2, Hbb-bh1)

## Outputs

- 2D TCM latent-space embedding of all cells (visualized via plot(mf, pch = 21, bg = bg.cell))
- Cell-to-subpopulation assignments for the target time point (e.g. cd41_subpopulation_assignments.csv with clusters C1-C4)
- Per-cluster marker-expression matrix / color overlays from num2color(log(X[gene, ] + 1))

## How to apply

Load the gene-by-cell count matrix (X) and the cell-by-time-point binary time.table, then filter low-quality cells with preprocess(X) before model fitting. Fit TCM with tcm(X, time.table = time.table) after setting a random seed for reproducibility, optionally specifying a landscape via landscape(type = 'temporal.convolving', ...) for control over K, n.prototype, n.circle and n.prev; for large datasets use stochastic variational inference with control = list(optimization.method = 'stochastic', batch.size = 2000, mc.cores = 4). Project all cells with plot(mf, pch = 21, bg = bg.cell), subset the cells of the target time point, and cluster them on their TCM latent-space coordinates into subpopulations. Extract marker expression per cluster as overlays using num2color(log(X[gene, ] + 1)); for example in the mouse mesoderm case, verify C1 shows Hand1/Fgfr1 up and Gata1/Hba-x down, C2-C3 show Zfpm1/Gata2, and C4 shows high Hbb-bh1/Hba-x. TCM was designed because subpopulations within a time point are usually indistinguishable on diffusion map and t-SNE latent spaces, where minor expression differences are dominated by temporal differences.

## Related tools

- **tcm** (R package that fits the TCM model on temporal scRNA-seq counts and produces the 2D latent space used for clustering and subpopulation identification) — https://github.com/gongx030/tcm
- **SummarizedExperiment** (Provides the count matrix (assays)$count and time.table (colData) containers fed into preprocess() and tcm())
- **devtools** (Used to install the tcm package from GitHub via install_github('gongx030/tcm')) — https://github.com/hadley/devtools
- **R** (Runtime environment (R version 3.4.3 reported in methods) in which the tcm workflow, plotting and marker overlays are executed)

## Examples

```
library(tcm); X <- preprocess(X); set.seed(1); mf <- tcm(X, time.table = time.table); plot(mf, pch = 21, bg = bg.cell)
```

## Evaluation signals

- Cluster marker directions match the expected Fig. 4b pattern: C1 high Hand1/Fgfr1 and low Gata1/Hba-x; C2-C3 positive for Zfpm1/Gata2; C4 high Hbb-bh1/Hba-x (qualitative check, no numeric threshold reported)
- Preprocessing ran successfully on the raw counts (X <- preprocess(X)) without dropping the target time-point cells
- plot(mf, pch = 21, bg = bg.cell) yields a 2D embedding in which cells from successive time points preserve the global developmental trajectory
- Subpopulations within the E7.75 Cd41+ subset are visually separable on the TCM latent space (the crowding problem is reduced)
- Results are reproducible after re-running with the same set.seed value

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- For very large datasets, stochastic variational inference with batching (~15 min for 10,000 cells on an Intel Xeon 2.6GHz with batch.size = 2000 and mc.cores = 4) may be needed
- Validation of the C1-C4 marker signatures is qualitative, since no numeric threshold is reported

## Evidence

- [intro] The subpopulations of cells within each time point are usually indistinguishable, due to minor expression differences compared with the more dominant temporal differences.: "The subpopulations of cells within each time point are usually indistinguishable, due to minor expression differences compared with the more dominant temporal differences."
- [discussion] TCM requires scRNA-seq datasets with a complete time index correlated with the underlying dynamic expression pattern.: "TCM requires the scRNA-seq datasets with complete time index and the time index needs to be correlated with the underlying dynamic expression pattern."
- [discussion] Downstream analysis such as trajectory inference, cell clustering and differential expression analysis could be readily performed on the latent space produced by TCM.: "Downstream analysis such as trajectory inference8,11, cell clustering13,23 and differential expression analysis14 could be readily performed on the latent space produced by TCM."
- [readme] README quick-start: preprocess counts, fit tcm, and plot the latent space.: "X <- preprocess(X); set.seed(1); mf <- tcm(X, time.table = time.table); plot(mf, pch = 21, bg = bg.cell, cex = 2.25)"
- [methods] Methods: scaling and log-transformation of raw read counts for comparison visualizations.: "X.log <- scale(log(X + 1))"
