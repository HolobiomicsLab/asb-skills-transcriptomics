---
name: cross-method-mapped-read-count-comparison
description: Use when evaluating a SPLiT-seq preprocessing method against the original
  barcode-filtering method by comparing STAR uniquely-mapped read counts across the
  five SPLiT-seq samples (SAMN08567259-64, mm10/hg19).
when_to_use_negative:
- Input data are not SPLiT-seq combinatorial-barcode reads (e.g. data from other single-cell
  platforms); the article states support for additional platforms is deferred to future
  updates and not yet available.
- The goal is to measure identification consistency or runtime rather than downstream
  mapping yield; those require the 97% consistency benchmark or 4-core runtime assessment
  instead.
- The reads have already been filtered and aligned and only a count matrix is available;
  no STAR re-mapping comparison can be performed.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCSit
  role: Multithreaded C preprocessing tool that labels SPLiT-seq reads with combinatorial
    barcodes and UMI, tolerating one mismatch and one INDEL, producing the read set
    to be mapped
  repo: https://github.com/shang-qian/SCSit
- name: STAR
  role: Spliced aligner used to map both SCSit-output and original-method reads to
    the combined mm10-hg19 (or mm10) genome and report uniquely mapped read counts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:d0aad984fc1542697a969f6ec12e37c7359390a7b594c4d2591ba3f5a2556e01
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-method-mapped-read-count-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking procedure that preprocesses the same SPLiT-seq FASTQ sample pairs with SCSit and with the original SPLiT-seq filtering, aligns both labeled-read sets with STAR to the combined mm10-hg19 (or mm10) genome, and tabulates uniquely mapped read counts per sample to show SCSit yields roughly twice the mapped reads.

## When to use

Use when evaluating a SPLiT-seq preprocessing method against the original barcode-filtering method by comparing STAR uniquely-mapped read counts across the five SPLiT-seq samples (SAMN08567259-64, mm10/hg19).

## When NOT to use

- Input data are not SPLiT-seq combinatorial-barcode reads (e.g. data from other single-cell platforms); the article states support for additional platforms is deferred to future updates and not yet available.
- The goal is to measure identification consistency or runtime rather than downstream mapping yield; those require the 97% consistency benchmark or 4-core runtime assessment instead.
- The reads have already been filtered and aligned and only a count matrix is available; no STAR re-mapping comparison can be performed.

## Inputs

- Five SPLiT-seq paired-end FASTQ sample pairs (SAMN08567263, SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567264)
- SCSit barcodeList and primerList TXT files (96-well plate oligonucleotide sequences per barcode round; forward/reverse primer sequences)
- Combined mm10-hg19 reference genome (and mm10 alone) for building a STAR index
- Original SPLiT-seq filtering rules (last-6-base barcode match, UMI phred <= 10 quality filter, <=1 mismatch in each 8 nt cell barcode)

## Outputs

- Labeled-read FASTQ pairs from SCSit per sample (Read1 with combinatorial barcodes and UMI, Read2 with sequencing data)
- Filtered-read FASTQ pairs from the original SPLiT-seq method per sample
- STAR uniquely mapped read counts per sample per method
- Comparison table (e.g. mapped_reads_comparison.csv) with per-sample counts and SCSit-to-original ratios

## How to apply

Run SCSit (C program, multithreaded, e.g. -t 4) on each of the five paired-end SPLiT-seq FASTQ sample pairs (SAMN08567263, SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567264) to output labeled reads with combinatorial barcodes and UMI. In parallel, apply the original SPLiT-seq filtering to the same samples: discard reads whose last 6 bases do not match the barcode sequence, reject UMIs with more than 1 low-quality base (phred <= 10), and reject reads whose three 8 nt cell barcodes contain more than one mismatch. Build a STAR index on the combined mm10-hg19 (or mm10 alone for the mouse-only sample) reference, align both the SCSit output and the original-method reads with STAR, and record uniquely mapped read counts per sample. Compute the SCSit-to-original mapped-read ratio per sample and assemble counts into a comparison table (e.g. mapped_reads_comparison.csv). The rationale is that SCSit's exact alignment of one-mismatch and one-INDEL barcodes retains more reads, which the STAR mapping step converts into an end-to-end measure of recoverable data (mapped reads about twice the original).

## Related tools

- **SCSit** (Multithreaded C preprocessing tool that labels SPLiT-seq reads with combinatorial barcodes and UMI, tolerating one mismatch and one INDEL, producing the read set to be mapped) — https://github.com/shang-qian/SCSit
- **STAR** (Spliced aligner used to map both SCSit-output and original-method reads to the combined mm10-hg19 (or mm10) genome and report uniquely mapped read counts)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- SCSit uniquely mapped read counts are approximately twice the original-method counts across all five samples
- The largest mapped-read increment occurs for SAMN08567264 (e.g. about 114,732,417 additional uniquely mapped reads)
- Per-sample read counts match the reported magnitudes, e.g. SAMN08567261 (163,780,622), SAMN08567260 (163,502,984), SAMN08567259 (148,423,169), SAMN08567264 (160,257,239)
- The comparison table contains all five samples with both method columns and a valid SCSit-to-original ratio per row
- STAR was run against the correct reference: combined mm10-hg19 for mixed human/mouse samples, mm10 for the mouse-only sample SAMN08567263

## Limitations

- The quantitative per-sample comparison values are deferred to supplementary Table S3, which is cited but not provided or linked in the text.
- SCSit's default tolerance is one mismatch and one indel; higher tolerance was evaluated but not adopted, increasing identified reads only 4-5% at 20-40% more runtime (supplementary Table S4).
- Comparison counts depend on the STAR genome choice (combined mm10-hg19 vs mm10); mixing references across samples invalidates the ratio.

## Evidence
