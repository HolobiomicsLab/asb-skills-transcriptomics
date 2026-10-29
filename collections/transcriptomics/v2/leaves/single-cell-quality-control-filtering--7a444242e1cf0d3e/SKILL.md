---
name: single-cell-quality-control-filtering
description: Use when raw SPLiT-seq paired-end FASTQ reads need barcode/UMI identification
  and quality filtering, and more retained, mappable reads are needed than the original
  filter that discards reads whose last 6 bases fail to match the barcode.
when_to_use_negative:
- Input data is not SPLiT-seq combinatorial-barcoded single-cell data; the fault-tolerant
  and indel alignment is proposed only for SPLiT-seq, and support for other platforms
  is deferred to future updates
- Barcodes are longer than 8 nt, where aligners like BWA can work effectively; SCSit
  targets the short 8-nt SPLiT-seq barcodes that BWA cannot exploit
- You only need quality-trimming of sequencing reads without barcode/UMI identification
  (e.g. generic FASTQ QC tools); this skill is specifically about combinatorial barcode
  and UMI classification
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCSit
  role: Multithreaded C preprocessing tool that identifies three rounds of barcode
    and UMI with one mismatch and one indel tolerance and outputs labeled FASTQ reads
  repo: https://github.com/shang-qian/SCSit
- name: STAR
  role: Aligns SCSit-labeled and original-method reads to the combined mm10-hg19 or
    mm10 genome to quantify uniquely mapped reads for the comparison
- name: Python 3.6
  role: Required working installation dependency for running SCSit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:134c167439c2c1261fa51e24057cd802032315707968e3eeca806fa5fa456fee
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
  - Python 3.6
  - SCSit
  - STAR
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# single-cell-quality-control-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess SPLiT-seq single-cell RNA-seq FASTQ pairs by fault-tolerant identification of three 8-nt combinatorial cell barcodes and UMI using SCSit (one mismatch and one indel tolerance), yielding labeled reads that map about twice as well as reads processed with the original SPLiT-seq filtering.

## When to use

Use when raw SPLiT-seq paired-end FASTQ reads need barcode/UMI identification and quality filtering, and more retained, mappable reads are needed than the original filter that discards reads whose last 6 bases fail to match the barcode.

## When NOT to use

- Input data is not SPLiT-seq combinatorial-barcoded single-cell data; the fault-tolerant and indel alignment is proposed only for SPLiT-seq, and support for other platforms is deferred to future updates
- Barcodes are longer than 8 nt, where aligners like BWA can work effectively; SCSit targets the short 8-nt SPLiT-seq barcodes that BWA cannot exploit
- You only need quality-trimming of sequencing reads without barcode/UMI identification (e.g. generic FASTQ QC tools); this skill is specifically about combinatorial barcode and UMI classification

## Inputs

- Paired-end SPLiT-seq FASTQ files (Illumina 150-nt kits), e.g. samples SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264
- primerList TXT (forward and reverse oligonucleotide primer sequences)
- barcodeList TXT (96-well plate oligonucleotides for each round of barcodes)
- Combined mm10-hg19 or mm10 reference genome with STAR index

## Outputs

- Output FASTQ pair per sample: Read1 carrying combinatorial barcodes and UMI, Read2 as corresponding sequence data
- STAR alignment summary with uniquely mapped read counts per sample per method
- Comparison table (e.g. mapped_reads_comparison.csv) of SCSit vs original mapped-read ratios

## How to apply

Run SCSit, a C program that parallelizes by multithread (-t), on each pair of SPLiT-seq FASTQ files with primerList (-p) and the 96-well plate barcodeList (-b); it slides an 8-nt window across Read2, maps index sequences to 23 k-mer segments, and uses quad-to-decimal conversion (0,1,2,3 for A,T,G,C) to identify the three barcodes. Apply SCSit's default tolerance of one mismatch and one indel (Dist threshold < 2 abandons Read2), plus primer trimming of Read1, and retain valid Read1 with combinatorial barcodes and UMI in the output FASTQ pair. For comparison, apply the original SPLiT-seq filtering to the same samples: discard reads whose last 6 bases do not match the barcode, drop UMIs with more than 1 low-quality base (phred <= 10), and reject reads whose three 8-nt cell barcodes contain more than one mismatch. Finally map both read sets to the combined mm10-hg19 (or mm10) genome with STAR and compare uniquely mapped counts.

## Related tools

- **SCSit** (Multithreaded C preprocessing tool that identifies three rounds of barcode and UMI with one mismatch and one indel tolerance and outputs labeled FASTQ reads) — https://github.com/shang-qian/SCSit
- **STAR** (Aligns SCSit-labeled and original-method reads to the combined mm10-hg19 or mm10 genome to quantify uniquely mapped reads for the comparison)
- **Python 3.6** (Required working installation dependency for running SCSit)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- SCit mapped reads are approximately twice those of the original alignment method across all five samples when aligned with STAR to mm10-hg19
- The largest mapped-read increment is observed for SAMN08567264 (114,732,417 additional reads; total 160,257,239 vs the original method)
- Consistency of identified reads from SCSit reaches about 97%
- Read2 whose Dist value is >= threshold 2 is abandoned; output FASTQ Read1 must contain combinatorial barcodes and UMI
- The UMI- and barcode-unidentified reads in the original method are 90-97% of uniquely identified reads in SCSit (supplementary Table S3)

## Limitations

- SCSit defaults to one mismatch and one indel; higher tolerance increases identified reads by only 4-5% but costs 20-40% more runtime (Table S4) and is not enabled by default
- Quantitative comparisons are deferred to supplementary Table S3/S4, cited without full data in the main text
- SCSit was proposed for SPLiT-seq data; application to other barcode-based single-cell data is potential, not demonstrated, and support for additional platforms is deferred to future updates
- SCSit requires a working installation of Python 3.6 and is optimized for x86-64 CPUs

## Evidence
