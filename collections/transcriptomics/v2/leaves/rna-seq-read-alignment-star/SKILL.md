---
name: rna-seq-read-alignment-star
description: Use when preprocessing labeled SPLiT-seq FASTQ reads (e.g., SCSit output
  with combinatorial barcodes and UMI) need to be aligned to mm10, hg19, or a combined
  mm10-hg19 reference to compare mapped-read yields.
when_to_use_negative:
- Input is not SPLiT-seq pair-end FASTQ data with combinatorial barcodes and UMI (e.g.,
  bulk RNA-seq or other single-cell platforms).
- The reference genome is neither mm10 nor hg19 nor a combined mm10-hg19 build.
- The goal is only to identify barcodes/UMIs (preprocessing) rather than to measure
  mapped-read yield.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STAR
  role: Aligns SCSit-labeled and original-method reads to the combined mm10-hg19 (or
    mm10) genome; uniquely mapped read counts are recorded for comparison.
- name: SCSit
  role: Preprocessing tool that outputs the labeled reads (with combinatorial barcodes
    and UMI) that are subsequently mapped with STAR.
  repo: https://github.com/shang-qian/SCSit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:c6ae028594e3b7e27acfee6435863b1fdbaf8cb57917006081916e0813e93e05
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
  - SCSit
  - STAR
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rna-seq-read-alignment-star

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align SPLiT-seq identified reads (from SCSit or the original method) to the combined mm10-hg19 genome (or mm10 alone) with STAR, and record uniquely mapped read counts to quantify how much more data SCSit retains.

## When to use

Use when preprocessing labeled SPLiT-seq FASTQ reads (e.g., SCSit output with combinatorial barcodes and UMI) need to be aligned to mm10, hg19, or a combined mm10-hg19 reference to compare mapped-read yields.

## When NOT to use

- Input is not SPLiT-seq pair-end FASTQ data with combinatorial barcodes and UMI (e.g., bulk RNA-seq or other single-cell platforms).
- The reference genome is neither mm10 nor hg19 nor a combined mm10-hg19 build.
- The goal is only to identify barcodes/UMIs (preprocessing) rather than to measure mapped-read yield.

## Inputs

- SCSit output FASTQ read pairs labeled with combinatorial barcodes and UMI
- Original-method FASTQ reads from SPLiT-seq filtering
- Combined mm10-hg19 reference genome (or mm10) for building the STAR index

## Outputs

- Uniquely mapped read counts per sample and per method
- Comparison table (mapped_reads_comparison.csv) with SCSit-to-original mapped-read ratios

## How to apply

Build a STAR genome index on the combined mm10-hg19 reference (or mm10 when the sample is mouse-only, e.g., SAMN08567263) before alignment. Align the SCSit output FASTQ reads for each of the five SPLiT-seq samples (SAMN08567263, SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567264) with STAR and record uniquely mapped read counts; repeat for reads identified by the original SPLiT-seq filtering (discard reads whose last 6 bases do not match the barcode, drop UMIs with more than 1 low-quality base at phred <= 10, reject reads whose three 8 nt cell barcodes contain more than one mismatch). Compute the SCSit-to-original mapped-read ratio per sample to quantify retention. The rationale is that SCSit's exact alignment of insertions and deletions labels more reads, which translates into roughly twice the uniquely mapped reads.

## Related tools

- **STAR** (Aligns SCSit-labeled and original-method reads to the combined mm10-hg19 (or mm10) genome; uniquely mapped read counts are recorded for comparison.)
- **SCSit** (Preprocessing tool that outputs the labeled reads (with combinatorial barcodes and UMI) that are subsequently mapped with STAR.) — https://github.com/shang-qian/SCSit

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output  # then align output FASTQs to combined mm10-hg19 index with STAR
```

## Evaluation signals

- Uniquely mapped reads from SCSit output are approximately twice those of the original method (e.g., BLAST and BWA based filtering).
- SAMN08567264 shows the largest incremental mapped reads (114,732,417) for SCSit.
- All five samples show more mapped reads with SCSit than with the original method (no sample should be lower).
- Identified reads from SCSit are consistent (97% consistency) and barcodes/UMIs unidentified by the original method are 90–97% of uniquely identified reads in SCSit (Table S3).

## Limitations

- Quantitative comparison results are deferred to supplementary Table S3, cited without being provided or linked in the text.
- Higher mismatch/indel tolerance in SCSit yields only 4–5% more identified reads at 20–40% more runtime (Table S4); only one mismatch and one indel are enabled by default.
- Results depend on the correct reference choice (combined mm10-hg19 for mixed-species samples, mm10 for mouse-only samples).

## Evidence
