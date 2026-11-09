---
name: read-set-overlap-consistency-analysis
description: Use when validating a new SPLiT-seq barcode/UMI identification tool against
  the original baseline by computing the overlap fraction of identified reads per
  sample on paired-end FASTQ data.
when_to_use_negative:
- Input data is not SPLiT-seq combinatorial-barcoded paired-end FASTQ (SCSit's fault-tolerant
  and indel alignment was proposed for SPLiT-seq data; other platforms are only potential
  future use)
- Goal is already-mapped read quantification rather than barcode/UMI identification
  consistency (downstream mapping with STAR to mm10-hg19 or mm10 is a separate step)
- Only one sample is available and a per-sample cross-method consistency range across
  five samples is required
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SCSit
  role: C program run multithreaded (e.g. -t 4) to identify three rounds of barcode
    and UMI and output labeled reads per sample
  repo: https://github.com/shang-qian/SCSit
- name: Python 3.6
  role: Scripting environment for implementing the original SPLiT-seq filters and
    intersecting identified-read sets per sample
- name: BLAST
  role: Alignment baseline of the original SPLiT-seq identification method that SCSit
    consistency is compared against
- name: BWA
  role: Alternative alignment baseline cited with BLAST as the original method whose
    identified reads are compared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:abe4893fdefce028d89a0f800e61f97d2c131521c6655042548187c2802ea865
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# read-set-overlap-consistency-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the 96~97% per-sample consistency between reads identified by SCSit and reads identified by the original SPLiT-seq filtering method, by intersecting per-sample sets of (read ID, barcode combination, UMI) over five SPLiT-seq datasets.

## When to use

Use when validating a new SPLiT-seq barcode/UMI identification tool against the original baseline by computing the overlap fraction of identified reads per sample on paired-end FASTQ data.

## When NOT to use

- Input data is not SPLiT-seq combinatorial-barcoded paired-end FASTQ (SCSit's fault-tolerant and indel alignment was proposed for SPLiT-seq data; other platforms are only potential future use)
- Goal is already-mapped read quantification rather than barcode/UMI identification consistency (downstream mapping with STAR to mm10-hg19 or mm10 is a separate step)
- Only one sample is available and a per-sample cross-method consistency range across five samples is required

## Inputs

- Paired-end FASTQ files (R1/R2) for five SPLiT-seq samples (SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264)
- barcodeList (96-well plate oligonucleotide sequences per barcode round) in TXT format
- primerList (forward and reverse oligonucleotide sequences) in TXT format
- Original SPLiT-seq filter criteria (last-6-base barcode match, UMI phred <= 10 with >1 low-quality base allowed as filter, one-mismatch barcode threshold)

## Outputs

- Per-sample SCSit-identified labeled reads (FASTQ with combinatorial barcodes and UMI)
- Per-sample original-method identified read sets
- Per-sample overlap counts and consistency percentages (expected 96-97%)
- Summary table of identified-read counts, overlaps, and consistency per sample

## How to apply

Run SCSit (a C program, multithreaded, e.g. -t 4) on each of five paired-end FASTQ sample pairs to obtain labeled reads with combinatorial barcodes and UMI. Independently implement the original SPLiT-seq baseline per sample: discard reads whose last 6 bases do not match the barcode sequence, filter UMIs containing more than 1 low-quality base (phred <= 10), and reject reads whose three 8 nt cell barcodes contain more than one mismatch. Restrict both methods to the same filtered clean data and intersect the identified-read sets (read ID, barcode combination, UMI) per sample, computing the fraction of original-method reads recovered by SCSit. Record per-sample counts, overlaps, and consistency percentages in a summary table; the article reports the two methods agree at 96~97% across the five samples (mouse SAMN08567263 plus four mixed human/mouse samples).

## Related tools

- **SCSit** (C program run multithreaded (e.g. -t 4) to identify three rounds of barcode and UMI and output labeled reads per sample) — https://github.com/shang-qian/SCSit
- **Python 3.6** (Scripting environment for implementing the original SPLiT-seq filters and intersecting identified-read sets per sample)
- **BLAST** (Alignment baseline of the original SPLiT-seq identification method that SCSit consistency is compared against)
- **BWA** (Alternative alignment baseline cited with BLAST as the original method whose identified reads are compared)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- Per-sample consistency between SCSit-identified and original-method-identified reads falls in the reported 96-97% range for all five samples
- Both methods are run on the same filtered clean data before intersecting (read ID, barcode combination, UMI) sets
- Original-method filters applied exactly: last-6-base barcode match enforced, UMI with >1 low-quality base (phred <= 10) filtered, barcodes with more than one mismatch discarded
- Summary table contains per-sample identified-read counts, overlap counts, and consistency percentages for all five datasets (SAMN08567259/60/61/63/64)

## Limitations

- SCSit's default tolerance is one mismatch and one indel; higher tolerance was evaluated but yields only 4~5% more identified reads at 20~40% more runtime (Table S4)
- Quantitative comparison details are deferred to supplementary Table S3, which is cited but not provided or linked in the text
- The comparison scope is SPLiT-seq data; application to other barcode-based single-cell data is stated as potential, not demonstrated

## Evidence
