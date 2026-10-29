---
name: multithreaded-fastq-preprocessing
description: Use when preprocessing SPLiT-seq paired-end FASTQ reads that require
  combinatorial barcode/UMI identification and the serial BLAST-based original method
  is too slow or CPU-bound; especially when runtime must be under 10% of the original.
when_to_use_negative:
- The data are not SPLiT-seq combinatorial-barcode single-cell reads — fault-tolerant
  and indel alignment was proposed only for SPLiT-seq data; other platforms are stated
  as potential future support, not yet available
- Barcodes are too short for a general aligner such as BWA (8 bp) and the task is
  generic alignment rather than SPLiT-seq barcode/UMI identification
- Tolerance beyond one mismatch and one indel is required — SCSit's default setting
  restricts to one mismatch and one indel; loosening yields only 4–5% more identified
  reads at 20–40% more runtime (Table S4)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: SCSit
  role: Multithreaded C program that directly identifies the three rounds of SPLiT-seq
    combinatorial barcodes and UMI, replacing serial BLAST alignment
  repo: https://github.com/shang-qian/SCSit
- name: BLAST
  role: Original SPLiT-seq preprocessing baseline (blastn-short, single CPU core,
    serial) against which runtime is compared
- name: BWA
  role: Faster alternative alignment tool evaluated but rejected because 8 bp barcodes
    are too short for it to take advantage of sequence alignment
- name: Python 3.6
  role: Working installation required by SCSit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:16c3b036bde36900df6284e0d067598931289a5b36a94c7c81cf3d8dc0422b68
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
  - Python 3.6
  - SCSit
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multithreaded-fastq-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Speed up preprocessing of paired-end SPLiT-seq FASTQ data by running SCSit, a C program that identifies the three rounds of combinatorial barcodes and UMI directly with multithreaded parallel processing, instead of serial single-core BLAST (blastn-short) alignment.

## When to use

Use when preprocessing SPLiT-seq paired-end FASTQ reads that require combinatorial barcode/UMI identification and the serial BLAST-based original method is too slow or CPU-bound; especially when runtime must be under 10% of the original.

## When NOT to use

- The data are not SPLiT-seq combinatorial-barcode single-cell reads — fault-tolerant and indel alignment was proposed only for SPLiT-seq data; other platforms are stated as potential future support, not yet available
- Barcodes are too short for a general aligner such as BWA (8 bp) and the task is generic alignment rather than SPLiT-seq barcode/UMI identification
- Tolerance beyond one mismatch and one indel is required — SCSit's default setting restricts to one mismatch and one indel; loosening yields only 4–5% more identified reads at 20–40% more runtime (Table S4)

## Inputs

- Paired-end SPLiT-seq FASTQ files (R1 and R2, any quality-encoding variant) from Illumina 150 nt kits
- Primer list TXT file of all oligonucleotide sequences used (primerList, Table S2)
- Barcode list TXT file of 96-well plate oligonucleotides for each round of barcodes
- Thread count for multithreaded execution (paper used 4 CPU cores)

## Outputs

- Output FASTQ file pair: Read1 annotated with combinatorial barcodes and UMI, Read2 as corresponding sequencing data
- Wall-clock runtime per sample for runtime comparison against the original blastn-short method (e.g. 0.62 h vs 17.68 h for SAMN08567263)

## How to apply

Install SCSit (Anaconda/PyPi or the precompiled C binary from the GitHub release page; Python 3.6 is required) and prepare the primer list (Table S2) and the 96-well barcode oligonucleotide list as TXT files. Run SCSit on a pair of FASTQ files with -r1/-r2, -p, -b, an output prefix (-o), and a thread count (-t), e.g. -t 4 to match the paper's 4-core runtime assessment; SCSit operates a pair of files multithreading at a time. Internally, Read2 index sequences are divided into 23 segments by 8 nt k-mer sliding window, quad-to-decimal conversion (0/1/2/3 = A/T/G/C) identifies indicator sequences, valid Read2 allow Read1 to be retained and primer-trimmed, and barcode identification accepts one mismatch and one indel by default. Measure wall-clock time and compare against the original SPLiT-seq preprocessing (blastn-short based, discarding reads whose last 6 bases do not match the barcode, filtering UMIs with more than 1 low-quality base at phred ≤ 10, and rejecting 8 nt barcodes with more than one mismatch) under identical core counts.

## Related tools

- **SCSit** (Multithreaded C program that directly identifies the three rounds of SPLiT-seq combinatorial barcodes and UMI, replacing serial BLAST alignment) — https://github.com/shang-qian/SCSit
- **BLAST** (Original SPLiT-seq preprocessing baseline (blastn-short, single CPU core, serial) against which runtime is compared)
- **BWA** (Faster alternative alignment tool evaluated but rejected because 8 bp barcodes are too short for it to take advantage of sequence alignment)
- **Python 3.6** (Working installation required by SCSit)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- SCSit-to-original runtime ratio is below 10% for all five datasets (SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264) under identical 4-core conditions
- Consistency of identified reads reaches 97% and mapped reads (after STAR mapping to mm10-hg19 or mm10) are twice those of the original blastn-short method
- Output Read1 FASTQ entries carry combinatorial barcodes and UMI, and Read2 FASTQ corresponds to the same reads
- Accepted Read2 identifications satisfy the default tolerance: complete index match, or one mismatch at position 9–21 with W1 and W23 fully matched, or one INDEL (index length ≠ 30 nt)
- Number of identified/labeled reads exceeds the original method, consistent with the paper's finding that SCSit obtains more labeled reads via exact insertion/deletion alignment

## Limitations

- SCSit's default tolerance is one mismatch and one indel; higher tolerance was evaluated but not adopted because it only increases identified reads 4–5% while taking 20–40% more runtime (Table S4)
- Fault-tolerant and indel alignment were proposed for SPLiT-seq data only; application to other barcode-based single-cell platforms is potential, not demonstrated, and support for more platforms is deferred to future updates
- Quantitative comparison details are deferred to supplementary Table S3, cited without being provided or linked in the text
- Large FASTQ files can take a couple of minutes to open; newly opened files only show finished percentage while running

## Evidence
