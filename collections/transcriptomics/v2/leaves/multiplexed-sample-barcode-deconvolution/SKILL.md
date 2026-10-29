---
name: multiplexed-sample-barcode-deconvolution
description: Use when multiplexed 10x Genomics droplet UMI count matrices (e.g., HiSeq
  4000 patterned-flow-cell data) must be split into real cells versus empty droplets,
  reporting per-sample called-cell counts and library-size distributions.
when_to_use_negative:
- Data are plate-based scRNA-seq (e.g., Smart-seq2 or the Richard mouse T-cell plate
  dataset) with no empty droplets to distinguish from cells
- Barcodes have not been demultiplexed per sample, so the cell exclusion step removing
  barcodes shared between samples cannot be applied
- The goal is removal of barcode-swapped pseudo-cells from already-called cells rather
  than initial cell calling from empty droplets
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DropletUtils
  role: R package providing emptyDrops for identification of cells from empty droplets
    in 10x Genomics droplet data
  repo: https://github.com/MarioniLab/DropletUtils
- name: 10x Genomics
  role: Droplet-based scRNA-seq protocol generating the molecule information parsed
    into per-sample UMI count matrices
- name: R
  role: Environment in which DropletUtils/emptyDrops is run and per-sample summaries
    are assembled
- name: HiSeq 4000
  role: Patterned-flow-cell Illumina instrument on which the multiplexed libraries
    were sequenced; source of barcode swapping motivating shared-barcode exclusion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:db51f4c0e4b2a39724c7be8f3003a1510bcd46ee190c2f9b533584c8c2d9c6ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - 10x Genomics
  - DropletUtils
  - HiSeq 4000
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# multiplexed-sample-barcode-deconvolution

## Summary

Calls putative cells from multiplexed 10x Genomics droplet data (e.g., HiSeq 4000-sequenced mouse epithelial libraries from ArrayExpress E-MTAB-6854) using emptyDrops with an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs, after removing barcodes shared between samples to guard against barcode swapping.

## When to use

Use when multiplexed 10x Genomics droplet UMI count matrices (e.g., HiSeq 4000 patterned-flow-cell data) must be split into real cells versus empty droplets, reporting per-sample called-cell counts and library-size distributions.

## When NOT to use

- Data are plate-based scRNA-seq (e.g., Smart-seq2 or the Richard mouse T-cell plate dataset) with no empty droplets to distinguish from cells
- Barcodes have not been demultiplexed per sample, so the cell exclusion step removing barcodes shared between samples cannot be applied
- The goal is removal of barcode-swapped pseudo-cells from already-called cells rather than initial cell calling from empty droplets

## Inputs

- Droplet-prepared 10x Genomics molecule information files from multiplexed mouse epithelial samples (ArrayExpress E-MTAB-6854, HiSeq 4000-sequenced)
- Per-sample UMI count matrices parsed from the molecule information
- emptyDrops parameters: FDR threshold 0.01, minimum cell size 1000 UMIs

## Outputs

- Per-sample set of barcodes called as cells (emptyDrops calls at FDR < 0.01)
- emptyDrops_called_cells.tsv: combined table with one row per sample reporting called-cell counts and library-size (total UMI count) distributions

## How to apply

Download the droplet-prepared mouse epithelial data (ArrayExpress E-MTAB-6854, HiSeq 4000-sequenced libraries) and parse the molecule information files into per-sample UMI count matrices. First restrict to barcodes that are not shared between samples, applying the cell exclusion strategy that removes all barcodes shared between samples, because approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000. For each sample's count matrix, run emptyDrops from the DropletUtils R package, which tests each barcode's expression profile for significant deviation from the ambient RNA profile, using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs; droplets with very large counts are automatically retained. For each sample, summarize the number of barcodes called as cells and the distribution of total UMI counts (library sizes) of the called cells, and write the combined per-sample summary to emptyDrops_called_cells.tsv.

## Related tools

- **DropletUtils** (R package providing emptyDrops for identification of cells from empty droplets in 10x Genomics droplet data) — https://github.com/MarioniLab/DropletUtils
- **10x Genomics** (Droplet-based scRNA-seq protocol generating the molecule information parsed into per-sample UMI count matrices)
- **R** (Environment in which DropletUtils/emptyDrops is run and per-sample summaries are assembled)
- **HiSeq 4000** (Patterned-flow-cell Illumina instrument on which the multiplexed libraries were sequenced; source of barcode swapping motivating shared-barcode exclusion)

## Examples

```
library(DropletUtils); called <- emptyDrops(counts_per_sample[[s]], lower=1000); cells <- rownames(called)[called$FDR <= 0.01 | is.na(called$FDR)]
```

## Evaluation signals

- emptyDrops_called_cells.tsv is well-formed with exactly one row per HiSeq 4000 sample
- The table reports both the number of called cells and the library-size (total UMI count) distribution for every sample
- Called cells correspond to barcodes with emptyDrops FDR below 0.01 and library size of at least 1000 UMIs (or automatic retention of very large-count droplets)
- No barcodes shared between samples appear among the called cells, confirming the cell exclusion strategy was applied before calling
- All HiSeq 4000 samples from E-MTAB-6854 are represented in the summary, none silently dropped

## Limitations

- Cell exclusion by removing all barcodes shared between samples can incorrectly discard genuine cell libraries; the fraction incorrectly discarded was counted in the authors' barcode sampling simulations
- Approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000, so multiplexed droplet data from patterned flow-cell instruments carry artefactual swapped libraries that motivated the exclusion step
- Barcode swapping may generate complex but artefactual cell libraries in droplet-based scRNA-seq studies, which emptyDrops cell calling alone does not correct

## Evidence

- [methods] Cell calling with emptyDrops at FDR threshold 0.01 and minimum cell size 1000 UMIs: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [discussion] Cell exclusion strategy removing barcodes shared between samples: "We applied cell exclusion by removing all barcodes that were shared between samples"
- [discussion] E-MTAB-6854 accession for droplet-prepared mouse epithelial cells: "and E-MTAB-6854 for droplet-prepared HiSeq 2500- and HiSeq 4000-sequenced mouse epithelial cells."
- [readme] DropletUtils provides identification of cells from empty droplets: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
- [abstract] Approximately 2.5% of reads mislabelled between samples on the HiSeq 4000: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000, which is lower than previous reports"
