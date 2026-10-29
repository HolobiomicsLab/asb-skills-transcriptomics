---
name: single-cell-te-expression-quantification
description: Use when you have 10x-style scRNA-seq BAM files (Cell Ranger CB:Z/UB:Z
  or STARsolo CR:Z/UR:Z tags) and need per-cell TE expression counts to study TE heterogeneity
  during development, reprogramming, or differentiation.
when_to_use_negative:
- Input is already a gene or TE feature count matrix — scTE requires raw BAM/SAM alignments
  as input
- The analysis goal is chromatin accessibility of TEs rather than expression — scTEATAC/scTEATAC_build
  on scATAC-seq BAMs is the appropriate variant
- Filtered/selected alignment files are the only available input — the README highly
  recommends unfiltered alignment files
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scTE
  role: 'Core pipeline: takes BAM/SAM input plus barcode/UMI tags and quantifies TE
    expression per single cell into an .h5ad matrix'
  repo: https://github.com/JiekaiLab/scTE
- name: scTE_build
  role: Builds the gene/TE genome index (e.g. mm10.exclusive.idx) from GTF and RepeatMasker
    rmsk annotations
  repo: https://github.com/JiekaiLab/scTE
- name: Cell Ranger
  role: Source of 10x-style BAM inputs whose barcodes/UMIs are carried in CB:Z/UB:Z
    tags read by scTE
- name: STARsolo
  role: Alternative aligner producing BAM inputs whose barcodes/UMIs are carried in
    CR:Z/UR:Z tags read by scTE
  repo: https://github.com/alexdobin/STAR
- name: scTEATAC
  role: 'ATAC-seq variant: processes scATAC-seq BAMs against a TE accessibility index
    built with scTEATAC_build'
  repo: https://github.com/JiekaiLab/scTE
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:df1eaf7717be16c7296ef84d74548f69f72966d0a4d5d56da9b17a6ccc7a02af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiekaiLab/scTE
  tools:
  - Cell Ranger
  - STARsolo
  - scTE
  - scTEATAC
  - scTE_build
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-te-expression-quantification

## Summary

Quantify transposable element (TE) expression in single-cell RNA-seq data using the scTE pipeline, which counts TE-derived reads per cell from aligned BAM/SAM files and yields a cell-by-TE matrix (e.g. .h5ad) for detecting lineage-specific TE dynamics.

## When to use

Use when you have 10x-style scRNA-seq BAM files (Cell Ranger CB:Z/UB:Z or STARsolo CR:Z/UR:Z tags) and need per-cell TE expression counts to study TE heterogeneity during development, reprogramming, or differentiation.

## When NOT to use

- Input is already a gene or TE feature count matrix — scTE requires raw BAM/SAM alignments as input
- The analysis goal is chromatin accessibility of TEs rather than expression — scTEATAC/scTEATAC_build on scATAC-seq BAMs is the appropriate variant
- Filtered/selected alignment files are the only available input — the README highly recommends unfiltered alignment files

## Inputs

- Unfiltered BAM/SAM alignment file from Cell Ranger or STARsolo with cell barcode and UMI tags (CB:Z/UB:Z or CR:Z/UR:Z)
- Gene annotation GTF (e.g. GENCODE release 30 for hg38, release M21 for mm10, or Ensembl release-103 GTFs)
- UCSC RepeatMasker repeat annotation rmsk.txt.gz for the matching genome assembly (e.g. hg38, mm10, panTro6, macFas5, dm6, danRer11, xenTro9)

## Outputs

- Cell-by-TE count matrix in .h5ad format (e.g. out.h5ad) when --hdf5 True is set
- scTE genome index (e.g. mm10.exclusive.idx) built from GTF + rmsk annotations
- Per-cell TE expression profiles used for lineage/branch differential TE analysis and summary tables

## How to apply

