---
name: count-matrix-driven-read-simulation
description: Use when benchmarking read-level bioinformatics tools (UMI deduplication,
  peak calling, alignment/coverage) needs synthetic scRNA-seq or scATAC-seq reads
  whose counts are grounded in a fitted synthetic count matrix.
when_to_use_negative:
- Benchmarking isoform abundance or alternative splicing tools — scReadSim provides
  gene-level UMI-count ground truths only, not isoform-level ground truths
- The count matrix itself is sufficient (count-level simulation alone) and no sequencing
  reads are needed — use scDesign2 directly instead of the read-level pipeline
- Multi-sample or multi-condition experiments where only a single BAM file is available
  — one BAM file per sample/condition must be supplied
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scReadSim
  role: 'End-to-end read simulator: genome segmentation, synthetic count generation,
    and synthetic read generation producing BAM/FASTQ reads with UMI-count ground
    truths'
  repo: https://github.com/JSB-UCLA/scReadSim
- name: scDesign2
  role: Internal count simulator trained by scReadSim to generate the synthetic gene-
    and inter-gene-by-cell UMI count matrices that drive read generation
  repo: https://github.com/JSB-UCLA/scDesign2
- name: MACS3
  role: Called via scReadSim's Utility.CallPeak to define ground-truth open chromatin
    peaks in the scATAC-seq mode
  repo: https://github.com/macs3-project/MACS
- name: minnow
  role: Alternative scRNA-seq read simulator used as baseline; it consumes scReadSim's
    synthetic UMI count matrix with default parameters but does not learn from real
    reads
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:79b7a86f022e2a11cad0caf5c9f76602eca931cb8d52853cd651403fd808e24f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - IGV
  - MACS3
  - bowtie2
  - minnow
  - scDesign2
  - scReadSim
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# count-matrix-driven-read-simulation

## Summary

Generates realistic synthetic scRNA-seq/scATAC-seq sequencing reads by first simulating a synthetic gene- or feature-by-cell count matrix (via scDesign2 trained on real UMI counts) and then using that matrix to guide read generation in BAM/FASTQ, as implemented in scReadSim.

## When to use

Use when benchmarking read-level bioinformatics tools (UMI deduplication, peak calling, alignment/coverage) needs synthetic scRNA-seq or scATAC-seq reads whose counts are grounded in a fitted synthetic count matrix.

## When NOT to use

- Benchmarking isoform abundance or alternative splicing tools — scReadSim provides gene-level UMI-count ground truths only, not isoform-level ground truths
- The count matrix itself is sufficient (count-level simulation alone) and no sequencing reads are needed — use scDesign2 directly instead of the read-level pipeline
- Multi-sample or multi-condition experiments where only a single BAM file is available — one BAM file per sample/condition must be supplied

## Inputs

- Real scRNA-seq BAM file containing reads in cells (e.g., mouse 10x Multiome RNA-seq, mm10 reference genome)
- Reference genome and chromosome size file (mm10, embedded in the package)
- Gene annotation GTF (e.g., gencode.vM10.annotation.gtf)
- Real UMI count matrices (gene-by-cell and inter-gene-by-cell) for training scDesign2

## Outputs

- Synthetic gene-by-cell UMI count matrix (intermediate output usable as ground truth)
- Synthetic paired-end reads as a genome-aligned BAM file (or FASTQ files)
- Ground-truth feature/peak annotations (for scATAC-seq, user-specified open chromatin regions)

## How to apply

Run scReadSim's three-step pipeline on a real BAM file (e.g., mouse 10x Multiome RNA-seq aligned to mm10 with gencode.vM10.annotation.gtf): (1) genome segmentation to build foreground/background feature sets (genes, inter-gene regions) via Utility.scRNA_CreateFeatureSets; (2) synthetic count generation, where scReadSim trains the count simulator scDesign2 on the real gene- and inter-gene-by-cell UMI count matrices (constructed with Utility.scRNA_bam2countmat or GenerateSyntheticCount.scRNA_GenerateSyntheticCount) to emit a synthetic UMI count matrix that also serves as ground truth; (3) synthetic read generation, which draws read 1 (concatenated UMI + cell barcode) and read 2 (RNA sequence extracted from the reference genome) according to the synthetic counts, producing genome-aligned paired-end reads in BAM (or FASTQ). Because counts are simulated before reads, the synthetic UMI count matrix doubles as ground truth for benchmarking tools such as UMI-tools; validate read-level realism by computing per-window read-coverage correlations against real reads.

## Related tools

- **scReadSim** (End-to-end read simulator: genome segmentation, synthetic count generation, and synthetic read generation producing BAM/FASTQ reads with UMI-count ground truths) — https://github.com/JSB-UCLA/scReadSim
- **scDesign2** (Internal count simulator trained by scReadSim to generate the synthetic gene- and inter-gene-by-cell UMI count matrices that drive read generation) — https://github.com/JSB-UCLA/scDesign2
- **MACS3** (Called via scReadSim's Utility.CallPeak to define ground-truth open chromatin peaks in the scATAC-seq mode) — https://github.com/macs3-project/MACS
- **minnow** (Alternative scRNA-seq read simulator used as baseline; it consumes scReadSim's synthetic UMI count matrix with default parameters but does not learn from real reads)

## Examples

```
pip install git+https://github.com/JSB-UCLA/scReadSim.git
```

## Evaluation signals

- Pseudo-bulk read-coverage Spearman correlation between synthetic and real reads (1000-bp bins on chr1) should reach ~0.930 for scReadSim (vs ~0.065 for minnow)
- Synthetic UMI count matrix from step 2 should be recovered as the ground truth when benchmarking UMI deduplication tools such as UMI-tools
- Output BAM must be genome-aligned to mm10 (FASTQ outputs from other simulators require alignment, e.g., with bowtie2, before comparison)
- Synthetic reads should carry valid read 1 structure: concatenated UMI and cell barcode sequences sampled like the real data
- Synthetic data should reproduce both count-level (UMI counts) and read-sequence-level (coverage tracks) behavior of the input dataset

## Limitations

- Default implementation may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect benchmarks such as cellranger default mode
- Gene-level ground truths only (UMI counts); no isoform abundance or alternative splicing ground truths, so not recommended for benchmarking isoform/splicing tools
- Requires one BAM file per sample or condition; no built-in multi-sample/condition generation
- Support for continuous trajectories, spatial locations, conditions/batches depends on scDesign3, which is not yet the internal count simulator

## Evidence

- [methods] three-step pipeline structure: "scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation."
- [methods] scDesign2 as internal count simulator: "In the second step—synthetic count generation, scReadSim trains the count simulator scDesign2"
- [intro] ground truth rationale: "the synthetic UMI count matrix—an intermediate output of scReadSim—serves as the ground truth for benchmarking scRNA-seq UMI deduplication tools"
- [intro] minnow limitation motivating count-matrix-driven reads: "minnow generates scRNA-seq reads only from annotated genes, so it cannot resemble real scRNA-seq data in intergenic regions"
- [abstract] read-level fidelity claim: "At both read-sequence and read-count levels, scReadSim mimics real scRNA-seq and scATAC-seq data."
- [methods] read 1 structure: "The read 1 is a concatenated sequence string of a UMI and a cell barcode."
- [readme] output formats and installation: [evidence span withheld — anchor status: external_doc]
