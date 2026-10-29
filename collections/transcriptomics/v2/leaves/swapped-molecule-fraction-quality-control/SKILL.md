---
name: swapped-molecule-fraction-quality-control
description: Use when running the swapped-molecule removal algorithm as a control
  on non-multiplexed 10x Genomics droplet scRNA-seq samples (e.g. HiSeq 2500-sequenced
  mouse epithelial cells from ArrayExpress E-MTAB-6854) to verify that molecule removal
  is negligible.
when_to_use_negative:
- Data are multiplexed samples sequenced on patterned flow-cell instruments (HiSeq
  4000, HiSeq X, NovaSeq), where substantial swapping (~2.5% of reads) is expected
  and the removal step is a corrective analysis, not a negative control.
- Data are plate-based scRNA-seq (e.g. Smart-seq2 plate data), for which the paper
  uses cell exclusion and regression-based swapping-fraction estimation rather than
  molecule exclusion.
- Input count matrices lack UMI/cell-barcode/gene molecule-level information, so the
  same-UMI/cell-barcode/gene combination criterion cannot be evaluated.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DropletUtils
  role: Implements the molecule exclusion algorithm and emptyDrops cell calling used
    for the swapped-molecule removal control
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Language in which the control pipeline (mapping outputs, emptyDrops, molecule
    removal, results table) is executed
- name: HiSeq 2500
  role: Non-patterned flow-cell sequencer providing the non-multiplexed control libraries
    where negligible swapping is expected
- name: HiSeq 4000
  role: Patterned flow-cell instrument against whose elevated swapping rates the HiSeq
    2500 control is contrasted
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:59ac661ce951ea0ec84fa0f5d78b6dd0ef2d7dd807aefc2b061083937fa0b1e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - DropletUtils
  - HiSeq 2500
  - HiSeq 4000
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# swapped-molecule-fraction-quality-control

## Summary

A negative-control quality-control procedure that runs the barcode-swapping molecule exclusion algorithm on non-multiplexed 10x Genomics droplet data sequenced on the HiSeq 2500, where negligible molecule removal (<0.0005%) is expected and confirms the removal step targets patterned flow-cell artefacts rather than genuine molecules.

## When to use

Use when running the swapped-molecule removal algorithm as a control on non-multiplexed 10x Genomics droplet scRNA-seq samples (e.g. HiSeq 2500-sequenced mouse epithelial cells from ArrayExpress E-MTAB-6854) to verify that molecule removal is negligible.

## When NOT to use

- Data are multiplexed samples sequenced on patterned flow-cell instruments (HiSeq 4000, HiSeq X, NovaSeq), where substantial swapping (~2.5% of reads) is expected and the removal step is a corrective analysis, not a negative control.
- Data are plate-based scRNA-seq (e.g. Smart-seq2 plate data), for which the paper uses cell exclusion and regression-based swapping-fraction estimation rather than molecule exclusion.
- Input count matrices lack UMI/cell-barcode/gene molecule-level information, so the same-UMI/cell-barcode/gene combination criterion cannot be evaluated.

## Inputs

- Raw FASTQ/bcl data from two non-multiplexed 10x Genomics mouse epithelial experiments sequenced on the HiSeq 2500 (ArrayExpress E-MTAB-6854)
- mm10 reference genome for read mapping
- DropletUtils molecule information files per sample (UMI, cell barcode, gene assignment per molecule)

## Outputs

- 10x-style UMI count matrices per sample (mm10 exonic read-pair counts per gene)
- Set of cell calls per sample from emptyDrops (FDR 0.01, min 1000 UMIs)
- Per-sample count and fraction of molecules removed by the swapping algorithm
- Summary table written to swapped_molecule_removal_results.csv

## How to apply

Obtain raw droplet-prepared data for two non-multiplexed 10x Genomics mouse epithelial experiments sequenced on the HiSeq 2500 (ArrayExpress E-MTAB-6854). Map reads to the mm10 genome and count exonic read pairs per gene to build 10x-style UMI count matrices, then call cells with emptyDrops using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs. Apply the molecule exclusion algorithm as implemented in DropletUtils, removing molecules that share the same combination of UMI, cell barcode, and aligned gene across the samples. Record the number and fraction of molecules removed per sample and validate that the removed fraction is below 0.0005%; a larger fraction would indicate the control failed and the removal step may be discarding genuine molecules.

## Related tools

- **DropletUtils** (Implements the molecule exclusion algorithm and emptyDrops cell calling used for the swapped-molecule removal control) — https://github.com/MarioniLab/DropletUtils
- **R** (Language in which the control pipeline (mapping outputs, emptyDrops, molecule removal, results table) is executed)
- **HiSeq 2500** (Non-patterned flow-cell sequencer providing the non-multiplexed control libraries where negligible swapping is expected)
- **HiSeq 4000** (Patterned flow-cell instrument against whose elevated swapping rates the HiSeq 2500 control is contrasted)

## Examples

```
Rscript -e 'library(DropletUtils); out <- swappedDrops(mol.info(c("sample1.mol.h5","sample2.mol.h5")))'
```

## Evaluation signals

- The reported removed-molecule fraction for each HiSeq 2500 non-multiplexed sample is below 0.0005%.
- emptyDrops cell calling used FDR threshold 0.01 and minimum cell size 1000 UMIs for every sample.
- Removed molecules all share the same combination of UMI, cell barcode, and aligned gene across samples.
- A per-sample summary table (swapped_molecule_removal_results.csv) exists with number and fraction of molecules removed for both samples.
- Removed molecule counts are orders of magnitude lower than the ~2.5% swapped-read fraction observed on the HiSeq 4000.

## Limitations

- The control applies only to libraries sequenced on non-patterned flow-cell machines (HiSeq 2500); it cannot demonstrate algorithm behaviour on HiSeq 4000/HiSeq X/NovaSeq data where swapping is prevalent.
- Molecule exclusion method details are deferred to Supplementary Note 6, and the electronic supplementary material is unavailable ('No changelog found.'), so exact implementation specifics are not fully recoverable from the main text.
- The algorithm depends on molecule information files with UMI, cell barcode, and gene assignment; datasets lacking this granularity cannot be processed.

## Evidence

- [discussion] Molecule exclusion: removal of molecules sharing the same UMI, cell barcode, and aligned gene combination: "removing molecules that share the same combination of UMI, cell barcode, and aligned gene"
- [discussion] Cell calling parameters for emptyDrops: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [discussion] Control on HiSeq 2500-sequenced libraries: "As a control, we repeated this procedure with the same libraries sequenced on HiSeq 2500."
- [readme] DropletUtils capabilities supporting the control: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
- [discussion] Source of the control datasets: "E-MTAB-6854 for droplet-prepared HiSeq 2500- and HiSeq 4000-sequenced mouse epithelial cells."
