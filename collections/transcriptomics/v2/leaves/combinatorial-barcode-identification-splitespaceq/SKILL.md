---
name: combinatorial-barcode-identification-splitespaceq
description: Use when preprocessing raw SPLiT-seq paired-end FASTQ files (150 nt Illumina
  kits) to assign cell-type barcodes and UMIs, especially when the original blastn-short
  preprocessing is too slow or discards reads with barcode mismatches or INDELs.
when_to_use_negative:
- The data are not SPLiT-seq combinatorial-indexing reads; fault-tolerant and INDEL
  alignment in this study was proposed only for SPLiT-seq data, and support for other
  single-cell platforms was deferred to future updates.
- 'You need tolerance beyond one mismatch and one INDEL by default: the authors evaluated
  this and found identified reads increase only 4–5% while runtime grows 20–40% (Table
  S4).'
- The task is full alignment of short 8 bp barcodes with BWA-style aligners; the paper
  notes barcode length (8 bp) is too short for BWA alignment to be effective.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SCSit
  role: 'Primary tool: multithreaded C program that identifies three rounds of combinatorial
    barcodes and UMI with one-mismatch/one-INDEL tolerance and outputs labeled FASTQ
    reads'
  repo: https://github.com/shang-qian/SCSit
- name: BLAST
  role: 'Baseline/benchmark: the original SPLiT-seq preprocessing uses blastn-short
    serially on a single core to determine all three barcodes; SCSit is compared against
    it for runtime'
- name: BWA
  role: Alternative baseline aligner mentioned in the article; unsuitable alone because
    8 bp barcodes are too short for BWA to align effectively
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:37e849efad16fee506fd7d584762c0a467e3e9f0be5ad6dc646cbeda9c1a70a4
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
  - SCSit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# combinatorial-barcode-identification-splitespaceq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify the three rounds of combinatorial cell barcodes and UMI in SPLiT-seq paired-end FASTQ reads using SCSit, a multithreaded C tool that replaces serial blastn-short/BWA alignment with sliding-window k-mer matching tolerant to one mismatch and one INDEL per barcode.

## When to use

Use when preprocessing raw SPLiT-seq paired-end FASTQ files (150 nt Illumina kits) to assign cell-type barcodes and UMIs, especially when the original blastn-short preprocessing is too slow or discards reads with barcode mismatches or INDELs.

## When NOT to use

- The data are not SPLiT-seq combinatorial-indexing reads; fault-tolerant and INDEL alignment in this study was proposed only for SPLiT-seq data, and support for other single-cell platforms was deferred to future updates.
- You need tolerance beyond one mismatch and one INDEL by default: the authors evaluated this and found identified reads increase only 4–5% while runtime grows 20–40% (Table S4).
- The task is full alignment of short 8 bp barcodes with BWA-style aligners; the paper notes barcode length (8 bp) is too short for BWA alignment to be effective.

## Inputs

- Paired-end FASTQ files from SPLiT-seq (any FASTQ quality encoding variant)
- primerList TXT file of all forward and reverse oligonucleotide primer sequences (paper Table S2)
- barcodeList TXT file of the 96 well plate oligonucleotides used for each round of barcodes

## Outputs

- Output Read1 FASTQ file annotated with combinatorial barcodes and UMI
- Corresponding labeled Read2 FASTQ file
- File-prefix-based output paths set via -o [default: ./output]

## How to apply

Run SCSit on each Read1/Read2 FASTQ pair with a primer list (Table S2 forward/reverse primers) and the 96-well plate barcode list, e.g. 'scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output', setting -t to the desired thread count. SCSit slides an 8 nt window across Read2, dividing index21 and index32 into 23 segments, converts the quad alphabet (A=0, T=1, G=2, C=3) to decimal, and accepts an index sequence on complete continuous match, one mismatch within positions 9–21 (with W1 and W23 fully matched), or one INDEL when the index length differs from 30 nt. A distance-function threshold of 2 decides barcode validity (Dist < 2 keeps Read2; otherwise it is abandoned). Valid Read2s retain their paired Read1, which is trimmed of primer sequences, and labeled reads with combinatorial barcodes and UMI are output as a FASTQ pair. Compared with the original SPLiT-seq filters (discard reads whose last 6 bases do not match the barcode, drop UMIs with more than one low-quality base at phred ≤ 10, reject 8 nt barcodes with more than one mismatch), SCSit raises identified-read consistency to 97% and roughly doubles mapped reads.

