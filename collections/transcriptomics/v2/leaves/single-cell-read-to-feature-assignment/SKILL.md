---
name: single-cell-read-to-feature-assignment
description: Use when quantifying TE expression or accessibility from single-cell
  RNA-seq or ATAC-seq alignments (unfiltered BAM/SAM with CB/CR and UB/UR barcode/UMI
  tags) and a cell-by-feature count matrix is needed.
when_to_use_negative:
- Input is already a gene/TE expression or accessibility feature table rather than
  aligned BAM/SAM reads.
- The analysis goal is bulk RNA-seq counting without cell barcodes/UMIs — scTE's demultiplexing
  by CB/UB tags assumes 10x-style single-cell data.
- Custom pre-built indices for the genome assembly in use are unavailable and only
  BED/GTF inputs incompatible with scTE_build's six-column BED requirement are provided.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scTE
  role: Core tool that assigns reads first to gene exons and then to TEs, demultiplexes
    by cell barcode, collapses UMIs, and writes the cell-by-gene/TE count matrix from
    BAM/SAM input.
  repo: https://github.com/JiekaiLab/scTE
- name: scTE_build
  role: Builds the genome index (-x) used by scTE from gene GTF and TE BED annotations,
    or downloads annotations automatically for supported assemblies (mm10, hg38, panTro6,
    macFas5, dm6, danRer11, xenTro9).
  repo: https://github.com/JiekaiLab/scTE
- name: scTEATAC
  role: Processes scATAC-seq BAM files against a TE accessibility index to quantify
    chromatin accessibility of TEs alone.
  repo: https://github.com/JiekaiLab/scTE
- name: scTEATAC_build
  role: Builds the scATAC-seq TE index from filtered repeat BED input (e.g., mm10.te.bed
    filtered to LINE|SINE|LTR|Retroposon classes).
  repo: https://github.com/JiekaiLab/scTE
- name: Cell Ranger
  role: Upstream aligner whose output BAM (with CB/UB tags) serves as recommended
    unfiltered input to scTE.
- name: STARsolo
  role: Upstream aligner whose output BAM (with CR/UR tags) serves as recommended
    unfiltered input to scTE.
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:fe3ca2788cbec955ef4af4a3a7d607fe1bd845098750e4b768b6f1baeee0180a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiekaiLab/scTE
  tools:
  - Cell Ranger
  - STARsolo
  - scTE
  - scTEATAC
  - scTEATAC_build
  - scTE_build
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-read-to-feature-assignment

## Summary

An allocation strategy for assigning single-cell sequencing reads to genomic features, implemented in scTE, in which reads are first allocated to gene exons and then to transposable element (TE) metagenes, with multimapping TE reads collapsed and UMIs demultiplexed per cell per feature to produce a cell-by-gene/TE count matrix.

## When to use

Use when quantifying TE expression or accessibility from single-cell RNA-seq or ATAC-seq alignments (unfiltered BAM/SAM with CB/CR and UB/UR barcode/UMI tags) and a cell-by-feature count matrix is needed.

## When NOT to use

- Input is already a gene/TE expression or accessibility feature table rather than aligned BAM/SAM reads.
- The analysis goal is bulk RNA-seq counting without cell barcodes/UMIs — scTE's demultiplexing by CB/UB tags assumes 10x-style single-cell data.
- Custom pre-built indices for the genome assembly in use are unavailable and only BED/GTF inputs incompatible with scTE_build's six-column BED requirement are provided.

## Inputs

- Unfiltered BAM/SAM alignment file from Cell Ranger or STARsolo with cell barcodes and UMIs in CB/CR or UB/UR tags
- Gene annotation GTF file (e.g., GENCODE vM21/v30 or Ensembl release-102/103 GTF)
- Transposable element annotation BED file (six-column BED derived from UCSC rmsk.txt.gz)

## Outputs

- Cell-by-gene/TE count matrix in CSV format (default) or .h5ad format with --hdf5 True
- Genome index file (e.g., mm10.exclusive.idx) built by scTE_build from GTF and BED annotations

## How to apply

