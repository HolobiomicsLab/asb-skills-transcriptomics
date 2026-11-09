---
name: umi-cellbarcode-gene-collision-resolution
description: Use when processing molecule information files from multiplexed 10x Genomics
  droplet libraries sequenced on patterned flow-cell instruments (HiSeq 4000, HiSeq
  X, NovaSeq) where ~2.5% of reads may be mislabelled between samples via barcode
  swapping.
when_to_use_negative:
- Libraries were sequenced on non-patterned-flowcell instruments such as the HiSeq
  2500, where swapping is not elevated — the paper used HiSeq 2500 sequencing precisely
  as a negative control
- The goal is plate-based (Smart-seq2) scRNA-seq swapping quantification, which the
  paper handles with regression-based estimation rather than molecule exclusion
- A coarse cell-exclusion strategy is acceptable and discarding entire cell libraries
  sharing barcodes between samples is tolerable
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DropletUtils
  role: R/Bioconductor package whose swappedDrops function implements the molecule-exclusion
    algorithm on molecule information files from 10x Genomics droplet data
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Analysis language in which the swapping detection and count-matrix cleaning
    is performed
- name: 10x Genomics
  role: Droplet scRNA-seq protocol whose molecule information files (UMI, cell barcode,
    gene) are the algorithm's input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:93c9abb6fb8986ca5a6f4281adfc65c3434f983ce77460db6176704cafc45ab1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - 10x Genomics
  - DropletUtils
  - R
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# umi-cellbarcode-gene-collision-resolution

## Summary

A molecule-level collision resolution procedure (implemented as swappedDrops in DropletUtils) that detects barcode-swapped molecules in multiplexed 10x Genomics droplet scRNA-seq data by grouping molecules sharing the same UMI, cell barcode, and aligned gene across samples, then reassigning or removing them based on a minimum read-fraction threshold.

## When to use

Use when processing molecule information files from multiplexed 10x Genomics droplet libraries sequenced on patterned flow-cell instruments (HiSeq 4000, HiSeq X, NovaSeq) where ~2.5% of reads may be mislabelled between samples via barcode swapping.

## When NOT to use

- Libraries were sequenced on non-patterned-flowcell instruments such as the HiSeq 2500, where swapping is not elevated — the paper used HiSeq 2500 sequencing precisely as a negative control
- The goal is plate-based (Smart-seq2) scRNA-seq swapping quantification, which the paper handles with regression-based estimation rather than molecule exclusion
- A coarse cell-exclusion strategy is acceptable and discarding entire cell libraries sharing barcodes between samples is tolerable

## Inputs

- Per-sample 10x Genomics molecule information files (UMI, cell barcode, aligned gene) from droplet libraries multiplexed on a HiSeq 4000 run
- min.frac threshold parameter (default 0.9; paper used 0.8)

## Outputs

- Cleaned per-sample sparse count matrices with swapped molecules reassigned or removed (e.g. cleaned_count_matrices.rds)

## How to apply

Load the 10x molecule information (UMI, cell barcode, gene) for each multiplexed droplet sample sequenced on the HiSeq 4000 (and HiSeq 2500 control). Group molecules across samples by identical UMI / cell-barcode / aligned-gene combination keys to find candidate swapping events. For each ambiguous combination, compute the fraction of reads contributed by each sample; if a single sample holds at least the min.frac threshold (DropletUtils default 0.9; the paper used 0.8), reassign the molecule to that sample-of-origin and remove its count from other samples. Molecules that cannot be confidently attributed (no sample meets min.frac) are removed from every sample. This molecule-level strategy avoids the cell-exclusion alternative, which incorrectly discards entire cell libraries sharing barcodes between samples. Save the cleaned per-sample sparse count matrices (e.g. cleaned_count_matrices.rds).

## Related tools

- **DropletUtils** (R/Bioconductor package whose swappedDrops function implements the molecule-exclusion algorithm on molecule information files from 10x Genomics droplet data) — https://github.com/MarioniLab/DropletUtils
- **R** (Analysis language in which the swapping detection and count-matrix cleaning is performed)
- **10x Genomics** (Droplet scRNA-seq protocol whose molecule information files (UMI, cell barcode, gene) are the algorithm's input)

## Examples

```
library(DropletUtils); cleaned <- swappedDrops(mol.info.files, min.frac=0.8); saveRDS(cleaned$cleaned, "cleaned_count_matrices.rds")
```

## Evaluation signals

- Swapped fraction of reads on HiSeq 4000 data should be reproducibly around 2.5%, consistent with the paper's estimate
- Same libraries sequenced on a HiSeq 2500 control should show no excess of ambiguous UMI/cell-barcode/gene collisions across samples
- Every molecule assigned across samples must have a dominant sample contributing at least the min.frac fraction of its reads; unattributable molecules must be absent from all cleaned matrices
- Cleaned per-sample matrices should contain no molecule (UMI/cell-barcode/gene key) shared across more than one sample
- Compared to cell exclusion, molecule exclusion should retain cell libraries that would otherwise be incorrectly discarded for sharing barcodes between samples

## Limitations

- Molecules whose reads cannot be confidently attributed to one sample (no sample meets min.frac) are discarded from every sample
- Key method details for molecule exclusion were deferred to Supplementary Note 6, which is absent from the available electronic supplementary material
- The threshold trades off sensitivity and specificity: the paper used 0.8 while the DropletUtils default is 0.9, so the choice of min.frac affects how many ambiguous molecules are reassigned versus removed

## Evidence

- [abstract] an algorithm to exclude individual molecules that have swapped between samples in 10x Genomics experiments: "we have developed an algorithm to exclude individual molecules that have swapped between samples in 10x Genomics experiments"
- [discussion] molecule sharing criterion: "removing molecules that share the same combination of UMI, cell barcode, and aligned gene"
- [discussion] cell exclusion downside: "We applied cell exclusion by removing all barcodes that were shared between samples, and counted the fraction of cell libraries that were incorrectly discarded."
- [abstract] mislabelled read fraction: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000, which is lower than previous reports"
- [readme] DropletUtils scope: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