Build a genome index with scTE_build from a gene annotation GTF (e.g. GENCODE hg38/mm10) and a UCSC RepeatMasker rmsk.txt.gz TE annotation, either via the automatic '-g hg38'/'-g mm10' shortcuts or custom '-te TEs.bed -gene Genes.gtf' options. Run scTE on the unfiltered BAM/SAM alignment file with the index, passing the cell-barcode and UMI tags matching the aligner (-CB CB -UMI UB for Cell Ranger, -CB CR -UMI UR for STARsolo) and --hdf5 True to emit an .h5ad cell-by-TE matrix; budget ~10 GB memory per thread. By default (exclusive mode) reads are allocated first to gene exons and then to TEs, so TEs inside exon/UTR regions of GENCODE genes contribute to the gene count only; switch to '-m inclusive' to count them for both, or '-m nointron' to drop intronic TEs. Downstream, aggregate TE-family counts by cell-type annotations (e.g. NKX2-5+ vs SPARC+ branches, IAP/RLTR45 in extraembryonic ectoderm) and validate against known phenomena such as LTR7/HERVH correlation (~0.8 Pearson, co-downregulation with POU5F1) or MERVL in mouse ESCs.

## Related tools

- **scTE** (Core pipeline: takes BAM/SAM input plus barcode/UMI tags and quantifies TE expression per single cell into an .h5ad matrix) — https://github.com/JiekaiLab/scTE
- **scTE_build** (Builds the gene/TE genome index (e.g. mm10.exclusive.idx) from GTF and RepeatMasker rmsk annotations) — https://github.com/JiekaiLab/scTE
- **Cell Ranger** (Source of 10x-style BAM inputs whose barcodes/UMIs are carried in CB:Z/UB:Z tags read by scTE)
- **STARsolo** (Alternative aligner producing BAM inputs whose barcodes/UMIs are carried in CR:Z/UR:Z tags read by scTE) — https://github.com/alexdobin/STAR
- **scTEATAC** (ATAC-seq variant: processes scATAC-seq BAMs against a TE accessibility index built with scTEATAC_build) — https://github.com/JiekaiLab/scTE

## Examples

```
scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CB -UMI UB
```

## Evaluation signals

- Output .h5ad file exists and loads as a valid cell-by-TE count matrix
- LTR7 and HERVH-int show a Pearson correlation of approximately 0.8 across cells and co-downregulate with POU5F1 during human differentiation
- Previously reported phenomena are reproduced, e.g. MERVL expression in mouse ESCs and LTR7/HERVH in human ESCs
- Expected lineage-specific TE signals are detected, e.g. IAP/RLTR45 in extraembryonic ectoderm, MER46C/RLTR20B3/LTRIS2 in Apoa2+ extraembryonic endoderm, and ETnERV3-int transient in early cardiac fate with L1ME3D in Tnnt2+ cardiomyocytes
- Mode choice is consistent with intent: exonic/UTR TE reads counted as gene only in exclusive mode, as both gene and TE in inclusive mode

## Limitations

- TEs inside exon/UTR regions of GENCODE genes contribute to the gene score and not the TE score by default (exclusive mode), potentially undercounting such TEs unless -m inclusive is set
- scTE requires roughly 10 GB of memory per thread, constraining parallelism
- TE information content is lower than that of genes; TE data serve mainly as an addendum to gene information
- TE-based enhancer inference from sparse scATAC-seq remains speculative and was not demonstrated
- BAM files from STARsolo must have barcodes/UMIs integrated into CR:Z/UR:Z tags, and Cell Ranger BAMs into CB:Z/UB:Z tags, or scTE cannot assign counts

## Evidence

- [methods] The scTE algorithm will allocate reads first to gene exons, and then to TEs by default.: "The scTE algorithm will allocate 
reads first to gene exons, and then to TEs by default."
- [methods] setting –mode/-m inclusive in scTE assigns reads to both TEs and genes: "setting `–mode/-m inclusive` in scTE, which will instruct scTE to assign the reads to both TEs and genes"
- [methods] scTE makes BAM/SAM file as input, highly recommend to use unfiltered alignment file as input.: "scTE makes BAM/SAM file as input, highly recommend to use unfiltered alignment file as input."
- [discussion] we could observe previously identified phenomena such as MERVL and LTR7/HERVH expression in mouse and human ESCs, respectively.: "By taking advantage of this tool, we could observe previously identified phenomena such as MERVL and LTR7/HERVH expression in mouse and human ESCs, respectively."
- [readme] the cell barcodes and UMI need to be integrated into the read 'CB:Z' or 'UB:Z' tage: "For `bam` file generated by [Cell Ranger](https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/what-is-cell-ranger) etc, the cell barcodes and UMI need to be"
- [abstract] scTE can discriminate cell type using chromatin accessibly of TEs alone.: "Finally, we apply scTE to single-cell ATAC-seq data, and demonstrate that scTE can discriminate cell type using chromatin accessibly of TEs alone."
