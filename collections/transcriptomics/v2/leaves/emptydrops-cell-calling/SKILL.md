---
name: emptydrops-cell-calling
description: Use when you have per-sample 10x-style UMI count matrices from droplet-prepared
  libraries (e.g., ArrayExpress E-MTAB-6854 mouse epithelial cells) and need to call
  which barcodes are real cells versus empty droplets, including re-calling after
  molecule-exclusion corrections.
when_to_use_negative:
- Data are plate-based (e.g., Smart-seq2 plate data such as E-MTAB-6843) rather than
  droplet-based, since emptyDrops models empty-droplet ambient profiles
- No raw droplet count/molecule information is available and barcodes cannot be tested
  against an ambient expression profile
- Barcodes have already been filtered by another cell-calling method and an ambient-profile
  re-test is not wanted
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DropletUtils
  role: R/Bioconductor package providing emptyDrops for distinguishing cells from
    empty droplets, plus molecule-information loading and swapped-molecule removal
    via swappedDrops
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Analysis environment in which DropletUtils/emptyDrops is run on the per-sample
    UMI count matrices
- name: 10x Genomics
  role: Droplet protocol whose molecule information files are parsed into the UMI
    count matrices that emptyDrops is applied to
provenance:
  source_task_ids:
  - task_007
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:85fc9be5f761b408662e70f963796f1f427a4140afe3c35a7bba9148fbf3d809
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - 10x Genomics
  - DropletUtils
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# emptydrops-cell-calling

## Summary

Distinguishes real cells from empty droplets in 10x Genomics droplet scRNA-seq count matrices using the emptyDrops algorithm from the DropletUtils R package, testing each barcode's expression profile for significant deviation from the ambient RNA profile. Used here with an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs to baseline and re-assess cell calls before and after swapped-molecule removal.

## When to use

Use when you have per-sample 10x-style UMI count matrices from droplet-prepared libraries (e.g., ArrayExpress E-MTAB-6854 mouse epithelial cells) and need to call which barcodes are real cells versus empty droplets, including re-calling after molecule-exclusion corrections.

## When NOT to use

- Data are plate-based (e.g., Smart-seq2 plate data such as E-MTAB-6843) rather than droplet-based, since emptyDrops models empty-droplet ambient profiles
- No raw droplet count/molecule information is available and barcodes cannot be tested against an ambient expression profile
- Barcodes have already been filtered by another cell-calling method and an ambient-profile re-test is not wanted

## Inputs

- 10x-style per-sample UMI count matrices derived from molecule information files (E-MTAB-6854 droplet mouse epithelial data, mm10-mapped exonic read-pair counts)
- emptyDrops parameters: FDR threshold 0.01, minimum cell size 1000 UMIs
- Optional: pre/post molecule-exclusion count matrices for before/after comparison

## Outputs

- Per-sample called-cell counts (barcodes passing emptyDrops FDR 0.01 and minimum 1000 UMIs)
- Per-sample distribution of total UMI counts (library sizes) of called cells
- Summary table (e.g., emptyDrops_called_cells.tsv) with one row per sample
- Comparison of cell calls before vs. after swapped-molecule removal (cells lost per sample)

## How to apply

Parse the 10x molecule information into per-sample UMI count matrices (reads mapped to mm10, exonic read pairs counted per gene). Restrict to barcodes that are not shared between samples, consistent with the cell-exclusion strategy that removes all barcodes shared between samples. Run emptyDrops on each sample's count matrix with an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs; emptyDrops tests each barcode's expression profile for significant deviation from the ambient profile and automatically retains very large-count droplets. After applying swapped-molecule removal (molecules sharing the same UMI, cell barcode, and aligned gene across samples, assigned to the majority 80% sample of origin), re-run emptyDrops with the same thresholds on the corrected matrices. Summarize called-cell counts and library-size (total UMI) distributions per sample and verify expected differences, e.g., loss of nearly all called cells in swapped-affected samples B1/B2 versus a negligible removed fraction (<0.0005%) on HiSeq 2500 non-multiplexed controls.

## Related tools

- **DropletUtils** (R/Bioconductor package providing emptyDrops for distinguishing cells from empty droplets, plus molecule-information loading and swapped-molecule removal via swappedDrops) — https://github.com/MarioniLab/DropletUtils
- **R** (Analysis environment in which DropletUtils/emptyDrops is run on the per-sample UMI count matrices)
- **10x Genomics** (Droplet protocol whose molecule information files are parsed into the UMI count matrices that emptyDrops is applied to)

## Examples

```
library(DropletUtils); called <- emptyDrops(counts, lower=1000, fdr.threshold=0.01); is.cell <- called$FDR <= 0.01 | is.na(called$FDR)
```

## Evaluation signals

- The output table reports called-cell counts and library-size distributions for every sample using exactly FDR 0.01 and minimum cell size 1000 UMIs, with one row per sample and a well-formed schema (e.g., emptyDrops_called_cells.tsv).
- Very large-count droplets are automatically retained as cells, per emptyDrops behaviour.
- On corrected (molecule-excluded) HiSeq 4000 matrices, nearly all called cells are lost from samples B1 and B2, consistent with almost-entirely artefactual swapped libraries.
- Approximately 2.5% of cell libraries from other HiSeq 4000 samples are no longer called as cells after exclusion, matching the ~2.5% swapped-read fraction.
- On the HiSeq 2500 non-multiplexed control, the removed molecule fraction is below 0.0005% per sample, confirming negligible removal.

## Limitations

- Cell exclusion by removing all barcodes shared between samples can incorrectly discard genuine cell libraries; the fraction incorrectly discarded was counted in the paper's simulation.
- emptyDrops relies on a well-estimated ambient profile; results depend on the chosen FDR threshold (0.01) and minimum cell size (1000 UMIs).
- Called-cell counts change substantially when applied after swapped-molecule removal, so baseline calls on uncorrected data may be misleading for swapped-affected samples (e.g., B1/B2).

## Evidence

- [results] emptyDrops distinguishes cells from empty droplets by testing each barcode's expression profile for significant deviation from the ambient profile: [evidence span withheld — anchor status: not_found]
- [methods] Cell calling parameters: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [results] droplets with significant deviations are called as cells at a specified FDR threshold (e.g., below 1%), with very large-count droplets automatically retained.: [evidence span withheld — anchor status: not_found]
- [readme] DropletUtils package scope: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
- [discussion] Cell exclusion strategy: "We applied cell exclusion by removing all barcodes that were shared between samples, and counted the fraction of cell libraries that were incorrectly discarded."
