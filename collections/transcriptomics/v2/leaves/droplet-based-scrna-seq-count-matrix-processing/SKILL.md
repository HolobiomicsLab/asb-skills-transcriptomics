---
name: droplet-based-scrna-seq-count-matrix-processing
description: Use when processing raw droplet-prepared 10x Genomics mouse data (e.g.
  ArrayExpress E-MTAB-6854) into UMI count matrices and defensible cell calls, or
  when needing to correct for cross-sample barcode swapping on patterned flow-cell
  instruments such as the HiSeq 4000.
when_to_use_negative:
- Data are plate-based (Smart-seq2) scRNA-seq rather than droplet/UMI-based 10x Genomics
  data
- Libraries were sequenced on a non-patterned flow-cell machine (e.g. HiSeq 2500),
  where swapped-read removal is expected to remove a negligible fraction and is only
  useful as a control
- The goal is single-sample processing with no multiplexed samples, so cross-sample
  molecule exclusion is not applicable
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: DropletUtils
  role: R/Bioconductor package used to load count matrices or molecule information
    files, call cells with emptyDrops (FDR 0.01, min 1000 UMIs), and remove barcode-swapped
    pseudo-cells via molecule exclusion
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Analysis environment for building UMI count matrices, running emptyDrops cell
    calling, and computing per-sample removal statistics
- name: HiSeq 2500
  role: Non-patterned-flow-cell sequencing control instrument; libraries re-sequenced
    on it provide the negative control where swapped-read removal should remove below
    0.0005% of molecules
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:e3bb9c74b11b508e4635f2175ccab7e10ca71a87fcf0e88b87f26ca227ea2d23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - DropletUtils
  - HiSeq 2500
  - R
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# droplet-based-scrna-seq-count-matrix-processing

## Summary

Process 10x Genomics droplet scRNA-seq data by mapping reads to mm10, counting exonic read pairs per gene into UMI count matrices, calling real cells with emptyDrops (FDR 0.01, minimum 1000 UMIs), and optionally removing barcode-swapped molecules shared across samples using DropletUtils.

## When to use

Use when processing raw droplet-prepared 10x Genomics mouse data (e.g. ArrayExpress E-MTAB-6854) into UMI count matrices and defensible cell calls, or when needing to correct for cross-sample barcode swapping on patterned flow-cell instruments such as the HiSeq 4000.

## When NOT to use

- Data are plate-based (Smart-seq2) scRNA-seq rather than droplet/UMI-based 10x Genomics data
- Libraries were sequenced on a non-patterned flow-cell machine (e.g. HiSeq 2500), where swapped-read removal is expected to remove a negligible fraction and is only useful as a control
- The goal is single-sample processing with no multiplexed samples, so cross-sample molecule exclusion is not applicable

## Inputs

- Raw droplet-prepared 10x Genomics FASTQ data (e.g. ArrayExpress E-MTAB-6854 mouse epithelial cells)
- Reference genome (mm10) for read alignment
- UMI count matrices per sample (gene x cell-barcode matrices)
- Molecule information (UMI, cell barcode, aligned gene, sample of origin) for molecule-level exclusion

## Outputs

- 10x-style gene-by-cell UMI count matrix per sample (exonic read-pair counts on mm10)
- Called-cell set per sample from emptyDrops (FDR <= 0.01, minimum 1000 UMIs)
- Molecule-excluded count matrices with swapped molecules removed across samples
- Summary statistics of cells lost and fraction of molecules removed per sample (e.g. swapped_molecule_removal_results.csv)

## How to apply

Obtain raw droplet-prepared FASTQ data (e.g. mouse epithelial cells from ArrayExpress E-MTAB-6854) and map reads to the mm10 genome, counting read pairs mapping to the exonic region of each gene to build 10x-style UMI count matrices per sample. Load the matrices into R with the DropletUtils package and call cells using emptyDrops with an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs to establish a baseline. If samples are multiplexed on a patterned flow-cell machine (HiSeq 4000), apply molecule exclusion — removing molecules sharing the same combination of UMI, cell barcode, and aligned gene across samples, assigning ambiguous molecules to their majority (80%) sample of origin — then re-run emptyDrops with the same parameters to obtain corrected cell calls. The rationale is that barcode swapping on the HiSeq 4000 mislabels approximately 2.5% of reads and can generate complex but artefactual cell libraries that must be excluded before downstream analysis.

## Related tools

- **DropletUtils** (R/Bioconductor package used to load count matrices or molecule information files, call cells with emptyDrops (FDR 0.01, min 1000 UMIs), and remove barcode-swapped pseudo-cells via molecule exclusion) — https://github.com/MarioniLab/DropletUtils
- **R** (Analysis environment for building UMI count matrices, running emptyDrops cell calling, and computing per-sample removal statistics)
- **HiSeq 2500** (Non-patterned-flow-cell sequencing control instrument; libraries re-sequenced on it provide the negative control where swapped-read removal should remove below 0.0005% of molecules)

## Examples

```
out <- DropletUtils::emptyDrops(m); cells <- rownames(out)[which(out$FDR <= 0.01 & out$Total >= 1000)]
```

## Evaluation signals

- emptyDrops cell calls are reproducible with FDR threshold 0.01 and minimum cell size 1000 UMIs on both uncorrected and corrected matrices
- When the molecule-exclusion control is applied to non-multiplexed HiSeq 2500-sequenced samples, the removed molecule fraction is below 0.0005%
- When applied to HiSeq 4000-sequenced multiplexed samples, nearly all called cells are lost from samples B1/B2 and approximately 2.5% of cell libraries from other samples are no longer called as cells
- Per-sample removed-molecule fractions and cell-loss counts are written to a summary table (e.g. swapped_molecule_removal_results.csv) that matches the expected magnitudes
- Swapped-read fraction on HiSeq 4000 data is consistent with the reported ~2.5% estimate

## Limitations

- Cell exclusion by removing all barcodes shared between samples can incorrectly discard genuine cell libraries; the molecule-level exclusion strategy is preferred for 10x Genomics data
- Barcodes from different samples can be legitimately present at the expected droplet frequencies, complicating swapped-library identification
- The swapping fraction (~2.5%) was estimated for the HiSeq 4000; other patterned flow-cell machines (HiSeq X, NovaSeq) exhibit swapping but rates may differ
- Molecule exclusion details are deferred to Supplementary Note 6, which may not be available in the main text

## Evidence

- [discussion] emptyDrops cell calling parameters: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [discussion] molecule exclusion criterion: "removing molecules that share the same combination of UMI, cell barcode, and aligned gene"
- [discussion] count matrix construction: "Reads were mapped to the mm10 genome, and the number of read pairs mapping to the exonic region of each gene were counted."
- [readme] DropletUtils scope: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
