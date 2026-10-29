---
name: swapped-molecule-fraction-thresholding
description: Use when multiplexed 10x Genomics droplet libraries (e.g. sequenced on
  patterned flow-cell HiSeq 4000) share molecule information files and you must remove
  or reassign reads mislabelled by barcode swapping (~2.5% on HiSeq 4000).
when_to_use_negative:
- Data were sequenced on non-patterned-flowcell instruments (e.g. HiSeq 2500), where
  swapping is negligible and the control in the paper showed minimal mislabelling
- Input is plate-based Smart-seq2 scRNA-seq data — use the plate-level swapping-fraction
  regression approach instead of molecule exclusion
- Only raw FASTQ is available and molecule information files have not been generated
  (barcodes/UMIs not yet extracted and aligned)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DropletUtils
  role: Provides swappedDrops for reading molecule information files, identifying
    barcode-swapped molecules across multiplexed samples, and returning cleaned count
    matrices
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Implementation environment for the molecule-exclusion algorithm and downstream
    scran/Rtsne analyses
- name: 10x Genomics
  role: Droplet scRNA-seq platform whose molecule information files supply the UMI/cell-barcode/gene
    keys grouped across samples
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:cd6c635fe7b38ac8b234645e4bbdae6f4affe0e548f8164e1ae37421639236e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - 10x Genomics
  - DropletUtils
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# swapped-molecule-fraction-thresholding

## Summary

A molecule-level correction for barcode swapping in multiplexed 10x Genomics droplet scRNA-seq: molecules sharing an identical UMI/cell-barcode/aligned-gene key across samples are reassigned to the majority sample (via a min.frac read-fraction threshold, default 0.9; the paper used 0.8) or excluded entirely.

## When to use

Use when multiplexed 10x Genomics droplet libraries (e.g. sequenced on patterned flow-cell HiSeq 4000) share molecule information files and you must remove or reassign reads mislabelled by barcode swapping (~2.5% on HiSeq 4000).

## When NOT to use

- Data were sequenced on non-patterned-flowcell instruments (e.g. HiSeq 2500), where swapping is negligible and the control in the paper showed minimal mislabelling
- Input is plate-based Smart-seq2 scRNA-seq data — use the plate-level swapping-fraction regression approach instead of molecule exclusion
- Only raw FASTQ is available and molecule information files have not been generated (barcodes/UMIs not yet extracted and aligned)

## Inputs

- 10x Genomics molecule information files (UMI, cell barcode, aligned gene per molecule) for each multiplexed droplet sample
- Per-sample sparse count matrices from 10x pipelines
- min.frac threshold parameter (default 0.9; paper used 0.8)

## Outputs

- Cleaned per-sample sparse count matrices with swapped molecules reassigned or removed (e.g. saved as cleaned_count_matrices.rds)
- Vector of sample-of-origin assignments for each called cell

## How to apply

Load the 10x molecule information files (UMI, cell barcode, gene) for each multiplexed droplet sample, then group molecules across samples by identical UMI/cell-barcode/aligned-gene combinations to find candidate swapping events. For each ambiguous combination, compute the fraction of reads contributed by each sample; if a single sample contributes at least the min.frac threshold (default 0.9, or 0.8 as in the paper), reassign the molecule to that sample-of-origin and remove its count from all other samples. Molecules with no sample meeting min.frac are discarded from every sample, which avoids the coarser cell-exclusion strategy that incorrectly discards entire cell libraries. Call cells beforehand with emptyDrops at an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs, and save the cleaned per-sample sparse matrices (e.g. to cleaned_count_matrices.rds).

## Related tools

- **DropletUtils** (Provides swappedDrops for reading molecule information files, identifying barcode-swapped molecules across multiplexed samples, and returning cleaned count matrices) — https://github.com/MarioniLab/DropletUtils
- **R** (Implementation environment for the molecule-exclusion algorithm and downstream scran/Rtsne analyses)
- **10x Genomics** (Droplet scRNA-seq platform whose molecule information files supply the UMI/cell-barcode/gene keys grouped across samples)

## Examples

```
library(DropletUtils); out <- swappedDrops(mol.info.files, min.frac=0.9); saveRDS(out$cleaned, "cleaned_count_matrices.rds")
```

## Evaluation signals

- Residual swapped reads: after correction, no molecule key (UMI/cell barcode/gene) should be shared across samples in the cleaned matrices
- Sensitivity to min.frac: with min.frac=0.8–0.9, few ambiguous molecules should remain unattributed; a large discarded fraction suggests an unexpected level of genuine multi-sample barcodes
- Control comparison: re-running on libraries sequenced on the HiSeq 2500 should yield near-zero detected swapping, as in the paper
- Count conservation: reads from majority-holding molecules are reassigned, not lost, so total UMI counts for confident molecules should be preserved up to the unattributed exclusions
- Cell calling is done first with emptyDrops at FDR 0.01 and minimum 1000 UMIs; verify called-cell sets match those expectations

## Limitations

- Molecules whose reads cannot be confidently attributed (no sample meets min.frac) are removed from every sample, discarding data
- The method targets patterned flow-cell instruments (HiSeq 4000, HiSeq X, NovaSeq); it is unnecessary for non-swapping platforms
- Cell exclusion (removing all barcodes shared between samples) is the alternative strategy but incorrectly discards cell libraries; molecule exclusion is preferred but details were deferred to Supplementary Note 6, which is unavailable in the extracted text

## Evidence

- [discussion] Molecule exclusion by shared UMI/cell-barcode/gene keys: "removing molecules that share the same combination of UMI, cell barcode, and aligned gene"
- [abstract] Algorithm purpose stated in abstract: "an algorithm to exclude individual molecules that have swapped between samples in 10x Genomics experiments"
- [discussion] emptyDrops cell calling parameters applied before exclusion: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [readme] DropletUtils capabilities from README: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