## Related tools

- **SCSit** (Primary tool: multithreaded C program that identifies three rounds of combinatorial barcodes and UMI with one-mismatch/one-INDEL tolerance and outputs labeled FASTQ reads) — https://github.com/shang-qian/SCSit
- **BLAST** (Baseline/benchmark: the original SPLiT-seq preprocessing uses blastn-short serially on a single core to determine all three barcodes; SCSit is compared against it for runtime)
- **BWA** (Alternative baseline aligner mentioned in the article; unsuitable alone because 8 bp barcodes are too short for BWA to align effectively)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- All three barcodes of each retained Read2 match within tolerance: complete continuous 8 nt window match, or exactly one mismatch in positions 9–21 with W1 and W23 fully matched, or exactly one INDEL (index length ≠ 30 nt).
- Distance-function check: barcode identification is valid only when Dist < threshold (2); reads failing this are abandoned.
- Identified-read consistency should approach the paper's reported 97%.
- Mapped reads (to combined mm10-hg19 or mm10 via STAR) should be roughly twice those of the original blastn-short-based method.
- Runtime on 4 CPU cores should be less than 10% of the original SPLiT-seq preprocessing runtime across benchmark samples (e.g. SAMN08567263: 0.62 h vs 17.68 h).

## Limitations

- Default tolerance is fixed at one mismatch and one indel; relaxing tolerance further increases identified reads by only 4–5% but takes 20–40% more runtime (Table S4).
- The INDEL-tolerant and fault-tolerant alignment was proposed for SPLiT-seq data only; application to other barcode-based single-cell data is potential, not demonstrated, and support for additional platforms was deferred to future updates.
- SCSit requires a working Python 3.6 installation and the precompiled C program is optimized for x86-64 CPUs; it only supports FASTQ input files.
- Quantitative comparisons are deferred to supplementary Tables S3 and S4, which are cited but not provided or linked in the main text.

## Evidence

- [intro] which automatically identifies three rounds of barcode and UMI and significant increase the: "we develop a high-efficiency preprocessing tool for single-cell sequencing data from SPLiT-seq (SCSit), which automatically identifies three rounds of barcode and UMI"
- [abstract] the consistency of identified reads from SCSit increases to 97%, and mapped reads are twice than the original: "the consistency of identified reads from SCSit increases to 97%, and mapped reads are twice than the original"
- [methods] Then the Read2 were mapped to 23 segments by sliding window (8 nt) and detected the position of index sequences of three barcodes.: "Then the Read2 were mapped to 23 segments by sliding window (8 nt) and detected the position of index sequences of three barcodes."
- [methods] If one mismatch exists in the position m of sequences (9 ≤ m ≤ 21), and W1 and W23 are all completely matched.: "If one mismatch exists in the position m of sequences (9 ≤ m ≤ 21), and W1 and W23 are all completely matched."
- [methods] If the value of Dist is less than threshold (2), the barcode identification of Read2 is valid, otherwise Read2 is abandoned.: "If the value of Dist is less than threshold (2), the barcode identification of Read2 is valid, otherwise Read2 is abandoned."
- [methods] The original method used in SPLiT-seq discarded reads that last 6 bases of them did not match barcode sequence: "The original method used in SPLiT-seq discarded reads that last 6 bases of them did not match barcode sequence"
- [readme] SCSit takes a pair of pair-end sequencing read files from SPLiT-seq in FASTQ format, barcodeList and primerList in TXT format: "SCSit takes a pair of pair-end sequencing read files from SPLiT-seq in FASTQ format, barcodeList and primerList in TXT format, and outputs a pair of files in FASTQ format."
- [results] the runtime of SCSit is less than 10% of the original.: "we assessed the runtime of SCSit for five datasets under 4 cores of CPU"
