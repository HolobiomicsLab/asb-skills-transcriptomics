---
name: ambient-rna-fdr-threshold-testing
description: Use when 10x Genomics droplet UMI count matrices (e.g., parsed from molecule
  information files of HiSeq 4000-sequenced libraries) must be separated into real
  cells versus empty droplets at a specified FDR.
when_to_use_negative:
- Input is plate-based scRNA-seq data (e.g., Smart-seq2 plate datasets such as Nestorowa
  GSE81682 or the Richard mouse T-cell plates) — there are no empty droplets or ambient
  profile for emptyDrops to test against.
- Barcodes shared between samples have not yet been removed from multiplexed HiSeq
  4000 10x data — apply cell exclusion first, since swapped molecules can produce
  artefactual cell libraries that would distort the calls.
- The input is already a matrix of called cells from a prior cell-calling step — re-running
  emptyDrops at FDR 0.01 would double-call rather than filter.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DropletUtils
  role: R package providing emptyDrops for identification of cells from empty droplets,
    plus utilities for loading 10x count matrices or molecule information files and
    removing barcode-swapped pseudo-cells
  repo: https://bioconductor.org/packages/DropletUtils
- name: R
  role: Environment in which DropletUtils/emptyDrops cell calling and the per-sample
    library-size summarization are executed
- name: 10x Genomics
  role: Droplet technology whose cell barcodes and molecule information supply the
    per-sample UMI count matrices being called
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:5673e3c308264f5064298a4b267bf762adb663e680cdfbd961d9249700a21d2e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics
  - DropletUtils
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ambient-rna-fdr-threshold-testing

## Summary

Statistical cell calling for 10x Genomics droplet scRNA-seq: emptyDrops (DropletUtils) tests each barcode's expression profile for significant deviation from the ambient profile, calling cells at FDR 0.01 with a minimum cell size of 1000 UMIs on the HiSeq 4000-sequenced mouse epithelial samples.

## When to use

Use when 10x Genomics droplet UMI count matrices (e.g., parsed from molecule information files of HiSeq 4000-sequenced libraries) must be separated into real cells versus empty droplets at a specified FDR.

## When NOT to use

- Input is plate-based scRNA-seq data (e.g., Smart-seq2 plate datasets such as Nestorowa GSE81682 or the Richard mouse T-cell plates) — there are no empty droplets or ambient profile for emptyDrops to test against.
- Barcodes shared between samples have not yet been removed from multiplexed HiSeq 4000 10x data — apply cell exclusion first, since swapped molecules can produce artefactual cell libraries that would distort the calls.
- The input is already a matrix of called cells from a prior cell-calling step — re-running emptyDrops at FDR 0.01 would double-call rather than filter.

## Inputs

- 10x Genomics droplet UMI count matrices / molecule information files for the droplet-prepared mouse epithelial samples (ArrayExpress E-MTAB-6854, HiSeq 4000-sequenced libraries)
- Per-sample barcode-by-gene count matrices restricted to barcodes not shared between samples (after cell exclusion)

## Outputs

- emptyDrops_called_cells.tsv: combined per-sample summary table (one row per sample) reporting the number of barcodes called as cells and the distribution of total UMI counts (library sizes) of the called cells

## How to apply

Download the droplet-prepared mouse epithelial data from ArrayExpress E-MTAB-6854 (HiSeq 4000-sequenced libraries) and parse the 10x Genomics molecule information into per-sample UMI count matrices. First apply cell exclusion by removing all barcodes that are shared between samples, because barcode swapping on the HiSeq 4000 mislabels approximately 2.5% of reads and can generate complex but artefactual swapped-cell libraries. Run emptyDrops from the DropletUtils R package on each sample's count matrix using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs; emptyDrops tests each barcode's expression profile for significant deviation from the ambient profile of empty droplets, and very large-count droplets are automatically retained. Finally, summarize for each sample the number of barcodes called as cells and the distribution of total UMI counts (library sizes) of the called cells, writing the combined per-sample summary to emptyDrops_called_cells.tsv with one row per sample.

## Related tools

- **DropletUtils** (R package providing emptyDrops for identification of cells from empty droplets, plus utilities for loading 10x count matrices or molecule information files and removing barcode-swapped pseudo-cells) — https://bioconductor.org/packages/DropletUtils
- **R** (Environment in which DropletUtils/emptyDrops cell calling and the per-sample library-size summarization are executed)
- **10x Genomics** (Droplet technology whose cell barcodes and molecule information supply the per-sample UMI count matrices being called)

## Examples

```
library(DropletUtils); e.out <- emptyDrops(counts, lower=1000); is.cell <- e.out$FDR <= 0.01 | e.out$Limited
```

## Evaluation signals

- emptyDrops_called_cells.tsv is well-formed with exactly one row per HiSeq 4000-sequenced sample from E-MTAB-6854.
- Per-sample called-cell counts correspond to barcodes with emptyDrops FDR <= 0.01, with very large-count droplets automatically retained alongside the significant calls.
- Called cells respect the 1000-UMI minimum cell size, so reported library-size distributions (total UMI counts) are consistent with that threshold.
- No cell barcode appears in more than one sample's called set — barcodes shared between samples were removed before cell calling.
- Each sample's row reports both a called-cell count and a library-size distribution, not just a single summary metric.

## Limitations

- Cell exclusion by removing all barcodes shared between samples can incorrectly discard genuine cell libraries — the paper explicitly counted the fraction of cell libraries that were incorrectly discarded.
- On the HiSeq 4000, approximately 2.5% of reads are mislabelled between samples and barcode swapping may generate complex but artefactual cell libraries; cell calling alone does not correct this — DropletUtils separately provides removal of barcode-swapped pseudo-cells.
- Calling depends on an ambient profile estimated from empty droplets, so the method requires droplet data containing a population of empty droplets.

## Evidence

- [discussion] Cell calling performed with emptyDrops at FDR threshold 0.01 and minimum cell size 1000 UMIs: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [discussion] Cell exclusion removes all barcodes shared between samples before calling: "We applied cell exclusion by removing all barcodes that were shared between samples"
- [discussion] Droplet-prepared mouse epithelial data acquired from ArrayExpress E-MTAB-6854: "and E-MTAB-6854 for droplet-prepared HiSeq 2500- and HiSeq 4000-sequenced mouse epithelial cells."
- [readme] DropletUtils identifies cells from empty droplets and removes barcode-swapped pseudo-cells: "identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
- [abstract] Approximately 2.5% of reads mislabelled between samples on the HiSeq 4000: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000"
- [abstract] Barcode swapping can generate artefactual cell libraries in droplet scRNA-seq: "barcode swapping may generate complex but artefactual cell libraries in droplet-based single-cell RNA-sequencing studies"
