---
name: pseudo-bulk-genomic-window-coverage-quantification
description: Use when validating that simulated scRNA-seq/scATAC-seq reads (BAM/FASTQ
  from scReadSim) resemble real reads, by comparing pseudo-bulk read coverage across
  fixed-width genomic windows against the real input data.
when_to_use_negative:
- Benchmarking isoform abundance or alternative splicing tools — scReadSim provides
  gene-level UMI count ground truths only, not isoform-level ground truths
- Count-level benchmarking of scRNA-seq methods — this skill assesses read-sequence/coverage
  fidelity, not the synthetic UMI count matrix comparison
- Multi-sample or multi-condition simulations from a single pooled BAM — scReadSim
  requires one BAM file per sample or condition
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scReadSim
  role: Generates the synthetic sequencing reads (FASTQ/BAM) whose pseudo-bulk window
    coverage is quantified and compared to real data
  repo: https://github.com/JSB-UCLA/scReadSim
- name: bowtie2
  role: Aligns synthetic paired-end reads to the mm10 reference genome prior to coverage
    computation
- name: samtools
  role: Processes aligned BAMs (e.g., depth/sort) to extract per-window read coverage
- name: bedtools
  role: Supports genomic interval operations (e.g., getfasta extraction of 90-nt reads)
    used in building the read inputs for coverage analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:f8efff6d351775377af77cfcf6ddcc8ca16c752df59687e357489750c10a66a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - bedtools
  - bowtie2
  - samtools
  - scReadSim
  - seqtk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pseudo-bulk-genomic-window-coverage-quantification

## Summary

Quantifies pseudo-bulk read coverage by binning a reference genome (e.g., chromosome 1 of the mm10 mouse genome) into fixed 1000-bp windows and computing per-window read coverage for real and scReadSim-generated synthetic BAM files, enabling read-level fidelity checks of synthetic data.

## When to use

Use when validating that simulated scRNA-seq/scATAC-seq reads (BAM/FASTQ from scReadSim) resemble real reads, by comparing pseudo-bulk read coverage across fixed-width genomic windows against the real input data.

## When NOT to use

- Benchmarking isoform abundance or alternative splicing tools — scReadSim provides gene-level UMI count ground truths only, not isoform-level ground truths
- Count-level benchmarking of scRNA-seq methods — this skill assesses read-sequence/coverage fidelity, not the synthetic UMI count matrix comparison
- Multi-sample or multi-condition simulations from a single pooled BAM — scReadSim requires one BAM file per sample or condition

## Inputs

- Real scRNA-seq BAM file (e.g., mouse 10x Multiome RNA modality, mm10 reference)
- Synthetic BAM files (or BED-derived paired-end FASTQ aligned with bowtie2) for each cell-number/depth condition
- mm10 chromosome size file (embedded in scReadSim)
- 1000-bp genomic window coordinates over chromosome 1

## Outputs

- Per-window read coverage values for the real data and each synthetic condition
- pseudo_bulk_coverage_summary.csv summarizing coverage vs. cell number at fixed depth and vs. sequencing depth at fixed cell number

## How to apply

First align synthetic paired-end reads (90-nt read length, 5-bp jitter size) to the mm10 reference genome with bowtie2, using samtools to sort/index the output BAM. Bin chromosome 1 of the mm10 reference into 1000-bp windows, then compute read coverage per window for the real BAM and each synthetic condition. Pool reads across all synthetic cells to form the pseudo-bulk profile, then aggregate per-window coverage into a summary table (pseudo_bulk_coverage_summary.csv) that records how coverage varies with cell number at fixed sequencing depth (1219, 2438, 4877, 9754, 19508 cells at 5.0M reads) and with sequencing depth at fixed cell number (1.2M–20.1M reads at 4877 cells). The expected invariant is that pseudo-bulk coverage decreases monotonically as synthetic cell number increases at fixed depth, and increases as sequencing depth increases at fixed cell number.

## Related tools

- **scReadSim** (Generates the synthetic sequencing reads (FASTQ/BAM) whose pseudo-bulk window coverage is quantified and compared to real data) — https://github.com/JSB-UCLA/scReadSim
- **bowtie2** (Aligns synthetic paired-end reads to the mm10 reference genome prior to coverage computation)
- **samtools** (Processes aligned BAMs (e.g., depth/sort) to extract per-window read coverage)
- **bedtools** (Supports genomic interval operations (e.g., getfasta extraction of 90-nt reads) used in building the read inputs for coverage analysis)

## Examples

```
samtools depth -a -b chr1_windows_1000bp.bed aligned_synthetic.bam > pseudo_bulk_coverage.tsv
```

## Evaluation signals

- All ten synthetic conditions (5 cell-number x fixed depth; 5 depths x fixed cell number) yield paired-end FASTQ and synthetic count matrix outputs
- Pseudo-bulk coverage of pooled 1219 synthetic cells decreases monotonically as synthetic cell number increases (1219 → 19508) at fixed 5.0M sequencing depth
- Coverage increases monotonically as sequencing depth increases (1.2M → 20.1M reads) at fixed 4877-cell number
- Per-window coverage computed on identical 1000-bp windows of chromosome 1 for both real and synthetic BAMs (apples-to-apples comparability)
- Summary table pseudo_bulk_coverage_summary.csv contains one coverage row per real/synthetic condition with the expected cell-number/depth trend

## Limitations

- Default scReadSim implementation may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect benchmarks such as cellranger default mode
- scReadSim requires users to input one BAM file per sample or condition, so multi-sample coverage comparisons need separate runs
- Coverage assessment is restricted to gene-level ground truths; isoform-level fidelity cannot be evaluated

## Evidence

- [methods] As an overview, scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation.: "As an overview, scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation."
- [methods] This BAM file uses mm10 as reference genome, the required chromosome size file is also embedded within the package.: "This BAM file uses mm10 as reference genome, the required chromosome size file is also embedded within the package."
- [readme] Trained on real data, scReadSim generates synthetic sequencing reads in BAM or FASTQ formats.: [evidence span withheld — anchor status: external_doc]
- [discussion] the default implementation of scReadSim may generate intronic synthetic reads when extracting sequences from the reference genome: "the default implementation of scReadSim may generate intronic synthetic reads when extracting sequences from the reference genome"
