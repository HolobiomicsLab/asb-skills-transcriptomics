---
name: single-cell-rnaseq-preprocessing-and-clustering
description: Use when raw single-cell RNA-seq count matrices (e.g., from liver transplant
  or SRA cohorts such as SRP313633) need normalization, clustering, and annotation
  into cell populations before downstream cell-cell communication or trajectory analysis.
when_to_use_negative:
- Input is already a normalized, annotated cell-type table or niche score matrix —
  no re-clustering is needed
- Data are bulk RNA-seq or spatial transcriptomics without single-cell resolution,
  since scCrossTalk and the niche cell-type definitions require per-cell annotations
- The goal is cell-cell communication inference itself — use the downstream scCrossTalk
  find_lrpairs/visualization functions on an already preprocessed object
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: R package used for normalization, dimensionality reduction, clustering, and
    cell-type annotation of the scRNA-seq data; scCrossTalk is compatible with Seurat
    5
  repo: https://github.com/multitalk/scCrossTalk
- name: scCrossTalk
  role: Downstream consumer of the preprocessed sc_data/sc_celltype objects for cell-cell
    communication inference via CellTalkDB ligand-receptor enrichment
  repo: https://github.com/multitalk/scCrossTalk
- name: R
  role: Analysis environment in which Seurat preprocessing and scCrossTalk installation
    (devtools::install_local / install_github) are performed
  repo: https://github.com/multitalk/scCrossTalk
- name: devtools
  role: Used to install scCrossTalk from GitHub or a local ZIP archive
  repo: https://github.com/multitalk/scCrossTalk
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:2d1a527738d7ccde3e105eabb2faa829d0b4a5b6ca45babf2375f35f738e4427
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/multitalk/scCrossTalk
  repo_url: https://github.com/multitalk/scCrossTalk
  tools:
  - R
  - Seurat
  - devtools
  - ggplot2
  - scCrossTalk
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-cell-rnaseq-preprocessing-and-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Standard Seurat-based preprocessing, dimensionality reduction, clustering, and cell-type annotation of scRNA-seq data from human liver transplantation samples, used to identify EAD-associated immune populations (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) and prepare inputs for scCrossTalk cell-cell communication inference.

## When to use

Use when raw single-cell RNA-seq count matrices (e.g., from liver transplant or SRA cohorts such as SRP313633) need normalization, clustering, and annotation into cell populations before downstream cell-cell communication or trajectory analysis.

## When NOT to use

- Input is already a normalized, annotated cell-type table or niche score matrix — no re-clustering is needed
- Data are bulk RNA-seq or spatial transcriptomics without single-cell resolution, since scCrossTalk and the niche cell-type definitions require per-cell annotations
- The goal is cell-cell communication inference itself — use the downstream scCrossTalk find_lrpairs/visualization functions on an already preprocessed object

## Inputs

- Raw scRNA-seq count matrix from human liver transplant samples (non-EAD and EAD, cold perfusion and portal reperfusion stages; 12 samples from 7 donors, 75,231 cells total)
- Independent verification cohort raw scRNA-seq data from SRA accession SRP313633
- Marker definitions for MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils from the scCrossTalk manuscript

## Outputs

- Annotated Seurat object with cell-type labels including MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils
- Normalized, clustered expression matrix (sc_data) and cell-type annotation vector (sc_celltype) formatted for create_scCrossTalk
- Per-sample cell abundance tables used to compute EAD immune niche scores

## How to apply

Download raw scRNA-seq data (e.g., from SRA accession SRP313633 for the independent verification cohort) and load it into R with the Seurat package. Perform standard Seurat preprocessing — normalization, dimensionality reduction, and clustering — then annotate cell populations using the marker definitions from the scCrossTalk manuscript, identifying MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils among the 75,231 cells profiled from non-EAD and EAD patients. Retain spliced/unspliced abundance layers where provided if the annotated subsets will feed Monocle 3-style pseudo-time trajectory ordering. The resulting annotated cell-type vector (sc_celltype) and count matrix (sc_data) are the required inputs to create_scCrossTalk in the scCrossTalk package, which is compatible with Seurat 5.

## Related tools

- **Seurat** (R package used for normalization, dimensionality reduction, clustering, and cell-type annotation of the scRNA-seq data; scCrossTalk is compatible with Seurat 5) — https://github.com/multitalk/scCrossTalk
- **scCrossTalk** (Downstream consumer of the preprocessed sc_data/sc_celltype objects for cell-cell communication inference via CellTalkDB ligand-receptor enrichment) — https://github.com/multitalk/scCrossTalk
- **R** (Analysis environment in which Seurat preprocessing and scCrossTalk installation (devtools::install_local / install_github) are performed) — https://github.com/multitalk/scCrossTalk
- **devtools** (Used to install scCrossTalk from GitHub or a local ZIP archive) — https://github.com/multitalk/scCrossTalk

## Examples

```
library(Seurat); obj <- create_scCrossTalk(sc_data, sc_celltype, species); obj <- find_lrpairs(obj, lrpairs = lrpairs)
```

## Evaluation signals

- Clustered object contains annotated MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils matching the manuscript marker definitions
- Cell count is consistent with the cohort scale (e.g., 75,231 cells for the discovery cohort, or the expected SRP313633 verification cohort size)
- create_scCrossTalk(sc_data, sc_celltype, species) accepts the preprocessed inputs and returns 'An object of class scCrossTalk' with 0 ligand-receptor interactions before find_lrpairs is run
- Annotated cell-type vector length matches the number of columns/cells in the count matrix sc_data
- Downstream niche scores computed from annotated populations reproduce the EAD vs non-EAD association reported in the SRP313633 verification (one-sided Welch's test)

## Limitations

- The scCrossTalk package was designed to dissect cell-cell communications in transplanted livers of EAD and non-EAD patients; marker definitions for the immune niche populations are specific to this manuscript
- The scCrossTalk repository has no changelog (missing artifact), so version-specific preprocessing compatibility beyond the stated 'Compatible with Seurat 5' is undocumented
- Discussion section content is absent from the available article text, so caveats about clustering robustness are not documented in the source

## Evidence

- [other] Download raw data and preprocess with Seurat for verification cohort: [evidence span withheld — anchor status: not_found]