First build a genome index with scTE_build from a gene annotation GTF (e.g., GENCODE or Ensembl GTF) and a TE annotation BED (e.g., from UCSC rmsk.txt.gz), or use prebuilt indices for mm10, hg38, panTro6, macFas5, dm6, danRer9/xenTro9 via 'scTE_build -g <assembly>'. Provide the unfiltered BAM/SAM (Cell Ranger or STARsolo output, with barcodes/UMIs in CB/CR and UB/UR tags) via -i along with the index via -x, and specify barcode/UMI tags with -CB and -UMI. By default scTE allocates reads exclusively: reads mapping to gene exons contribute only to the gene, so TEs inside exon/UTR regions contribute to the gene and not the TE score; switch to '-m inclusive' to count such reads for both genes and TEs, or '-m nointron' to remove TEs inside gene introns. scTE demultiplexes reads by cell barcode and collapses UMIs per cell per feature, then writes a cell-by-gene/TE count matrix as CSV by default or .h5ad with '--hdf5 True'. For scATAC-seq, use scTEATAC_build/scTEATAC after filtering the rmsk annotations to LINE|SINE|LTR|Retroposon classes.

## Related tools

- **scTE** (Core tool that assigns reads first to gene exons and then to TEs, demultiplexes by cell barcode, collapses UMIs, and writes the cell-by-gene/TE count matrix from BAM/SAM input.) — https://github.com/JiekaiLab/scTE
- **scTE_build** (Builds the genome index (-x) used by scTE from gene GTF and TE BED annotations, or downloads annotations automatically for supported assemblies (mm10, hg38, panTro6, macFas5, dm6, danRer11, xenTro9).) — https://github.com/JiekaiLab/scTE
- **scTEATAC** (Processes scATAC-seq BAM files against a TE accessibility index to quantify chromatin accessibility of TEs alone.) — https://github.com/JiekaiLab/scTE
- **scTEATAC_build** (Builds the scATAC-seq TE index from filtered repeat BED input (e.g., mm10.te.bed filtered to LINE|SINE|LTR|Retroposon classes).) — https://github.com/JiekaiLab/scTE
- **Cell Ranger** (Upstream aligner whose output BAM (with CB/UB tags) serves as recommended unfiltered input to scTE.)
- **STARsolo** (Upstream aligner whose output BAM (with CR/UR tags) serves as recommended unfiltered input to scTE.) — https://github.com/alexdobin/STAR

## Examples

```
scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CB -UMI UB
```

## Evaluation signals

- The output count matrix file exists with cells as rows and gene/TE features as columns in the requested CSV or h5ad (--hdf5 True) format.
- With the default exclusive mode, reads overlapping TEs inside exon/UTR regions of GENCODE-annotated genes are counted toward the gene only, not the TE score.
- UMI counts are collapsed per cell per feature, and cells are demultiplexed by the CB/CR barcode tags present in the input BAM.
- The index passed via -x was built with scTE_build from a matching pair of gene GTF and six-column TE BED annotations for the correct genome assembly.

## Limitations

- Multimapping TE reads are collapsed, so individual TE copies cannot be resolved; TE information content is lower than that of genes and TEs serve mainly as an addendum to gene information.
- scATAC-seq data are very sparse — individual peaks in individual cells are challenging to resolve — and using TE data as enhancer information in single-cell ATAC-seq remains speculative.
- An unfiltered alignment file is highly recommended as input; filtered BAMs may lose barcodes from low-quality cells needed for demultiplexing.

## Evidence

- [methods] reads allocated to genes then TEs by default: "The scTE algorithm will allocate reads first to gene exons, and then to TEs by default."
- [readme] TEs in exonic regions contribute to gene only under default mode: "Hence TEs inside exon/UTR regions of genes annotated in GENCODE will only contribute to the gene, and not to the TE score."
- [methods] inclusive mode assigns reads to both TEs and genes: "setting `–mode/-m inclusive` in scTE, which will instruct scTE to assign the reads to both TEs and genes"
- [methods] unfiltered BAM/SAM as recommended input: "scTE makes BAM/SAM file as input, highly recommend to use unfiltered alignment file as input."
- [readme] scTE CLI invocation with barcode and UMI tags: "$ scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CR -UMI UR"
- [methods] STARsolo BAM requires barcodes in CR/UR tags: "For `bam` file generated by [STARsolo](https://github.com/alexdobin/STAR) etc, the cell barcodes and UMI need to be integrated into the read 'CR:Z' or 'UR:Z' tage"
