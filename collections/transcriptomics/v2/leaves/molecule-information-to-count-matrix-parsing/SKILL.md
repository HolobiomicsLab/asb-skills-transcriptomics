---
name: molecule-information-to-count-matrix-parsing
description: Use when starting from 10x Genomics droplet molecule information (raw
  per-molecule records with cell barcode, UMI, gene) and you need per-sample UMI count
  matrices for cell calling or barcode-swapping exclusion.
when_to_use_negative:
- Input is already a gene-by-cell count matrix (e.g. a 10x matrix.mtx/h5 bundle);
  skip parsing and go directly to cell calling or exclusion.
- Data are plate-based (e.g. Smart-seq2 mouse T-cell plates from E-MTAB-6843); molecule
  information parsing for 10x droplet data does not apply.
- The analysis target is whole-transcript coverage outside exonic regions; the pipeline
  here counts only read pairs mapping to the exonic region of each gene.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DropletUtils
  role: R/Bioconductor package providing data loading from count matrices or molecule
    information files, used to parse 10x droplet data into per-sample UMI count matrices
  repo: https://github.com/MarioniLab/DropletUtils
- name: 10x Genomics
  role: Droplet scRNA-seq protocol whose molecule information files are the source
    data for parsing
- name: HiSeq 4000
  role: Sequencing instrument used for the droplet libraries being parsed; patterned
    flow-cell machine relevant to barcode swapping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:5eefda5776f46adacbff425896d91e77c92e3bd3acee7a1071090862affe8747
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - 10x Genomics
  - DropletUtils
  - HiSeq 4000
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# molecule-information-to-count-matrix-parsing

## Summary

Parse 10x Genomics molecule information files into per-sample UMI count matrices as the first step of droplet-based scRNA-seq processing, enabling downstream cell calling with emptyDrops. In this study it prepares the droplet-prepared mouse epithelial libraries (ArrayExpress E-MTAB-6854, sequenced on HiSeq 2500 and HiSeq 4000) for barcode-swapping and cell-calling analyses.

## When to use

Use when starting from 10x Genomics droplet molecule information (raw per-molecule records with cell barcode, UMI, gene) and you need per-sample UMI count matrices for cell calling or barcode-swapping exclusion.

## When NOT to use

- Input is already a gene-by-cell count matrix (e.g. a 10x matrix.mtx/h5 bundle); skip parsing and go directly to cell calling or exclusion.
- Data are plate-based (e.g. Smart-seq2 mouse T-cell plates from E-MTAB-6843); molecule information parsing for 10x droplet data does not apply.
- The analysis target is whole-transcript coverage outside exonic regions; the pipeline here counts only read pairs mapping to the exonic region of each gene.

## Inputs

- 10x Genomics molecule information files from droplet-prepared mouse epithelial libraries (ArrayExpress E-MTAB-6854, HiSeq 2500- and HiSeq 4000-sequenced)
- Sample demultiplexing information (expected sample barcodes) to assign molecules to per-sample matrices
- mm10 genome annotation for gene-level exonic counting

## Outputs

- Per-sample gene-by-barcode UMI count matrices (mm10 exonic gene counts per droplet barcode)
- Per-sample barcode sets restricted to non-shared barcodes for downstream emptyDrops cell calling

## How to apply

Download the droplet-prepared mouse epithelial cell data from ArrayExpress accession E-MTAB-6854 and parse the molecule information files into per-sample UMI count matrices using utilities from the DropletUtils R package, which supports loading from molecule information files. Reads were mapped to the mm10 genome, and the number of read pairs mapping to the exonic region of each gene were counted, so the count matrix rows are genes (mm10 exonic counts) and columns are droplet barcodes, one matrix per sample. Before or during cell calling, restrict to barcodes that are not shared between samples, per the cell exclusion strategy that removes all barcodes shared between samples. The resulting per-sample matrices are then ready for emptyDrops cell calling at an FDR threshold of 0.01 with a minimum cell size of 1000 UMIs.

## Related tools

- **DropletUtils** (R/Bioconductor package providing data loading from count matrices or molecule information files, used to parse 10x droplet data into per-sample UMI count matrices) — https://github.com/MarioniLab/DropletUtils
- **10x Genomics** (Droplet scRNA-seq protocol whose molecule information files are the source data for parsing)
- **HiSeq 4000** (Sequencing instrument used for the droplet libraries being parsed; patterned flow-cell machine relevant to barcode swapping)

## Examples

```
library(DropletUtils); counts <- read10xCounts("E-MTAB-6854_sample_molecule_info.h5")
```

## Evaluation signals

- Each parsed sample matrix has genes as rows (mm10 exonic genes) and droplet barcodes as columns, with integer UMI counts.
- One count matrix is produced per sample, and barcode sets across samples are checked for overlap; barcodes shared between samples are flagged for removal.
- Barcode counts per sample are consistent with expectations for 10x droplet libraries (many more barcodes than expected cells, ready for emptyDrops filtering).
- Downstream emptyDrops on the parsed matrices at FDR 0.01 / minimum 1000 UMIs yields called-cell counts and library-size distributions for every HiSeq 4000 sample in a well-formed one-row-per-sample table (emptyDrops_called_cells.tsv).

## Limitations

- Cell exclusion by removing all barcodes shared between samples can incorrectly discard genuine cell libraries; the fraction incorrectly discarded was counted in simulation.
- Approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000 due to barcode swapping, which can generate complex but artefactual cell libraries in droplet-based data if not excluded.
- The paper counts only read pairs mapping to the exonic region of each gene (mm10); intronic or ambiguous alignments are not captured in these matrices.

## Evidence

- [discussion] emptyDrops distinguishes cells from empty droplets...: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [discussion] Cell exclusion strategy: "We applied cell exclusion by removing all barcodes that were shared between samples"
- [discussion] Counting exonic reads against mm10: "Reads were mapped to the mm10 genome, and the number of read pairs mapping to the exonic region of each gene were counted."
- [discussion] Dataset availability: "and E-MTAB-6854 for droplet-prepared HiSeq 2500- and HiSeq 4000-sequenced mouse epithelial cells."
- [readme] DropletUtils capabilities: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
