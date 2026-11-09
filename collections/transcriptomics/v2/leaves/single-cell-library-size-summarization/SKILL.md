---
name: single-cell-library-size-summarization
description: Use when raw droplet 10x Genomics data (e.g. mouse epithelial cells from
  ArrayExpress E-MTAB-6854 sequenced on HiSeq 4000) must be reduced from all barcodes
  to called cells, with per-sample cell counts and library-size summaries reported.
when_to_use_negative:
- Input barcodes shared between samples have not yet been removed via the cell exclusion
  strategy, so swapping-derived artefactual libraries would contaminate the cell calls
- The data are plate-based (e.g. Smart-seq2) with no empty-droplet background, so
  emptyDrops ambient-profile testing does not apply
- A different cell-calling method or threshold is required, since the skill is pinned
  to emptyDrops with FDR 0.01 and a 1000-UMI minimum cell size
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DropletUtils
  role: Provides emptyDrops and related utilities for loading 10x Genomics count matrices,
    identifying cells from empty droplets, and removing barcode-swapped pseudo-cells
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Environment in which the emptyDrops cell calling and per-sample summarization
    script is executed
- name: 10x Genomics
  role: Source of the droplet-prepared mouse epithelial scRNA-seq data and molecule
    information files processed into UMI count matrices
- name: HiSeq 4000
  role: Patterned-flowcell sequencing instrument whose libraries require the barcode-swapping-aware
    cell exclusion before cell calling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:ea94179106992e96099d9792fe8ed5e48a5e950c88d045513db4c9599f4a4983
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - 10x Genomics
  - DropletUtils
  - HiSeq 4000
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# single-cell-library-size-summarization

## Summary

Calls putative cells from 10x Genomics droplet UMI count matrices using emptyDrops (FDR threshold 0.01, minimum cell size 1000 UMIs) and summarizes per sample the number of called cells and the distribution of their total UMI counts (library sizes).

## When to use

Use when raw droplet 10x Genomics data (e.g. mouse epithelial cells from ArrayExpress E-MTAB-6854 sequenced on HiSeq 4000) must be reduced from all barcodes to called cells, with per-sample cell counts and library-size summaries reported.

## When NOT to use

- Input barcodes shared between samples have not yet been removed via the cell exclusion strategy, so swapping-derived artefactual libraries would contaminate the cell calls
- The data are plate-based (e.g. Smart-seq2) with no empty-droplet background, so emptyDrops ambient-profile testing does not apply
- A different cell-calling method or threshold is required, since the skill is pinned to emptyDrops with FDR 0.01 and a 1000-UMI minimum cell size

## Inputs

- Molecule information files / count matrices from droplet-prepared 10x Genomics samples (mouse epithelial cells, ArrayExpress E-MTAB-6854, HiSeq 4000-sequenced libraries)
- Per-sample UMI count matrices restricted to barcodes not shared between samples

## Outputs

- emptyDrops_called_cells.tsv: combined table with one row per sample, reporting the number of barcodes called as cells and the distribution of total UMI counts (library sizes) of called cells

## How to apply

Parse molecule information from the droplet-prepared samples into per-sample UMI count matrices, then remove all barcodes that are shared between samples (the cell exclusion strategy that guards against barcode-swapped pseudo-cells on patterned-flowcell instruments like the HiSeq 4000). For each sample's count matrix, run emptyDrops from the DropletUtils R package with an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs; emptyDrops tests each barcode's expression profile against the ambient profile and automatically retains very large-count droplets. For each sample, record the number of barcodes called as cells (FDR <= 0.01) and summarize the distribution of total UMI counts (library sizes) of the called cells. Combine the per-sample summaries into a single well-formed table (emptyDrops_called_cells.tsv) with one row per sample.

## Related tools

- **DropletUtils** (Provides emptyDrops and related utilities for loading 10x Genomics count matrices, identifying cells from empty droplets, and removing barcode-swapped pseudo-cells) — https://github.com/MarioniLab/DropletUtils
- **R** (Environment in which the emptyDrops cell calling and per-sample summarization script is executed)
- **10x Genomics** (Source of the droplet-prepared mouse epithelial scRNA-seq data and molecule information files processed into UMI count matrices)
- **HiSeq 4000** (Patterned-flowcell sequencing instrument whose libraries require the barcode-swapping-aware cell exclusion before cell calling)

## Examples

```
library(DropletUtils); e <- emptyDrops(counts, lower = 1000); called <- which(e$FDR <= 0.01, useNames = TRUE); sum(librarySize = colSums(counts)[called])
```

## Evaluation signals

- emptyDrops_called_cells.tsv is well-formed with one row per HiSeq 4000 sample and reports both called-cell counts and library-size distributions
- Cell calling used the specified emptyDrops parameters: FDR threshold of 0.01 and minimum cell size of 1000 UMIs
- All barcodes shared between samples were excluded before cell calling (cell exclusion strategy)
- Very large-count droplets are automatically retained by emptyDrops, so the called-cell count should not miss total-count outliers
- Called-cell library sizes (total UMI counts) fall above the 1000-UMI minimum cell size

## Limitations

- Cell exclusion by removing all barcodes shared between samples can incorrectly discard genuine cell libraries, a fraction the authors quantified via barcode sampling simulation
- Barcode swapping on the HiSeq 4000 (~2.5% of reads mislabelled) may generate complex but artefactual cell libraries in droplet-based scRNA-seq studies, so barcode sharing between samples must be handled before trusting cell calls
- Supplementary Notes containing details of the molecule-exclusion and simulation methods were unavailable from the electronic supplementary material ('No changelog found.')

## Evidence

- [discussion] Cell calling with emptyDrops at FDR threshold 0.01 and minimum cell size 1000 UMIs: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [discussion] Cell exclusion removes barcodes shared between samples: "We applied cell exclusion by removing all barcodes that were shared between samples"
- [readme] DropletUtils provides empty-droplet cell identification: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
- [readme] DropletUtils targets 10x Genomics droplet data: "The **DropletUtils** package provides a number of utility functions for handling single-cell (RNA-seq) data from droplet technologies such as 10X Genomics."
- [discussion] Droplet dataset source accession: "and E-MTAB-6854 for droplet-prepared HiSeq 2500- and HiSeq 4000-sequenced mouse epithelial cells."
