---
name: spliseq-read2-barcode-index-kmer-matching
description: Use when preprocessing SPLiT-seq paired-end FASTQ to extract cell-type
  barcodes and UMI from Read2, especially when reads may contain insertions or deletions
  in the 8-nt barcode index regions.
when_to_use_negative:
- Data are not SPLiT-seq-style combinatorially barcoded paired-end reads (SCSit fault-tolerant/indel
  alignment was proposed for SPLiT-seq only; other platforms are deferred to future
  updates).
- Barcodes are long enough for standard aligners — the paper notes BWA fails here
  only because the 8-bp barcodes are too short.
- You require tolerance beyond one mismatch and one indel per read; SCSit's default
  is restricted and higher tolerance was evaluated but not adopted.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: SCSit
  role: C program (compiled, multithreaded) whose k-mer index matching logic this
    skill re-implements; also serves as the reference implementation for agreement
    checks
  repo: https://github.com/shang-qian/SCSit
- name: Python 3.6
  role: Implementation language for the independent re-implementation of barcode/UMI
    identification and validation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:42703caee690ea160a6019b6304525427274538f0ec16a2cb7aec24378e855f5
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
  - Python
  - Python 3.6
  - SCSit
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# spliseq-read2-barcode-index-kmer-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs SCSit's k-mer sliding-window identification of the three combinatorial barcodes and UMI in each SPLiT-seq Read2, tolerating one mismatch and one indel via 8-nt segment matching and quad-to-decimal encoded barcode lookups.

## When to use

Use when preprocessing SPLiT-seq paired-end FASTQ to extract cell-type barcodes and UMI from Read2, especially when reads may contain insertions or deletions in the 8-nt barcode index regions.

## When NOT to use

- Data are not SPLiT-seq-style combinatorially barcoded paired-end reads (SCSit fault-tolerant/indel alignment was proposed for SPLiT-seq only; other platforms are deferred to future updates).
- Barcodes are long enough for standard aligners — the paper notes BWA fails here only because the 8-bp barcodes are too short.
- You require tolerance beyond one mismatch and one indel per read; SCSit's default is restricted and higher tolerance was evaluated but not adopted.

## Inputs

- SPLiT-seq paired-end FASTQ files (Read1 and Read2, any quality encoding variant, Illumina 150-nt kits)
- barcodeList TXT file (96-well plate oligonucleotide sequences for each round of barcodes, e.g. Table S1)
- primerList TXT file (forward and reverse primer oligonucleotide sequences, e.g. Table S2)

## Outputs

- Per-read calls of the three 8-nt combinatorial barcodes and the UMI extracted from Read2
- Labeled-read output FASTQ pair: Read1 composed of combinatorial barcodes plus UMI, Read2 the corresponding sequence, with primers trimmed

## How to apply

Parse the barcodeList into the joined index sequences index21/index32, split them into 23 segments of 8-nt k-mers, and encode indicator sequences by quad-to-decimal conversion (0,1,2,3 = A,T,G,C) with intSeq = sum(seq[i]*4^(n-i-1)) for O(1) lookup. Scan Read2 with an 8-nt sliding window to locate the 30-nt index sequences, accepting a complete match (Me-Ms = 30), one mismatch at position m (shift start to Ms-m for m in 1-8, extend end to Me+m for m in 22-30, unchanged for m in 9-21), or one INDEL (29 nt for deletion, 31 nt for insertion, with analogous boundary adjustments). Extract the UMI and three 8-nt indicator barcodes from the located regions and validate each with the distance function Dist(x,y), keeping Read2 only when Dist < 2 for all barcodes; retain valid Read1 with its Read2 and trim primers via primerList. The one-mismatch/one-indel default balances efficiency: loosening tolerance only gains 4-5% reads at 20-40% more runtime.

## Related tools

- **SCSit** (C program (compiled, multithreaded) whose k-mer index matching logic this skill re-implements; also serves as the reference implementation for agreement checks) — https://github.com/shang-qian/SCSit
- **Python 3.6** (Implementation language for the independent re-implementation of barcode/UMI identification and validation)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- Join per-read calls with SCSit reference output by read ID and confirm ≥97% agreement of the three barcodes and UMI (barcode_agreement_report.csv).
- Verify located index-sequence span length: 30 nt for complete matches, 29 nt for a deletion, 31 nt for an insertion.
- Confirm every emitted read satisfies Dist(x,y) < 2 for all three 8-nt indicator barcodes; reads failing this are abandoned.
- Check that invalid Read2 reads are excluded and each retained Read1 carries the combinatorial barcode plus UMI with primers trimmed per primerList.
- Compare labeled-read yield against BLAST/BWA-based original SPLiT-seq processing: SCSit-style output should roughly double mapped reads.

## Limitations

- Default tolerance is one mismatch and one indel; permitting more yields only 4-5% more identified reads at 20-40% more runtime (Table S4) and is not enabled by default.
- The fault-tolerant and indel alignment was proposed for SPLiT-seq data; applicability to other barcode-based single-cell platforms is potential, not demonstrated.
- Support for additional single-cell sequencing platforms is deferred to future updates and not yet available.
- Read2 is abandoned outright when any indicator barcode has Dist ≥ 2 against the barcode list.

## Evidence
