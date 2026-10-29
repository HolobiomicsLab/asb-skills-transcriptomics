---
name: combined-mouse-human-reference-genome-mapping
description: Use when preprocessing SPLiT-seq FASTQ samples (e.g. SAMN08567259–SAMN08567264)
  with SCSit vs. the original filter and you must quantify alignment yield differences
  on a combined mm10-hg19 STAR reference.
when_to_use_negative:
- The input data is not SPLiT-seq combinatorial-barcoding data — the article states
  the fault-tolerant and indel alignment was proposed for SPLiT-seq data and application
  to other single-cell data is only potential, not demonstrated
- The goal is species-specific alignment without mixed-cell contamination (e.g. pure
  human samples), where a combined mm10-hg19 index is unnecessary
- You need cell-level clustering or expression matrices rather than raw mapped-read
  yield comparison — this skill covers alignment counting only
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: SCSit
  role: Preprocesses SPLiT-seq FASTQ pairs to output labeled reads with combinatorial
    barcodes and UMI before STAR alignment
  repo: https://github.com/shang-qian/SCSit
- name: STAR
  role: Aligns SCSit-labeled and original-method reads to the combined mm10-hg19 (or
    mm10) genome and reports uniquely mapped read counts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:81f90ab18809a534b0882df69a1ad5120763a7910605a667363881b1f3a27ad3
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Combined Mouse-Human Reference Genome Mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps SPLiT-seq identified reads to a combined mm10-hg19 (or mm10-only) STAR genome index to compare uniquely mapped read counts between SCSit-labeled reads and original-method reads, revealing that SCSit yields roughly twice the mapped reads across five samples.

## When to use

Use when preprocessing SPLiT-seq FASTQ samples (e.g. SAMN08567259–SAMN08567264) with SCSit vs. the original filter and you must quantify alignment yield differences on a combined mm10-hg19 STAR reference.

## When NOT to use

- The input data is not SPLiT-seq combinatorial-barcoding data — the article states the fault-tolerant and indel alignment was proposed for SPLiT-seq data and application to other single-cell data is only potential, not demonstrated
- The goal is species-specific alignment without mixed-cell contamination (e.g. pure human samples), where a combined mm10-hg19 index is unnecessary
- You need cell-level clustering or expression matrices rather than raw mapped-read yield comparison — this skill covers alignment counting only

## Inputs

- Paired-end SPLiT-seq FASTQ files (R1/R2, Illumina 150 nt kits) for samples SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567264, and SAMN08567263
- SCSit-labeled FASTQ output reads (combinatorial barcodes and UMI in Read1)
- Original-method filtered reads (last-6-base barcode match, UMI phred ≤ 10 filtering, ≤ 1 mismatch per 8 nt cell barcode)
- Combined mm10-hg19 (or mm10) reference genome FASTA for STAR index building

## Outputs

- Uniquely mapped read counts per sample for both SCSit and original-method alignments
- Per-sample SCSit-to-original mapped-read ratio
- mapped_reads_comparison.csv containing all mapped read counts across the five samples

## How to apply

First preprocess each SPLiT-seq sample pair two ways: run SCSit (C program, multithreaded, default one mismatch plus one indel tolerance) to output labeled reads with combinatorial barcodes and UMI, and apply the original SPLiT-seq filtering (discard reads whose last 6 bases do not match the barcode, drop UMIs with more than 1 low-quality base at phred ≤ 10, and reject reads whose three 8 nt cell barcodes contain more than one mismatch). Build a STAR genome index on the combined mm10-hg19 reference (or mm10 alone for mouse-only samples such as SAMN08567263). Align the SCSit output reads and the original-method reads for each of the five samples with STAR and record uniquely mapped read counts per sample. Compute the SCSit-to-original mapped-read ratio per sample and assemble the counts into a comparison table. Accept the result when SCSit mapped reads are approximately twice the original method across the five samples, with the largest increment for SAMN08567264 (114,732,417 additional mapped reads).

## Related tools

- **SCSit** (Preprocesses SPLiT-seq FASTQ pairs to output labeled reads with combinatorial barcodes and UMI before STAR alignment) — https://github.com/shang-qian/SCSit
- **STAR** (Aligns SCSit-labeled and original-method reads to the combined mm10-hg19 (or mm10) genome and reports uniquely mapped read counts)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output && STAR --genomeDir mm10_hg19_index --readFilesIn output_R1.fastq output_R2.fastq
```

## Evaluation signals

- SCSit mapped reads are approximately twice the original method's mapped reads across all five samples
- The largest incremental mapped reads (114,732,417) is observed for sample SAMN08567264
- Mapped read counts for each sample (e.g. SAMN08567261 at 163,780,622, SAMN08567260 at 163,502,984, SAMN08567259 at 148,423,169, SAMN08567264 at 160,257,239) exceed the original-method counts
- mapped_reads_comparison.csv contains both SCSit and original uniquely mapped read counts plus per-sample ratios for all five samples
- Unidentified-read fraction in the original method should be 90–97% of uniquely identified reads in SCSit, per supplementary Table S3

## Limitations

- Quantitative comparison results are deferred to supplementary Table S3, which is cited but not linked in the text
- The mm10-hg19 combined index approach is specific to mixed human-mouse samples; mouse-only samples (SAMN08567263) map to mm10
- SCSit preprocessing requires python 3.6, and the STAR index build must match the genome combination used in the original study

## Evidence
