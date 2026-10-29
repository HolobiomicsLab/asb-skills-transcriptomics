---
name: latent-space-coordinate-export
description: Use when a fitted TCM model (e.g. mf <- tcm(X, time.table)) must be archived
  as a per-cell 2D coordinate table and a reproducible scatter plot colored by cell
  metadata such as source.name or developmental stage.
when_to_use_negative:
- The dataset lacks a complete time index or the time index is not correlated with
  the underlying dynamic expression pattern — TCM requires both
- The goal is immediate biological interpretation of cell clusters; TCM does not provide
  this and further pathway analysis is needed
- Data are not temporal single-cell expression counts (e.g. already a feature table
  or non-scRNA-seq matrix)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2229
tools:
- name: tcm
  role: Fits the TCM dimension reduction model whose latent coordinates and plot method
    are exported; installed via devtools install_github('gongx030/tcm')
  repo: github.com/gongx030/tcm
- name: scDatasets
  role: Provides the guo2 dataset (count matrix via assays(guo2)$count and colData
    metadata) used as input
- name: SummarizedExperiment
  role: Accesses assay counts and colData metadata from the guo2 object
- name: devtools
  role: Installs the tcm package from GitHub via install_github
  repo: https://github.com/hadley/devtools
- name: R
  role: Execution environment for the tcm package and plotting/export scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:cde047f06da613bb19a90bf4a17da2932ea15609a998727fcb54fcc43c1cb498
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SummarizedExperiment
  - devtools
  - scDatasets
  - tcm
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# latent-space-coordinate-export

## Summary

Exports the fitted TCM model's two-dimensional latent coordinates for every cell to a CSV file and a labelled scatter plot (PNG), preserving the somatic/PGC lineage separation and female (17 week, gold) versus male (19 week, purple) PGC bifurcation seen for the guo2 dataset.

## When to use

Use when a fitted TCM model (e.g. mf <- tcm(X, time.table)) must be archived as a per-cell 2D coordinate table and a reproducible scatter plot colored by cell metadata such as source.name or developmental stage.

## When NOT to use

- The dataset lacks a complete time index or the time index is not correlated with the underlying dynamic expression pattern — TCM requires both
- The goal is immediate biological interpretation of cell clusters; TCM does not provide this and further pathway analysis is needed
- Data are not temporal single-cell expression counts (e.g. already a feature table or non-scRNA-seq matrix)

## Inputs

- Fitted TCM model object mf returned by tcm(X, time.table = time.table)
- Preprocessed gene-by-cell count matrix from assays(guo2)$count after preprocess(X)
- Cell metadata from colData(guo2): source.name, developmental.stage, gender, and time.table

## Outputs

- guo2_tcm_coordinates.csv — one row per cell with two 2D latent coordinate columns
- tcm_guo2.png — labelled scatter plot (pch = 21, bg = bg.cell, cex = 2.25) showing somatic and PGC lineages with the female/male PGC bifurcation

## How to apply

After fitting TCM with set.seed(1) via mf <- tcm(X, time.table = time.table) on the preprocessed count matrix (X <- preprocess(X)), assign per-cell plotting colors from cell metadata: derive bg.cell from colData(guo2)[['source.name']] via a rainbow palette, then override female PGCs at '17 week gestation' to 'gold' and male PGCs at '19 week gestation' to 'purple'. Plot the fitted model with plot(mf, pch = 21, bg = bg.cell, cex = 2.25), writing the scatter plot to a PNG (e.g. tcm_guo2.png). Export mf's 2D latent coordinates so that the output CSV contains one row per cell with two coordinate columns (e.g. guo2_tcm_coordinates.csv), enabling downstream trajectory inference, clustering, or differential expression analysis on the latent space as described in the article's discussion.

## Related tools

- **tcm** (Fits the TCM dimension reduction model whose latent coordinates and plot method are exported; installed via devtools install_github('gongx030/tcm')) — github.com/gongx030/tcm
- **scDatasets** (Provides the guo2 dataset (count matrix via assays(guo2)$count and colData metadata) used as input)
- **SummarizedExperiment** (Accesses assay counts and colData metadata from the guo2 object)
- **devtools** (Installs the tcm package from GitHub via install_github) — https://github.com/hadley/devtools
- **R** (Execution environment for the tcm package and plotting/export scripts)

## Examples

```
library(tcm); X <- preprocess(assays(guo2)$count); set.seed(1); mf <- tcm(X, time.table = colData(guo2)$time.table); plot(mf, pch = 21, bg = bg.cell, cex = 2.25)
```

## Evaluation signals

- Output CSV contains exactly one row per cell with two numeric coordinate columns
- PNG reproduces the reported figure: somatic and PGC lineages resolved, with the female (gold, 17 week) versus male (purple, 19 week) PGC bifurcation visible on the latent space
- Somatic cells and PGCs from weeks 7 to 11 are resolved on the 2D latent space, which other tools (t-SNE, diffusion map) did not achieve
- set.seed(1) is used before tcm() so the latent coordinates are reproducible
- Per-cell colors match colData metadata: gold only for female PGCs at 17 week gestation and purple only for male PGCs at 19 week gestation

## Limitations

- TCM requires scRNA-seq datasets with a complete time index, and the time index needs to be correlated with the underlying dynamic expression pattern
- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- The bifurcation of female (17 weeks) and male (19 weeks) PGCs is preserved by TCM but not by other visualization tools, so it should not be validated against t-SNE or diffusion map outputs

## Evidence

- [results] TCM clearly identified two major lineages of somatic cell and PGC development and a bifurcation of female (17 weeks) and male (19 weeks) PGCs: [evidence span withheld — anchor status: not_found]
- [readme] tcn model fitting and plot invocation from README: "set.seed(1)
mf <- tcm(X, time.table = time.table)
plot(mf, pch = 21, bg = bg.cell, cex = 2.25)"
- [readme] Female/male PGC color overrides: "bg.cell[colData(guo2)[['developmental.stage']] == '17 week gestation' & colData(guo2)[['gender']] == 'female' & colData(guo2)[['source.name']] == 'Primordial Germ Cells'] <- 'gold'"
- [discussion] Downstream analysis on latent space: "Downstream analysis such as trajectory inference8,11, cell clustering13,23 and differential expression analysis14 could be readily performed on the latent space produced by TCM."
- [discussion] TCM requires complete time index: "TCM requires the scRNA-seq datasets with complete time index and the time index needs to be correlated with the underlying dynamic expression pattern."
