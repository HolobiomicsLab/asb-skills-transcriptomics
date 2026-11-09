---
name: umi-cell-barcode-synthesis
description: Use when benchmarking scRNA-seq UMI deduplication or read-level tools
  on 10X-style data and you need synthetic reads whose true UMI counts and cell barcodes
  are known (e.g. from a 10X BAM with CB/UMI structure).
when_to_use_negative:
- Benchmarking tools that need isoform-level ground truths (isoform abundance or alternative
  splicing events) — scReadSim provides gene-level UMI counts only
- Multi-sample or multi-condition studies requiring automatic simulation across samples
  — one BAM file per sample/condition must be supplied manually
- Data whose cells follow continuous trajectories or spatial locations, unless substituting
  scDesign3 as the count simulator
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scReadSim
  role: Orchestrates UMI modeling, synthetic count generation, and read 1 (cell barcode
    + UMI) sequence synthesis
  repo: https://github.com/JSB-UCLA/scReadSim
- name: scDesign2
  role: Count simulator trained on real UMI count matrices to generate the synthetic
    UMI-by-cell matrix when cells belong to distinct clusters
  repo: https://github.com/JSB-UCLA/scDesign2
- name: scDesign3
  role: Alternative count simulator for cells following continuous trajectories
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:afea6eef6fe069811e7dddb2059bd186926c50e48f714b4bf10609c0e49b60d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - Picard
  - bedtools
  - bowtie2
  - fgbio
  - samtools
  - scDesign2
  - scDesign3
  - scReadSim
  - seqtk
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# umi-cell-barcode-synthesis

## Summary

Generates synthetic read 1 sequences (a concatenated UMI and 10X-style cell barcode) plus a synthetic UMI-by-cell count matrix, using scDesign2 trained on real gene/inter-gene UMI count matrices so downstream UMI deduplication tools can be benchmarked against known ground truth.

## When to use

Use when benchmarking scRNA-seq UMI deduplication or read-level tools on 10X-style data and you need synthetic reads whose true UMI counts and cell barcodes are known (e.g. from a 10X BAM with CB/UMI structure).

## When NOT to use

- Benchmarking tools that need isoform-level ground truths (isoform abundance or alternative splicing events) — scReadSim provides gene-level UMI counts only
- Multi-sample or multi-condition studies requiring automatic simulation across samples — one BAM file per sample/condition must be supplied manually
- Data whose cells follow continuous trajectories or spatial locations, unless substituting scDesign3 as the count simulator

## Inputs

- BAM file containing 10X scRNA-seq reads in cells (aligned to mm10)
- Gene/inter-gene UMI count matrices from Utility.scRNA_bam2countmat (UMI_modeling=True)
- Optional cell type labels or a cell number / sequencing depth specification

## Outputs

- *.scDesign2Simulated.txt synthetic UMI count matrix (ground truth)
- synthetic_cell_barcode.txt file
- Synthetic read 1 sequences (UMI + cell barcode concatenation) within the paired-end FASTQ/BAM output

## How to apply

First construct UMI count matrices from the input BAM (e.g. 10X_RNA_chr1_3073253_4526737.bam aligned to mm10) with Utility.scRNA_bam2countmat setting UMI_modeling=True to obtain the UMI-by-cell matrix. Train scDesign2 via GenerateSyntheticCount.scRNA_GenerateSyntheticCount on the gene and inter-gene count matrices to emit a synthetic count matrix (default: same cell number and sequencing depth as the real data; Louvain clustering is applied if no cell labels are given; scDesign3 can be substituted when cells follow continuous trajectories). Then, during synthetic read generation with scRNA_GenerateBAM.scRNA_GenerateBAMCoord (read_len=90, jitter_size=5, CB_len=16, UB_len=10), build read 1 as a concatenated sequence string of a UMI and a cell barcode, sampling real reads from the BAM and writing synthetic_cell_barcode.txt. Validate by confirming the *.scDesign2Simulated.txt matrix and synthetic_cell_barcode.txt exist; the synthetic UMI count matrix serves as ground truth for benchmarking deduplication tools like UMI-tools, Alevin, and STARsolo.

## Related tools

- **scReadSim** (Orchestrates UMI modeling, synthetic count generation, and read 1 (cell barcode + UMI) sequence synthesis) — https://github.com/JSB-UCLA/scReadSim
- **scDesign2** (Count simulator trained on real UMI count matrices to generate the synthetic UMI-by-cell matrix when cells belong to distinct clusters) — https://github.com/JSB-UCLA/scDesign2
- **scDesign3** (Alternative count simulator for cells following continuous trajectories)

## Examples

```
python -m scReadSim Utility.scRNA_bam2countmat --UMI_modeling=True; GenerateSyntheticCount.scRNA_GenerateSyntheticCount --input_dir <dir> --output_dir <dir>; scRNA_GenerateBAM.scRNA_GenerateBAMCoord --read_len=90 --jitter_size=5 --CB_len=16 --UB_len=10
```

## Evaluation signals

- *.scDesign2Simulated.txt synthetic count matrix and synthetic_cell_barcode.txt exist in the output directory
- *.read1.bed2fa.fq and *.read2.bed2fa.fq paired-end FASTQ files are produced
- Synthetic read 1 decodes to a CB_len=16 cell barcode followed by a UB_len=10 UMI
- Synthetic UMI matrix matches the requested cell number and sequencing depth (defaults mirror the real data)
- Deduplicated counts from tools like UMI-tools can be compared cell-by-cell against the synthetic UMI matrix as ground truth

## Limitations

- Default implementation may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect benchmarks like cellranger default mode
- Ground truths are gene-level UMI counts only; isoform abundance and alternative splicing events are not simulated
- One BAM file per sample or condition is required; multi-sample generation is not automated

## Evidence

- [methods] Read 1 concatenation: "The read 1 is a concatenated sequence string of a UMI and a cell barcode."
- [intro] Ground truth for benchmarking: "the synthetic UMI count matrix—an intermediate output of scReadSim—serves as the ground truth for benchmarking scRNA-seq UMI deduplication tools"
- [methods] UMI modeling step: "To generate the gene- and inter-gene-by-cell UMI count matrices for synthetic cells, scReadSim trains the count simulator scDesign2"
- [intro] Simulator choice rationale: "scReadSim trains the count simulator scDesign26 (if the cells belong to distinct clusters; otherwise, scDesign322 can be used"
- [abstract] Tool benchmarking context: "UMI-tools achieves the top accuracy in scRNA-seq UMI deduplication"
