---
name: cross-sample-cell-library-loss-quantification
description: Use when multiplexed 10x Genomics droplet data sequenced on a patterned
  flow-cell instrument (HiSeq 4000/X/NovaSeq) needs per-sample assessment of how many
  cell calls are artefactual after swapped-molecule removal.
when_to_use_negative:
- Data are plate-based scRNA-seq where molecule exclusion is not applicable — use
  the per-plate linear model / cell exclusion approaches instead
- Libraries were sequenced on non-patterned flow-cell instruments (e.g. HiSeq 2500),
  where swapping is negligible and cell-call loss after molecule exclusion would reflect
  noise rather than artefactual libraries
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DropletUtils
  role: R package used to load droplet count matrices and call cells with emptyDrops
    before and after molecule exclusion
  repo: https://github.com/MarioniLab/DropletUtils
- name: emptyDrops
  role: Cell-calling method applied with FDR threshold 0.01 and minimum cell size
    1000 UMIs on both uncorrected and molecule-excluded matrices
  repo: https://github.com/MarioniLab/DropletUtils
- name: HiSeq 4000
  role: Patterned flow-cell sequencer whose ~2.5% barcode-swapped read fraction produces
    the artefactual cell libraries being quantified
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:8532a1b5812d091380e3af79a0e1fb49c2f39cb6b53e62dccf4f9c47105b5045
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - DropletUtils
  - HiSeq 4000
  - emptyDrops
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cross-sample-cell-library-loss-quantification

## Summary

Quantifies how many called cells each multiplexed sample loses after removal of barcode-swapped molecules, exposing samples that consist almost entirely of artefactual swapped libraries (e.g. samples B1/B2 of the mouse epithelial HiSeq 4000 dataset).

## When to use

Use when multiplexed 10x Genomics droplet data sequenced on a patterned flow-cell instrument (HiSeq 4000/X/NovaSeq) needs per-sample assessment of how many cell calls are artefactual after swapped-molecule removal.

## When NOT to use

- Data are plate-based scRNA-seq where molecule exclusion is not applicable — use the per-plate linear model / cell exclusion approaches instead
- Libraries were sequenced on non-patterned flow-cell instruments (e.g. HiSeq 2500), where swapping is negligible and cell-call loss after molecule exclusion would reflect noise rather than artefactual libraries

## Inputs

- Multiplexed 10x Genomics droplet count matrices per sample (e.g. ArrayExpress E-MTAB-6854, HiSeq 2500- and HiSeq 4000-sequenced mouse epithelial cells)
- Molecule information: UMI, cell barcode, and aligned gene per read/molecule across samples

## Outputs

- Per-sample counts of called cells before and after molecule exclusion
- Number of called cells lost per sample (e.g. near-total loss for B1/B2)
- Fraction of cell libraries from other samples no longer called as cells (expected ~2.5% on HiSeq 4000)

## How to apply

Load the droplet count matrices (e.g. ArrayExpress E-MTAB-6854 mouse epithelial cells) with DropletUtils in R and call cells on the uncorrected data with emptyDrops at FDR 0.01 and a minimum cell size of 1000 UMIs. Remove molecules sharing the same UMI, cell barcode, and aligned gene combination across samples (molecule exclusion, assigning each ambiguous molecule to its majority sample of origin), then re-run emptyDrops with the same FDR 0.01 / 1000-UMI settings on the corrected matrices. Per sample, count called cells lost relative to baseline and compute the fraction of cell libraries from other samples no longer called as cells; a sample losing nearly all its called cells (B1/B2 pattern) indicates it was composed almost entirely of swapped libraries. The magnitude of loss in other samples (~2.5%) should match the independently estimated barcode-swapped read fraction for the HiSeq 4000.

## Related tools

- **DropletUtils** (R package used to load droplet count matrices and call cells with emptyDrops before and after molecule exclusion) — https://github.com/MarioniLab/DropletUtils
- **emptyDrops** (Cell-calling method applied with FDR threshold 0.01 and minimum cell size 1000 UMIs on both uncorrected and molecule-excluded matrices) — https://github.com/MarioniLab/DropletUtils
- **HiSeq 4000** (Patterned flow-cell sequencer whose ~2.5% barcode-swapped read fraction produces the artefactual cell libraries being quantified)

## Examples

```
library(DropletUtils); calls <- emptyDrops(counts, fdr.threshold=0.01, max.min=1000); is.cell <- which(calls$FDR <= 0.01 & calls$Total >= 1000)
```

## Evaluation signals

- Nearly all called cells are lost from the artefact-dominated samples (B1/B2) after molecule exclusion
- Approximately 2.5% of cell libraries from other samples are no longer called as cells, matching the reported HiSeq 4000 swapped read fraction
- emptyDrops parameters (FDR 0.01, minimum 1000 UMIs) are identical for baseline and corrected cell calls so the loss comparison is valid
- Corrected cell calls are reproducible when the molecule-exclusion step is re-run with the same 80% majority-sample-of-origin assignment

## Limitations

- Cell exclusion (removing all barcodes shared between samples) can incorrectly discard genuine cell libraries; the fraction incorrectly discarded was counted in simulations
- Detailed molecule exclusion method (majority-sample assignment) is described only in Supplementary Note 6, and the electronic supplementary material was absent from the document
- A high-throughput droplet protocol on an unswapped instrument (e.g. HiSeq 2500 control) should show no such cell-library loss; losses elsewhere indicate confounding factors

## Evidence

- [methods] calling cells with emptyDrops at FDR threshold 0.01 and minimum cell size 1000 UMIs: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [methods] Molecule exclusion removes molecules sharing the same UMI, cell barcode, and aligned gene combination: "removing molecules that share the same combination of UMI, cell barcode, and aligned gene"
- [results] Loss of nearly all called cells from samples B1/B2 indicates artefactual swapped libraries: [evidence span withheld — anchor status: not_found]
- [abstract] ~2.5% of reads mislabelled between samples on the HiSeq 4000: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000"
- [readme] DropletUtils provides utilities including removal of barcode-swapped pseudo-cells: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
