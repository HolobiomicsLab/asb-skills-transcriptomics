---
name: splitseq-read-barcoding-preprocessing
description: Use when preprocessing SPLiT-seq paired-end FASTQ data (e.g., SAMN08567259–SAMN08567264)
  to identify combinatorial barcodes and UMI fault-tolerantly, and to compare STAR
  mapped-read counts against the original BLAST/BWA-based SPLiT-seq filtering.
when_to_use_negative:
- Input data is from other single-cell sequencing platforms (e.g., non-SPLiT-seq barcode
  chemistries) — the article states fault-tolerant/indel alignment was proposed for
  SPLiT-seq data and support for more platforms is deferred to future updates
- Barcodes are longer than 8 nt — the article notes BWA fails to exploit such short
  8 bp barcodes and BLAST-based determination is slow; SCSit's k-mer/indicator design
  targets the 8 nt SPLiT-seq barcodes
- You need tolerance beyond one mismatch and one indel by default — the article reports
  this increases identified reads only 4–5% while taking 20–40% more runtime, so it
  was not adopted
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: SCSit
  role: C program, multithreaded, that identifies the three 8 nt cell barcodes (tolerating
    one mismatch and one INDEL) and UMI, trims primers from Read1, and outputs labeled
    FASTQ reads
  repo: https://github.com/shang-qian/SCSit
- name: STAR
  role: Spliced aligner used to map SCSit-identified and original-method reads to
    the combined mm10-hg19 or mm10 genome to count uniquely mapped reads
- name: BLAST
  role: Serial per-core barcode-determination aligner used by the original SPLiT-seq
    method; serves as the runtime/yield baseline (e.g., for SAMN08567264 with 160,257,239
    reads)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:2ae3e49bbc237f79f1206c6ccfe66ce19c23218cea4e34a597be23028a7127c6
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
  - SCSit
  - STAR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# splitseq-read-barcoding-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A preprocessing skill for SPLiT-seq single-cell data in which SCSit, a multithreaded C program, fault-tolerantly identifies three 8 nt cell barcodes and UMI (one mismatch, one INDEL) from paired-end FASTQ reads, and the resulting labeled reads are aligned with STAR to a combined mm10-hg19 or mm10 genome to demonstrate roughly doubled mapped-read yield versus the original SPLiT-seq method.

## When to use

Use when preprocessing SPLiT-seq paired-end FASTQ data (e.g., SAMN08567259–SAMN08567264) to identify combinatorial barcodes and UMI fault-tolerantly, and to compare STAR mapped-read counts against the original BLAST/BWA-based SPLiT-seq filtering.

## When NOT to use

- Input data is from other single-cell sequencing platforms (e.g., non-SPLiT-seq barcode chemistries) — the article states fault-tolerant/indel alignment was proposed for SPLiT-seq data and support for more platforms is deferred to future updates
- Barcodes are longer than 8 nt — the article notes BWA fails to exploit such short 8 bp barcodes and BLAST-based determination is slow; SCSit's k-mer/indicator design targets the 8 nt SPLiT-seq barcodes
- You need tolerance beyond one mismatch and one indel by default — the article reports this increases identified reads only 4–5% while taking 20–40% more runtime, so it was not adopted

## Inputs

- SPLiT-seq paired-end FASTQ files (R1 and R2), e.g. samples SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264 (Illumina, 150 nt kits)
- barcodeList TXT file (96-well plate oligonucleotides for each round of barcodes)
- primerList TXT file (forward and reverse oligonucleotide sequences)
- Combined mm10-hg19 (or mm10) reference genome for STAR index construction

## Outputs

- Labeled reads in FASTQ format: output Read1 composed of combinatorial barcodes and UMI, Read2 as corresponding sequencing data
- Uniquely mapped read counts per sample for SCSit-identified and original-method reads (STAR alignment to combined mm10-hg19/mm10)
- Comparison table of SCSit-to-original mapped-read ratios (e.g., mapped_reads_comparison.csv)

## How to apply

Run SCSit on each SPLiT-seq R1/R2 FASTQ pair with barcodeList (-b) and primerList (-p) in TXT format and multiple threads (e.g., -t 4); SCSit maps Read2 against 23 8 nt k-mer segments of the index21/index32 sequences, performs quad-to-decimal indicator-sequence conversion (A,T,G,C = 0,1,2,3), and accepts barcodes with at most one mismatch and one INDEL (default tolerance), while Read1 is retained and trimmed by matched primer segments. In parallel, apply the original SPLiT-seq filters to the same samples: discard reads whose last 6 bases do not match the barcode, reject UMIs with more than 1 low-quality base (phred <= 10), and reject reads whose three 8 nt cell barcodes contain more than one mismatch. Build a STAR index on the combined mm10-hg19 (or mm10) reference and align SCSit output and original-method reads separately, recording uniquely mapped counts per sample. The rationale is that SCSit's exact insertion/deletion alignment recovers reads the original stringent method discards, so comparison of uniquely mapped counts quantifies the gain.

## Related tools

- **SCSit** (C program, multithreaded, that identifies the three 8 nt cell barcodes (tolerating one mismatch and one INDEL) and UMI, trims primers from Read1, and outputs labeled FASTQ reads) — https://github.com/shang-qian/SCSit
- **STAR** (Spliced aligner used to map SCSit-identified and original-method reads to the combined mm10-hg19 or mm10 genome to count uniquely mapped reads)
- **BLAST** (Serial per-core barcode-determination aligner used by the original SPLiT-seq method; serves as the runtime/yield baseline (e.g., for SAMN08567264 with 160,257,239 reads))

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- Consistency of identified reads from SCSit reaches ~97%
- SCSit mapped reads are approximately twice those of the original alignment method across all five samples after STAR alignment to the combined mm10-hg19 genome
- Largest incremental mapped reads observed for SAMN08567264 (114,732,417 more uniquely mapped reads than the original method)
- SCSit runtime is less than 10% of the original BLAST/BWA-based method (assessed on 5 datasets under 4 CPU cores)
- Output Read1 FASTQ entries contain combinatorial barcodes plus UMI, and paired Read2 contains the corresponding sequencing data

## Limitations

- Default tolerance is restricted to one mismatch and one indel; higher tolerance increases identified reads only 4–5% at 20–40% more runtime (Table S4), so it is not enabled by default
- Fault-tolerant and indel alignment was proposed for SPLiT-seq data; applicability to other barcode-based single-cell platforms is stated as potential, not demonstrated
- Quantitative comparisons are deferred to supplementary Tables S3 and S4, which are cited but not provided or linked in the text
- SCSit requires a working installation of python 3.6; README installation instructions (conda/pip) conflict with the C-program build (./make.sh from the cloned repository)

## Evidence
