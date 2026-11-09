---
name: mesoderm-differentiation-lineage-mapping
description: Use when a temporal scRNA-seq dataset of hESC mesodermal differentiation
  (e.g., the 'loh' dataset from scDatasets) needs a 2D trajectory visualization that
  separates somite and cardiomyocyte lineages within each time point.
when_to_use_negative:
- The dataset lacks a complete time index, or the time index is not correlated with
  the underlying dynamic expression pattern — TCM explicitly requires both
- The goal is biological interpretation of clusters or trajectories — TCM does not
  provide immediate biological interpretation and further pathway analysis is needed
- The data are not temporal single-cell expression data over specified time periods
  (e.g., a static feature table or already-embedded coordinates)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0089
tools:
- name: tcm
  role: R package providing preprocess(), tcm() model fitting, and plot() of the fitted
    model's 2D latent space; installed via devtools install_github('gongx030/tcm')
  repo: github.com/gongx030/tcm
- name: scDatasets
  role: Supplies the loh dataset (hESC-derived mesodermal lineages, Loh et al.) via
    data(loh)
- name: SummarizedExperiment
  role: Provides assays() and colData() accessors used to extract the count matrix
    and time.table from the loh object
- name: devtools
  role: Used to install the tcm package from GitHub with install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:b23c12ac314a1c665396a3742ea40b19a61b6b02f97ca3a6460664bab9d8881f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SummarizedExperiment
  - devtools
  - scDatasets
  - tcm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# mesoderm-differentiation-lineage-mapping

## Summary

Reproduce the TCM (Topographic Cell Map) visualization of the somite vs. cardiomyocyte mesoderm bifurcation on the Loh hESC-derived mesoderm scRNA-seq dataset (days 0–4), demonstrating that TCM preserves global developmental trajectories while resolving lineage subpopulations on a single 2D latent space.

## When to use

Use when a temporal scRNA-seq dataset of hESC mesodermal differentiation (e.g., the 'loh' dataset from scDatasets) needs a 2D trajectory visualization that separates somite and cardiomyocyte lineages within each time point.

## When NOT to use

- The dataset lacks a complete time index, or the time index is not correlated with the underlying dynamic expression pattern — TCM explicitly requires both
- The goal is biological interpretation of clusters or trajectories — TCM does not provide immediate biological interpretation and further pathway analysis is needed
- The data are not temporal single-cell expression data over specified time periods (e.g., a static feature table or already-embedded coordinates)

## Inputs

- loh dataset from the scDatasets R package (SummarizedExperiment with assays(loh)$count gene-by-cell count matrix and colData(loh)$time.table cell-by-time-point binary matrix)
- colData(loh)[['sample_alias']] sample labels (H7hESC, H7_derived_APS, H7_derived_MPS, H7_derived_D2LtM, H7_derived_DLL1pPXM, H7_dreived_D2.25_Smtmrs, H7_derived_ESMT, H7_derived_D3GARPpCrdcM)

## Outputs

- tcm_loh.png — 2D TCM latent-space plot with cells colored by day-0-to-day-4 mesodermal sample label (pch = 21, cex = 2.25)
- tcm_loh_coordinates.csv — one row per loh cell with two latent-dimension columns plus the sample label

## How to apply

Load the loh dataset via library(scDatasets); data(loh), extracting the gene-by-cell count matrix with assays(loh)$count and the cell-by-time-point binary matrix with colData(loh)$time.table using SummarizedExperiment accessors. Filter the counts with the tcm package's preprocess(X) using default gene-filtering settings. Build per-cell background colors by mapping colData(loh)[['sample_alias']] through the lab2name vector (H7hESC→hESC(D0), H7_derived_APS→Anterior PS(D1), H7_derived_MPS→Mid PS(D1), H7_derived_D2LtM→Lateral Mesoderm(D2), H7_derived_DLL1pPXM→Paraxial Mesoderm(D2), H7_dreived_D2.25_Smtmrs→Somitomeres(D2.25), H7_derived_ESMT→Early Somite(D3), H7_derived_D3GARPpCrdcM→Cardiac Mesoderm(D3-D4)) and assigning rainbow(nlevels(cg)) colors. Fit the model with mf <- tcm(X, time.table = time.table) at default settings, then plot(mf, pch = 21, bg = bg.cell, cex = 2.25) so cells from sequential differentiation stages lay out along continuous mesodermal trajectories, with the bifurcation toward somites (outer circle, cyan) and cardiomyocytes (outer circle, pink) as in Fig. 3a. Export the per-cell 2D latent coordinates plus sample label to tcm_loh_coordinates.csv and the figure to tcm_loh.png.

## Related tools

- **tcm** (R package providing preprocess(), tcm() model fitting, and plot() of the fitted model's 2D latent space; installed via devtools install_github('gongx030/tcm')) — github.com/gongx030/tcm
- **scDatasets** (Supplies the loh dataset (hESC-derived mesodermal lineages, Loh et al.) via data(loh))
- **SummarizedExperiment** (Provides assays() and colData() accessors used to extract the count matrix and time.table from the loh object)
- **devtools** (Used to install the tcm package from GitHub with install_github('gongx030/tcm')) — https://github.com/hadley/devtools

## Examples

```
library(scDatasets); data(loh); X <- preprocess(assays(loh)$count); mf <- tcm(X, time.table = colData(loh)$time.table); plot(mf, pch = 21, bg = bg.cell, cex = 2.25)
```

## Evaluation signals

- tcm_loh_coordinates.csv exists with exactly one row per loh cell and two latent-coordinate columns plus a sample-label column
- The saved figure shows cells from sequential differentiation stages (hESC D0 through Cardiac Mesoderm D3-D4) laid out along continuous mesodermal trajectories, comparable to the published tcm_loh.png (Fig. 3a)
- The bifurcation of two major mesodermal lineages toward somites (outer circle, cyan dots) and cardiomyocytes (outer circle, pink dots) is visible in the outer circle of the plot
- preprocess(X) runs with default settings and returns a filtered count matrix compatible in cell count with time.table
- Per-cell colors correctly map all eight sample_alias levels through lab2name to rainbow(nlevels(cg)) colors

## Limitations

- TCM requires the scRNA-seq dataset to have a complete time index that is correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- TCM identifies subpopulations within time points but its latent space is intended for visualization, with downstream trajectory inference, clustering, and differential expression analysis performed separately on the latent space

## Evidence

- [abstract] TCM preserves global developmental trajectories on the Loh mesoderm dataset: "Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point"
- [methods] Model fitting step for temporal scRNA-seq: "mf <- tcm(X, time.table = time.table)"
- [discussion] Requirement on the time index: "TCM requires the scRNA-seq datasets with complete time index and the time index needs to be correlated with the underlying dynamic expression pattern."
- [readme] Loh case study data loading in README: "library(scDatasets)
data(loh)
X <- assays(loh)$count
time.table <- colData(loh)$time.table"
- [readme] Loh case study color assignment and plotting in README: "cg <- factor(cn, names(lab2name), lab2name)
col.cg <- rainbow(nlevels(cg))"
