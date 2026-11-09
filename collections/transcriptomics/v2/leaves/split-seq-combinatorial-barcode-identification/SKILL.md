---
name: split-seq-combinatorial-barcode-identification
description: Use when preprocessing raw SPLiT-seq paired-end Illumina FASTQ reads
  to label each read's cellular origin by identifying combinatorial barcodes (three
  8 nt rounds) and UMI, especially when the original method's filters would discard
  too many reads.
when_to_use_negative:
- Input data is not from SPLiT-seq combinatorial barcoding — SCSit's fault-tolerant
  and indel alignment was proposed for SPLiT-seq data, and support for other single-cell
  sequencing platforms is deferred to future updates
- The task is genome alignment of clean reads (use STAR against mm10 or combined mm10-hg19
  after barcode identification, not SCSit)
- A permissive mismatch/indel tolerance beyond one mismatch and one indel is desired
  for maximal read recovery — this costs 20–40% more runtime for only 4–5% more identified
  reads (Table S4)
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCSit
  role: 'Primary preprocessing tool: identifies three rounds of barcode and UMI in
    SPLiT-seq paired-end reads and outputs labeled FASTQ files, with one mismatch
    and one indel tolerance'
  repo: https://github.com/shang-qian/SCSit
- name: BLAST
  role: Baseline alignment tool of the original SPLiT-seq method for barcode determination;
    serial on a single CPU core and time-consuming, used for comparison
- name: BWA
  role: Faster baseline aligner used for comparison; 8 bp barcode length is too short
    for BWA to take advantage for sequence alignment
- name: Python 3.6
  role: Required working installation for running SCSit (conda/pip distribution)
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:3d2180b90b59b1765ef689b5657ba612711b28f4b50c6c8ddd1f32711437255c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/shang-qian/SCSit
  repo_url: https://github.com/shang-qian/SCSit
  tools:
  - BLAST
  - BWA
  - Python
  - Python 3.6
  - SCSit
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spliT-seq-combinatorial-barcode-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify three rounds of combinatorial cell barcodes and UMI in SPLiT-seq paired-end reads using SCSit, which tolerates one mismatch and one indel during barcode alignment and outputs labeled Read1/Read2 FASTQ pairs.

## When to use

Use when preprocessing raw SPLiT-seq paired-end Illumina FASTQ reads to label each read's cellular origin by identifying combinatorial barcodes (three 8 nt rounds) and UMI, especially when the original method's filters would discard too many reads.

## When NOT to use

- Input data is not from SPLiT-seq combinatorial barcoding — SCSit's fault-tolerant and indel alignment was proposed for SPLiT-seq data, and support for other single-cell sequencing platforms is deferred to future updates
- The task is genome alignment of clean reads (use STAR against mm10 or combined mm10-hg19 after barcode identification, not SCSit)
- A permissive mismatch/indel tolerance beyond one mismatch and one indel is desired for maximal read recovery — this costs 20–40% more runtime for only 4–5% more identified reads (Table S4)

## Inputs

- Raw paired-end Illumina FASTQ files from SPLiT-seq (e.g. SRA/BioSample accessions SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264)
- primerList TXT file of oligonucleotide sequences used in each round
- barcodeList TXT file of the 96-well plate oligonucleotides for each round of barcodes

## Outputs

- Output Read1 FASTQ containing reads with identified combinatorial barcodes and UMI
- Corresponding output Read2 FASTQ with the sequencing data for identified reads
- Identified-read counts and identified-read ratios per sample (e.g. 148,423,169 reads / 66.98% for SAMN08567259; 160,257,239 / 66.26% for SAMN08567264; 163,502,984 / 75.84% for SAMN08567260; 163,780,622 / 74.89% for SAMN08567261; 56,833,343 / 73.22% for SAMN08567263)

## How to apply

Run SCSit (a C program, parallelized by multithread, e.g. -t 4) on each paired-end FASTQ pair with the primerList and barcodeList TXT files: 'scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output'. SCSit maps Read2 to 23 segments by an 8 nt sliding window to locate the index sequences of the three barcode rounds, converts indicator sequences via a quad-to-decimal conversion (0,1,2,3 for A,T,G,C), and permits one mismatch and one indel (the default setting) plus a distance-function threshold of 2; Read1 is trimmed by the primer list and retained only when its Read2 is valid. The output is a pair of FASTQ files where Read1 contains combinatorial barcodes and UMI. Rationale: tolerating one mismatch/one indel identifies more labeled reads than the original SPLiT-seq filters (discard last-6-base barcode mismatches, UMI with >1 base of phred ≤ 10, barcodes with >1 mismatch) while relaxing tolerance further only gains 4–5% reads at 20–40% more runtime.

## Related tools

- **SCSit** (Primary preprocessing tool: identifies three rounds of barcode and UMI in SPLiT-seq paired-end reads and outputs labeled FASTQ files, with one mismatch and one indel tolerance) — https://github.com/shang-qian/SCSit
- **BLAST** (Baseline alignment tool of the original SPLiT-seq method for barcode determination; serial on a single CPU core and time-consuming, used for comparison)
- **BWA** (Faster baseline aligner used for comparison; 8 bp barcode length is too short for BWA to take advantage for sequence alignment)
- **Python 3.6** (Required working installation for running SCSit (conda/pip distribution))

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- Identified-read ratio for each sample should be above 65% and closely match the reported SCSit values (66.98%, 66.26%, 75.84%, 74.89%, 73.22% for the five SPLiT-seq samples)
- Identified-read counts should match reported values: 148,423,169 (SAMN08567259), 163,502,984 (SAMN08567260), 163,780,622 (SAMN08567261), 56,833,343 (SAMN08567263), 160,257,239 (SAMN08567264)
- Consistency with the original SPLiT-seq identification method should reach the reported 96–97% range when read ID, barcode combination, and UMI sets are intersected
- Both an output Read1 FASTQ (with combinatorial barcodes and UMI) and a corresponding Read2 FASTQ must be produced for every sample
- Runtime under 4 CPU cores should be less than 10% of the original method's runtime

## Limitations

- Default SCSit configuration is restricted to one mismatch and one indel; higher tolerance is not enabled by default and yields only 4–5% more identified reads at 20–40% more runtime (Table S4)
- Fault-tolerant and indel alignment were proposed for SPLiT-seq data; application to other barcode-based single-cell data is potential, not demonstrated
- Support for additional single-cell sequencing platforms is deferred to future updates and not yet available

## Evidence
