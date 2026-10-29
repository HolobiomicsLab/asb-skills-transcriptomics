---
name: cardiomyocyte-marker-expression-profiling
description: Use when assigning biological identity to day-60 terminal subpopulations
  on a TCM latent map of hiPSC-to-cardiomyocyte scRNA-seq data by overlaying log-expression
  of TNNT2, MYL2, NPPA, NPPB, NR2F2 and CDH11.
when_to_use_negative:
- The dataset has no complete time index (no time.table), since TCM requires the time
  index to be correlated with the underlying dynamic expression pattern.
- The six marker genes (TNNT2, MYL2, NPPA, NPPB, NR2F2, CDH11) are absent from the
  gene panel, e.g. low-dimensional single-cell PCR data such as the guo dataset.
- The goal is de novo trajectory or cluster discovery on non-cardiac differentiation
  data - the C1-C4 identities here are specific to the day-60 hiPSC-to-cardiomyocyte
  terminal populations.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: tcm
  role: Fits the temporal-convolving TCM model on the iPSC counts and supplies plot(mf,
    ...) and num2color() to overlay marker expression on the 2D latent map used to
    define C1-C4.
  repo: github.com/gongx030/tcm
- name: scDatasets
  role: Supplies the human iPSC-CM differentiation dataset via data(iPSC) (count matrix
    and time.table).
- name: SummarizedExperiment
  role: Provides assays()/colData() accessors used to extract the count matrix and
    time.table from the iPSC object.
- name: num2color
  role: Maps log(X + 1) marker expression values to background colors for per-gene
    latent-map overlays (e.g., TNNT2).
  repo: github.com/gongx030/tcm
- name: devtools
  role: Installs the tcm package via install_github('gongx030/tcm').
  repo: https://github.com/hadley/devtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:c3d03da0626af9119daf7a20f8c7a0b1fa27963b7f50574ba68c82dbe0d757b5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SummarizedExperiment
  - devtools
  - num2color
  - scDatasets
  - tcm
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# cardiomyocyte-marker-expression-profiling

## Summary

Profiles canonical cardiomyocyte-lineage markers (TNNT2, MYL2, NPPA, NPPB, NR2F2, CDH11) as log(X + 1)/num2color overlays on the TCM 2D latent map of the scDatasets iPSC hiPSC-to-cardiomyocyte differentiation scRNA-seq dataset, to verify the identities of the four day-60 terminal subpopulations C1-C4.

## When to use

Use when assigning biological identity to day-60 terminal subpopulations on a TCM latent map of hiPSC-to-cardiomyocyte scRNA-seq data by overlaying log-expression of TNNT2, MYL2, NPPA, NPPB, NR2F2 and CDH11.

## When NOT to use

- The dataset has no complete time index (no time.table), since TCM requires the time index to be correlated with the underlying dynamic expression pattern.
- The six marker genes (TNNT2, MYL2, NPPA, NPPB, NR2F2, CDH11) are absent from the gene panel, e.g. low-dimensional single-cell PCR data such as the guo dataset.
- The goal is de novo trajectory or cluster discovery on non-cardiac differentiation data - the C1-C4 identities here are specific to the day-60 hiPSC-to-cardiomyocyte terminal populations.

## Inputs

- Gene-by-cell raw count matrix X from scDatasets::data(iPSC) (human hiPSC-CM differentiation scRNA-seq)
- Cell-by-time-point binary time.table (colData of the iPSC SummarizedExperiment)
- Fitted TCM model object (mf) with 2D latent coordinates for all cells
- Day-60 cell subpopulation assignments C1-C4 derived from the TCM map
- Marker gene list: TNNT2, MYL2, NPPA, NPPB, NR2F2, CDH11

## Outputs

- Per-marker overlay plots on the TCM latent map with num2color(log(X + 1)) background colors
- CSV assignment table with one row per day-60 cell labeled C1-C4
- CSV matrix of log-transformed expression for the six markers across day-60 cells

## How to apply

