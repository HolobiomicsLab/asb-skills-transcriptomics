---
name: subpopulation-annotation-from-latent-space
description: Use when a temporal scRNA-seq dataset (cell-by-time-point time.table
  available) has been fitted with a TCM latent map and cells at a terminal time point
  must be split into subpopulations and annotated with known marker genes.
when_to_use_negative:
- The scRNA-seq data lack a complete time index, or the time index is not correlated
  with the underlying dynamic expression pattern — TCM requires this and the temporal
  convolving landscape cannot be built.
- Only a single time point exists with no time.table, so no temporal convolving structure
  or day-level restriction is possible.
- Subpopulations must be defined without prior knowledge of marker genes — TCM provides
  no immediate biological interpretation of clusters and downstream pathway analysis
  is needed.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0202
tools:
- name: tcm
  role: Fits the TCM landscape via tcm() with temporal.convolving landscape and 'backward'
    initialization, and provides plot(mf) for the 2D latent map used to delimit subpopulations
  repo: github.com/gongx030/tcm
- name: scDatasets
  role: Supplies the iPSC (human iPSC-CM differentiation) dataset used for day-60
    subpopulation annotation
- name: SummarizedExperiment
  role: Provides assays()/colData() access to the count matrix and time.table needed
    as tcm inputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:84d295635ee81937b4683aef12b20cfedb86bf7640791f8b531a8bf7e522277a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SummarizedExperiment
  - scDatasets
  - tcm
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# subpopulation-annotation-from-latent-space

## Summary

Annotates subpopulations of cells within a terminal time point by reading cell positions on a fitted TCM 2D latent map and validating each cluster against marker-gene expression (e.g. TNNT2/MYL2, NPPA/NPPB, NR2F2, CDH11 in the hiPSC-to-cardiomyocyte day-60 data). TCM's temporal convolving landscape separates within-time-point subpopulations that are indistinguishable on t-SNE or diffusion map.

## When to use

Use when a temporal scRNA-seq dataset (cell-by-time-point time.table available) has been fitted with a TCM latent map and cells at a terminal time point must be split into subpopulations and annotated with known marker genes.

## When NOT to use

- The scRNA-seq data lack a complete time index, or the time index is not correlated with the underlying dynamic expression pattern — TCM requires this and the temporal convolving landscape cannot be built.
- Only a single time point exists with no time.table, so no temporal convolving structure or day-level restriction is possible.
- Subpopulations must be defined without prior knowledge of marker genes — TCM provides no immediate biological interpretation of clusters and downstream pathway analysis is needed.

## Inputs

- Gene-by-cell scRNA-seq read count matrix (e.g. the iPSC dataset from the scDatasets package, accessed via assays())
- Cell-by-time-point binary time.table matrix (from colData) indexing each cell's time point
- Fitted TCM model object (mf) with 2D latent coordinates
- List of marker genes for subpopulation validation (e.g. TNNT2, MYL2, NPPA, NPPB, NR2F2, CDH11)

## Outputs

- Assignment table (CSV) with one row per terminal-time-point cell carrying a C1–C4 subpopulation label
- Marker-expression matrix (CSV) of log-transformed counts (log(X + 1)) for the validation marker set
- 2D latent map plot colored by subpopulation assignment and marker expression via num2color

## How to apply

Preprocess the gene-by-cell count matrix with preprocess(X, min.expressed.gene = 0), then build the landscape with landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2). With set.seed(1), fit the model via tcm(X, time.table = time.table, ls = ls, init = list(method = 'backward', update.beta = TRUE)) and plot the 2D latent map. Extract the latent coordinates, restrict to the terminal time point (day-60 cells in the hiPSC dataset), and assign cells to subpopulations C1–C4 based on the map's structural clusters. Verify identities by building a marker-expression matrix for TNNT2, MYL2, NPPA, NPPB, NR2F2 and CDH11 using num2color(log(X + 1)) scaling, expecting TNNT2/MYL2 in C1–C2, NPPA/NPPB in C1, NR2F2 in C3 and CDH11 in C4 (Fig. 4d pattern). Write the assignment table and marker-expression matrix to CSV files.

## Related tools

- **tcm** (Fits the TCM landscape via tcm() with temporal.convolving landscape and 'backward' initialization, and provides plot(mf) for the 2D latent map used to delimit subpopulations) — github.com/gongx030/tcm
- **scDatasets** (Supplies the iPSC (human iPSC-CM differentiation) dataset used for day-60 subpopulation annotation)
- **SummarizedExperiment** (Provides assays()/colData() access to the count matrix and time.table needed as tcm inputs)

## Examples

```
set.seed(1); ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2); mf <- tcm(X, time.table = time.table, ls = ls, init = list(method = 'backward', update.beta = TRUE))
```

## Evaluation signals

- The assignment CSV contains exactly one row per day-60 cell, each carrying a valid C1–C4 label.
- The marker-expression matrix shows TNNT2/MYL2 expression in C1–C2, NPPA/NPPB enrichment in C1, NR2F2 in C3 and CDH11 in C4, reproducing the reported Fig. 4d pattern.
- Model fit is reproducible: set.seed(1) with init = list(method = 'backward', update.beta = TRUE) yields the same four-cluster structure on the latent map.
- Landscape parameters match the paper's specification: K = 15, n.prototype = 15, n.circle = 10, n.prev = 2 with type = 'temporal.convolving'.
- Marker values are computed with the log(X + 1) / num2color scaling before plotting, not raw counts.

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index must be correlated with the underlying dynamic expression pattern.
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed.
- Subpopulations within each time point are usually indistinguishable on t-SNE or diffusion map latent spaces, so those views cannot be substituted for the TCM map when annotating within-time-point clusters.

## Evidence

- [methods] ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2): "ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)"
- [discussion] Downstream analysis such as trajectory inference, cell clustering and differential expression analysis could be readily performed on the latent space produced by TCM.: "cell clustering13,23 and differential expression analysis14 could be readily performed on the latent space produced by TCM."
- [abstract] Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point: "Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point"
- [readme] ls <- tcm(X, time.table = time.table, ls = ls) with explicit landscape and preprocess(min.expressed.gene = 0) in README case study: "X <- preprocess(X, min.expressed.gene = 0)"
- [readme] Marker expression visualization via num2color log-scaled counts (TNNT2 example): "plot(mf, pch = 21, bg = num2color(log(X['ENSG00000118194', ] + 1)), cex = 1.25, main = 'TNNT2')"
