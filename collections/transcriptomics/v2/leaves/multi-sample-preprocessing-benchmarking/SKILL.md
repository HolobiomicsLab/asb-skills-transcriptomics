---
name: multi-sample-preprocessing-benchmarking
description: Use when evaluating a new SPLiT-seq preprocessing tool against the original
  barcode/UMI filtering baseline across five or more samples, and you need per-sample
  identified-read overlap and 96-97% consistency rates as validation.
when_to_use_negative:
- Input data is not SPLiT-seq combinatorial barcoding reads; the fault-tolerant and
  indel alignment was proposed only for SPLiT-seq data in this study
- The goal is read mapping or genome alignment quantification rather than barcode/UMI
  identification benchmarking; use STAR-based mapping assessment instead
- Only a single sample is available and per-sample cross-method consistency across
  five samples cannot be computed
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCSit
  role: Multithreaded C preprocessing tool that identifies combinatorial barcodes
    and UMIs from SPLiT-seq reads; the method under benchmark
  repo: https://github.com/shang-qian/SCSit
- name: BLAST
  role: Alignment-based baseline alternative used in the original SPLiT-seq method
    for barcode determination
- name: BWA
  role: Faster alignment baseline alternative used in the original SPLiT-seq method
    for barcode determination
- name: Python 3.6
  role: Runtime dependency required for SCSit installation and operation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:2da8defa19aadec2516fa8d53ee1c2507879d22e1a7eb3f74d89f1125eb501b1
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multi-sample-preprocessing-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a preprocessing tool (e.g., SCSit) against a baseline method (original SPLiT-seq filtering) across multiple samples by intersecting identified reads and computing per-sample consistency percentages of identified reads.

## When to use

Use when evaluating a new SPLiT-seq preprocessing tool against the original barcode/UMI filtering baseline across five or more samples, and you need per-sample identified-read overlap and 96-97% consistency rates as validation.

## When NOT to use

- Input data is not SPLiT-seq combinatorial barcoding reads; the fault-tolerant and indel alignment was proposed only for SPLiT-seq data in this study
- The goal is read mapping or genome alignment quantification rather than barcode/UMI identification benchmarking; use STAR-based mapping assessment instead
- Only a single sample is available and per-sample cross-method consistency across five samples cannot be computed

## Inputs

- Five paired-end SPLiT-seq FASTQ sample pairs (R1/R2) from mouse and mixed human-mouse cells (SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264)
- barcodeList (TXT, 96 well plate oligonucleotides per barcode round)
- primerList (TXT, forward and reverse oligonucleotide sequences)
- Original SPLiT-seq filter parameters: last-6-base barcode match, UMI phred <= 10 threshold, one-mismatch tolerance on three 8 nt cell barcodes

## Outputs

- Per-sample table of SCSit-identified read counts, original-method identified read counts, intersection counts, and consistency percentages
- Summary consistency rate (expected 96-97%) between SCSit and original SPLiT-seq identified reads

## How to apply

Run SCSit (C program, multithreaded, e.g. -t 4) on each of the five paired-end FASTQ sample pairs (SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264) to obtain labeled reads with combinatorial barcodes and UMI. Independently implement the original SPLiT-seq baseline per sample: discard reads whose last 6 bases do not match the barcode sequence, filter UMIs with more than 1 low-quality base (phred <= 10), and reject reads whose three 8 nt cell barcodes contain more than one mismatch. Restrict both methods to the same filtered clean data, then intersect identified reads (read ID, barcode combination, UMI) per sample to compute the fraction of original-method reads also identified by SCSit. Record per-sample counts, overlaps, and consistency percentages into a summary table; consistency should reach the reported 96-97% range across the five samples.

## Related tools

- **SCSit** (Multithreaded C preprocessing tool that identifies combinatorial barcodes and UMIs from SPLiT-seq reads; the method under benchmark) — https://github.com/shang-qian/SCSit
- **BLAST** (Alignment-based baseline alternative used in the original SPLiT-seq method for barcode determination)
- **BWA** (Faster alignment baseline alternative used in the original SPLiT-seq method for barcode determination)
- **Python 3.6** (Runtime dependency required for SCSit installation and operation)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- Per-sample consistency of SCSit-identified reads with original SPLiT-seq identified reads reaches 96-97% across all five samples
- Mapped reads using SCSit output are approximately twice those of the original alignment method (BLAST/BWA)
- SCSit runtime is less than 10% of the original method runtime under 4 CPU cores
- Intersected reads share matching read ID, barcode combination, and UMI across both methods
- SCSit default tolerance is restricted to one mismatch and one indel; higher tolerance yields only 4-5% more reads at 20-40% more runtime

## Limitations

- Quantitative per-sample comparison values are deferred to supplementary Table S3 and not provided in the main text
- Runtime and tolerance trade-off evaluation is deferred to supplementary Table S4
- Default SCSit configuration is restricted to one mismatch and one indel; higher tolerance is not enabled by default and yields diminishing returns
- Application to other barcode-based single-cell platforms is stated as potential, not demonstrated; support deferred to future updates

## Evidence