First fit TCM on the scDatasets iPSC dataset: preprocess(X, min.expressed.gene = 0), build the landscape via landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2), then with set.seed(1) run tcm(X, time.table = time.table, ls = ls, init = list(method = 'backward', update.beta = TRUE)). Extract the 2D latent coordinates, subset to day-60 cells (max column of time.table), and group them into the four terminal subpopulations C1-C4 following the map's circle structure. For each marker - TNNT2, MYL2, NPPA, NPPB, NR2F2, CDH11 - compute log(X + 1) counts and render them as background colors with num2color, e.g. plot(mf, pch = 21, bg = num2color(log(X[gene, ] + 1))). Read identities off the map: C1-C2 are TNNT2/MYL2-positive contractile cardiomyocytes, C1 is additionally NPPA/NPPB-enriched, C3 is NR2F2-positive, and C4 is CDH11-positive, reproducing the reported Fig. 4d arrangement. Export the per-cell C1-C4 assignment table and the 6-marker log-expression matrix as CSVs for validation.

## Related tools

- **tcm** (Fits the temporal-convolving TCM model on the iPSC counts and supplies plot(mf, ...) and num2color() to overlay marker expression on the 2D latent map used to define C1-C4.) — github.com/gongx030/tcm
- **scDatasets** (Supplies the human iPSC-CM differentiation dataset via data(iPSC) (count matrix and time.table).)
- **SummarizedExperiment** (Provides assays()/colData() accessors used to extract the count matrix and time.table from the iPSC object.)
- **num2color** (Maps log(X + 1) marker expression values to background colors for per-gene latent-map overlays (e.g., TNNT2).) — github.com/gongx030/tcm
- **devtools** (Installs the tcm package via install_github('gongx030/tcm').) — https://github.com/hadley/devtools

## Examples

```
plot(mf, pch = 21, bg = num2color(log(X['ENSG00000118194', ] + 1)), cex = 1.25, main = 'TNNT2')
```

## Evaluation signals

- Assignment CSV contains exactly one row per day-60 cell and every label is one of C1-C4.
- TNNT2 and MYL2 log-expression is high in C1 and C2 but low in C3 and C4.
- NPPA and NPPB are enriched in C1 relative to C2 (atrial-like identity).
- NR2F2 marks C3 and CDH11 marks C4, both negative for TNNT2/MYL2.
- The spatial arrangement of marker-high cells reproduces the reported Fig. 4d pattern.

## Limitations

- TCM does not provide immediate biological interpretation of cell clusters on the latent space; marker readout covers only the six curated genes and further pathway analysis is needed for fuller annotation.
- TCM requires scRNA-seq datasets with a complete time index that correlates with the underlying dynamic expression pattern.
- C1-C4 labels depend on the TCM map geometry and the set.seed(1) 'backward' initialization; different seeds or landscape parameters (K = 15, n.prototype = 15, n.circle = 10, n.prev = 2) may change subpopulation structure.

## Evidence

- [abstract] Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point: "Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point"
- [methods] ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2): "ls <- landscape(type = 'temporal.convolving', time.points = ncol(time.table), K = 15, n.prototype = 15, n.circle = 10, n.prev = 2)"
- [readme] plot(mf, pch = 21, bg = num2color(log(X['ENSG00000118194', ] + 1)), cex = 1.25, main = 'TNNT2'): "plot(mf, pch = 21, bg = num2color(log(X['ENSG00000118194', ] + 1)), cex = 1.25, main = 'TNNT2')"
- [discussion] TCM does not provide immediate biological interpretation of cell clusters on the latent space, and further pathway analysis will need to be conducted to elucidate the biological meaning of each: "TCM does not provide immediate biological interpretation of cell clusters on the latent space, and further pathway analysis will need to be conducted to elucidate the biological meaning of each"
- [discussion] TCM requires the scRNA-seq datasets with complete time index and the time index needs to be correlated with the underlying dynamic expression pattern.: "TCM requires the scRNA-seq datasets with complete time index and the time index needs to be correlated with the underlying dynamic expression pattern."
