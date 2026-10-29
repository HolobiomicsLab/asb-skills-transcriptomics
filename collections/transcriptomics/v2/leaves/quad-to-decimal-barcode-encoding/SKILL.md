---
name: quad-to-decimal-barcode-encoding
description: Use when identifying fixed-length cell barcodes or UMI flanking sequences
  in SPLiT-seq reads and you need constant-time matching of 8-nt indicator sequences
  against a barcodeList instead of per-read alignment (e.g. BLAST/BWA).
when_to_use_negative:
- Barcode or k-mer length is much longer than 8 bp, where full aligners such as BWA
  remain appropriate (the paper notes 8 bp is too short for BWA to exploit)
- You need tolerance beyond one mismatch/one indel per barcode by default; SCSit restricts
  the default to one mismatch and one indel, and looser settings cost 20–40% more
  runtime for only 4–5% more reads
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: SCSit
  role: C program whose barcode identification uses quad-to-decimal encoding of the
    8-nt indicator sequences for O(1) list-index lookup
  repo: https://github.com/shang-qian/SCSit
- name: Python 3.6
  role: Runtime dependency required to run SCSit, which performs the encoded barcode
    lookups
- name: BLAST
  role: Baseline alignment approach that the encoding replaces because blastn-short
    barcode determination runs serially on a single CPU core
- name: BWA
  role: Baseline aligner deemed unsuitable since the 8-bp barcodes are too short for
    BWA sequence alignment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:95122bfbdd5aaa5fbc4be3d9b94e7fe71aac20cb42a6ebb3bb57b6152ff62d41
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
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# quad-to-decimal-barcode-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An O(1) barcode-lookup encoding from SCSit in which nucleotides are mapped to quaternary digits (A,T,G,C = 0,1,2,3) and each 8-nt SPLiT-seq indicator barcode is converted via intSeq = sum(seq[i]*4^(n-i-1)) to an integer used as a direct list index during barcode identification.

## When to use

Use when identifying fixed-length cell barcodes or UMI flanking sequences in SPLiT-seq reads and you need constant-time matching of 8-nt indicator sequences against a barcodeList instead of per-read alignment (e.g. BLAST/BWA).

## When NOT to use

- Barcode or k-mer length is much longer than 8 bp, where full aligners such as BWA remain appropriate (the paper notes 8 bp is too short for BWA to exploit)
- You need tolerance beyond one mismatch/one indel per barcode by default; SCSit restricts the default to one mismatch and one indel, and looser settings cost 20–40% more runtime for only 4–5% more reads

## Inputs

- barcodeList (Table S1) of 96-well plate oligonucleotide sequences per round of barcodes (TXT)
- 8-nt indicator barcode windows extracted from Read2 of SPLiT-seq paired-end FASTQ

## Outputs

- Integer-encoded index sequences enabling O(1) lookup of barcode identities
- Per-read calls of the three 8-nt indicator barcodes (accepted when Dist(x,y) < 2)

## How to apply

Build the joined index sequences index21/index32 from the barcodeList (Table S1), split them into 23 segments of 8-nt k-mers, and encode every segment by quad-to-decimal conversion using 0, 1, 2, 3 for A, T, G, C with intSeq = sum(seq[i]*4^(n-i-1)). Encode each 8-nt window extracted from Read2 the same way, so a candidate indicator sequence maps to a list index in O(1). Validate the match with the distance function Dist(x, y), accepting the barcode call only when Dist < 2 (i.e. at most one mismatch), otherwise abandoning Read2. This encoding underpins SCSit's fault-tolerant identification of the three rounds of combinatorial barcodes and the UMI, replacing serial BLAST/BWA alignment that is slow or impractical for 8-bp barcodes.

## Related tools

- **SCSit** (C program whose barcode identification uses quad-to-decimal encoding of the 8-nt indicator sequences for O(1) list-index lookup) — https://github.com/shang-qian/SCSit
- **Python 3.6** (Runtime dependency required to run SCSit, which performs the encoded barcode lookups)
- **BLAST** (Baseline alignment approach that the encoding replaces because blastn-short barcode determination runs serially on a single CPU core)
- **BWA** (Baseline aligner deemed unsuitable since the 8-bp barcodes are too short for BWA sequence alignment)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- Per-read agreement of the three barcodes and UMI with reference SCSit calls reaches ~97% consistency when joined by read ID
- Every accepted barcode call satisfies Dist(x, y) < 2 (at most one mismatch per 8-nt indicator); reads failing this are abandoned
- Each 8-nt window encodes to a unique integer within the 23-segment index list (0 <= intSeq < 4^8) with no index collisions among valid barcodeList entries
- Number of labeled reads output is roughly twice that of the original BLAST/BWA-based SPLiT-seq alignment method

## Limitations

- SCSit default tolerance is one mismatch and one indel; higher tolerance identifies only 4–5% more reads at 20–40% more runtime (Table S4) and is not enabled by default
- The fault-tolerant and indel-aware alignment was proposed for SPLiT-seq data; application to other barcode-based single-cell data is stated as potential, not demonstrated
- 8-bp barcode length is too short for general aligners like BWA, so the encoding approach is specific to short fixed-length indicator sequences

## Evidence
