---
name: developmental-hematopoiesis-marker-interpretation
description: Use when temporal scRNA-seq of developing mesoderm/hematopoiesis (e.g.
  E7.5/E7.75 mouse Cd41+ cells) has been projected with TCM and you must identify
  and biologically interpret subpopulations within one time point using marker genes.
when_to_use_negative:
- The dataset lacks a complete time index, or the time index is not correlated with
  the underlying dynamic expression pattern (a stated requirement of TCM)
- You only need clustering on raw expression without trajectory context — TCM's temporal
  convolving adds value only when preserving global developmental trajectories matters
- The cells are not from a developmental time course and cannot be assigned to time
  points for the time.table input
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tcm
  role: Fits the Topographic Cell Map model producing the 2D latent space on which
    the E7.75 Cd41+ cells are clustered and marker overlays are plotted
  repo: https://github.com/gongx030/tcm
- name: SummarizedExperiment
  role: Provides the assays()/colData() interface for loading count matrices and time
    tables into the TCM workflow
- name: devtools
  role: Installs the tcm package via install_github('gongx030/tcm')
  repo: https://github.com/hadley/devtools
- name: R
  role: Implementation environment for the tcm package and downstream clustering/marker-overlay
    analyses (R version 3.4.3 in the paper)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05112-9
    title: TCM visualizes trajectories and cell populations from single cell data
  source_content_hash: sha256:a9fcb44e9063fb56ac717469c37e851c79fa8afc7f186b68e156498c255b6f9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gongx030/tcm
  tools:
  - R
  - SummarizedExperiment
  - devtools
  - tcm
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# developmental-hematopoiesis-marker-interpretation

## Summary

Clusters E7.75 Cd41+ hematopoietic cells on the TCM 2D latent space into subpopulations C1–C4 and interprets them via marker-gene expression overlays (Hand1, Fgfr1, Gata1, Hba-x, Zfpm1, Gata2, Hbb-bh1), distinguishing progenitor (C1), Zfpm1/Gata2 (C2–C3), and differentiated erythroid (C4, Hbb-bh1/Hba-x high) states.

## When to use

Use when temporal scRNA-seq of developing mesoderm/hematopoiesis (e.g. E7.5/E7.75 mouse Cd41+ cells) has been projected with TCM and you must identify and biologically interpret subpopulations within one time point using marker genes.

## When NOT to use

- The dataset lacks a complete time index, or the time index is not correlated with the underlying dynamic expression pattern (a stated requirement of TCM)
- You only need clustering on raw expression without trajectory context — TCM's temporal convolving adds value only when preserving global developmental trajectories matters
- The cells are not from a developmental time course and cannot be assigned to time points for the time.table input

## Inputs

- Gene-by-cell scRNA-seq count matrix (X) from E7.5/E7.75 mouse mesoderm cells
- Binary cell-by-time-point time.table indicating each cell's time point
- Cd41+ cell subset metadata for E7.75 cells
- Named marker gene list: Hand1, Fgfr1, Gata1, Hba-x, Zfpm1, Gata2, Hbb-bh1

## Outputs

- Cell-to-subpopulation assignments for E7.75 Cd41+ cells (cd41_subpopulation_assignments.csv, clusters C1–C4)
- Per-cluster marker-expression matrix for Hand1, Fgfr1, Gata1, Hba-x, Zfpm1, Gata2, Hbb-bh1
- TCM 2D map with per-cluster marker-expression color overlays

## How to apply

Load the E7.5/E7.75 count matrix and binary time.table, filter low-quality cells with preprocess(X), then fit TCM with a fixed random seed via tcm(X, time.table = time.table). Plot all cells on the 2D TCM map (plot(mf, pch = 21, bg = bg.cell)) and subset the E7.75 Cd41+ cells. Cluster those cells on their TCM latent-space coordinates into four groups C1–C4, then overlay each marker gene with num2color(log(X[gene, ] + 1)) on the TCM map. Interpret directions: C1 should show Hand1/Fgfr1 up with Gata1/Hba-x down (progenitor state), C2–C3 show Zfpm1/Gata2, and C4 shows high Hbb-bh1/Hba-x. Acceptance is qualitative — the C1–C4 signature directions must match the Fig. 4b pattern, since no numeric threshold is reported.

## Related tools

- **tcm** (Fits the Topographic Cell Map model producing the 2D latent space on which the E7.75 Cd41+ cells are clustered and marker overlays are plotted) — https://github.com/gongx030/tcm
- **SummarizedExperiment** (Provides the assays()/colData() interface for loading count matrices and time tables into the TCM workflow)
- **devtools** (Installs the tcm package via install_github('gongx030/tcm')) — https://github.com/hadley/devtools
- **R** (Implementation environment for the tcm package and downstream clustering/marker-overlay analyses (R version 3.4.3 in the paper))

## Examples

```
library(tcm); X <- preprocess(X); set.seed(1); mf <- tcm(X, time.table = time.table); plot(mf, pch = 21, bg = num2color(log(X['Hbb-bh1', ] + 1)))
```

## Evaluation signals

- C1 shows increased Hand1 and Fgfr1 and decreased Gata1 and Hba-x relative to other clusters, indicating progenitor-state cells
- C2 and C3 show Zfpm1/Gata2 expression
- C4 shows high Hbb-bh1 and Hba-x expression consistent with differentiated erythroid cells
- The C1–C4 marker-signature directions match the Fig. 4b pattern (qualitative acceptance check; no numeric threshold is reported)
- TCM fit is reproducible after setting a random seed (e.g. set.seed(1) as in README examples)

## Limitations

- TCM does not provide immediate biological interpretation of cell clusters on the latent space; further pathway analysis is needed
- Acceptance is qualitative (signature directions), since no numeric threshold is reported for the marker patterns
- TCM requires scRNA-seq datasets with a complete time index that correlates with the underlying dynamic expression pattern

## Evidence

- [abstract] TCM preserves trajectories and identifies subpopulations: "Our software preserves the global developmental trajectories over a specified time course, and it also identifies subpopulations of cells within each time point"
- [methods] TCM model fitting call: "mf <- tcm(X, time.table = time.table)"
- [methods] preprocessing of count matrices: [evidence span withheld — anchor status: not_found]
- [discussion] downstream analysis on TCM latent space: "Downstream analysis such as trajectory inference8,11, cell clustering13,23 and differential expression analysis14 could be readily performed on the latent space produced by TCM."
- [discussion] no built-in biological interpretation: "TCM does not provide immediate biological interpretation of cell clusters on the latent space, and further pathway analysis will need to be conducted"
- [readme] marker gene expression overlay on TCM map: "plot(mf, pch = 21, bg = num2color(log(X['ENSG00000118194', ] + 1)), cex = 1.25, main = 'TNNT2')"
