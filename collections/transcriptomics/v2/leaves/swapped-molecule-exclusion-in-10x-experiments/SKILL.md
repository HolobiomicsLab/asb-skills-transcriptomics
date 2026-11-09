---
name: swapped-molecule-exclusion-in-10x-experiments
description: Use when running a negative control for the swapped-molecule removal
  algorithm on non-multiplexed 10x Genomics droplet datasets, or when validating that
  molecule exclusion does not over-remove on HiSeq 2500-sequenced data.
when_to_use_negative:
- Data are multiplexed samples sequenced on a patterned flow-cell instrument (HiSeq
  4000, HiSeq X, NovaSeq), where substantial swapping (~2.5% of reads) is expected
  and the algorithm is applied as a correction rather than a control
- Data are plate-based (e.g. Smart-seq2) rather than 10x Genomics droplet experiments,
  where molecule-level exclusion does not apply and cell-level exclusion is used instead
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DropletUtils
  role: R/Bioconductor package implementing emptyDrops cell calling and the swapped-molecule
    exclusion algorithm (removal of molecules sharing UMI, cell barcode, and aligned
    gene)
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Language in which the counting, cell calling, and molecule exclusion pipeline
    is executed
- name: HiSeq 2500
  role: Non-patterned-flow-cell sequencer used for the control libraries, expected
    to show negligible barcode swapping
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:e56d17d1b24f69145c5982969158e51b6430d38937d24217ba6842028d736c49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - DropletUtils
  - HiSeq 2500
  - R
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# swapped-molecule-exclusion-in-10x-experiments

## Summary

A computational control procedure that applies the barcode-swapping molecule exclusion algorithm (implemented in DropletUtils) to non-multiplexed 10x Genomics droplet scRNA-seq experiments, verifying that it removes only a negligible fraction of molecules (below 0.0005%) when sequenced on the non-patterned-flow-cell HiSeq 2500.

## When to use

Use when running a negative control for the swapped-molecule removal algorithm on non-multiplexed 10x Genomics droplet datasets, or when validating that molecule exclusion does not over-remove on HiSeq 2500-sequenced data.

## When NOT to use

- Data are multiplexed samples sequenced on a patterned flow-cell instrument (HiSeq 4000, HiSeq X, NovaSeq), where substantial swapping (~2.5% of reads) is expected and the algorithm is applied as a correction rather than a control
- Data are plate-based (e.g. Smart-seq2) rather than 10x Genomics droplet experiments, where molecule-level exclusion does not apply and cell-level exclusion is used instead

## Inputs

- Raw FASTQ data from non-multiplexed droplet-prepared 10x Genomics mouse epithelial experiments (ArrayExpress E-MTAB-6854, HiSeq 2500-sequenced)
- mm10 reference genome for read mapping
- UMI count matrices / molecule information (UMI, cell barcode, aligned gene) per sample

## Outputs

- 10x-style UMI count matrices per sample after emptyDrops cell calling
- swapped_molecule_removal_results.csv: summary table of number and fraction of molecules removed per sample
- Validation check that removed molecule fraction is below 0.0005% per sample

## How to apply

Obtain raw droplet-prepared mouse epithelial data for the non-multiplexed 10x experiments from ArrayExpress accession E-MTAB-6854 (HiSeq 2500-sequenced libraries). Map reads to the mm10 genome and count exonic read pairs per gene to build 10x-style UMI count matrices for each sample. Call cells using emptyDrops with an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs. Then apply the molecule exclusion algorithm, which removes molecules sharing the same combination of UMI, cell barcode, and aligned gene across samples, as implemented in DropletUtils. Record the number and fraction of molecules removed per sample; because barcode swapping arises on patterned flow-cell instruments (HiSeq 4000, HiSeq X, NovaSeq), the removal fraction on HiSeq 2500 data should be negligible.

## Related tools

- **DropletUtils** (R/Bioconductor package implementing emptyDrops cell calling and the swapped-molecule exclusion algorithm (removal of molecules sharing UMI, cell barcode, and aligned gene)) — https://github.com/MarioniLab/DropletUtils
- **R** (Language in which the counting, cell calling, and molecule exclusion pipeline is executed)
- **HiSeq 2500** (Non-patterned-flow-cell sequencer used for the control libraries, expected to show negligible barcode swapping)

## Examples

```
Rscript -e 'library(DropletUtils); removed <- swappedDrops(mol.info.files, min.gene=1, force.proportion=0.5)'
```

## Evaluation signals

- Removed molecule fraction per HiSeq 2500 non-multiplexed sample is below 0.0005%
- Cell calling uses emptyDrops with FDR threshold 0.01 and minimum cell size 1000 UMIs as specified
- Reads are mapped to mm10 and only exonic read pairs per gene are counted in the UMI matrices
- Summary table swapped_molecule_removal_results.csv is produced with per-sample removal counts and fractions

## Limitations

- The control is only valid for non-multiplexed libraries on non-patterned flow-cell instruments; on HiSeq 4000 data the algorithm removes a non-negligible fraction due to ~2.5% read mislabelling
- The cell-exclusion strategy (removing all barcodes shared between samples) can incorrectly discard cell libraries, which is why molecule-level exclusion is preferred
- Detailed molecule exclusion method parameters are deferred to Supplementary Note 6 and unavailable in the main text

## Evidence

- [discussion] As a control, we repeated this procedure with the same libraries sequenced on HiSeq 2500.: "As a control, we repeated this procedure with the same libraries sequenced on HiSeq 2500."
- [discussion] Molecule exclusion definition: "removing molecules that share the same combination of UMI, cell barcode, and aligned gene"
- [discussion] Cell calling parameters: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [discussion] Dataset accession: "and E-MTAB-6854 for droplet-prepared HiSeq 2500- and HiSeq 4000-sequenced mouse epithelial cells."
- [readme] README DropletUtils capabilities: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
