---
name: indel-tolerant-oligo-position-detection
description: Use when preprocessing SPLiT-seq or similar combinatorial-barcode single-cell
  FASTQ data and reads may contain a single mismatch or INDEL inside the barcode/index
  regions, so naive exact matching would discard them.
when_to_use_negative:
- The barcodes are longer than ~8 nt, where general aligners like BWA are no longer
  disadvantaged (the paper notes 8 bp is too short for BWA to be effective)
- Input is not SPLiT-seq-style combinatorial barcode data; indel-tolerant alignment
  was proposed only for SPLiT-seq and extension to other platforms is potential, not
  demonstrated
- Reads need tolerance beyond one mismatch and one indel, which is not the SCSit default
  and yields only 4-5% more identified reads at 20-40% more runtime
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: SCSit
  role: Compiled C program (multithreaded) that executes the sliding-window, mismatch-
    and INDEL-tolerant barcode/UMI identification on SPLiT-seq reads
  repo: https://github.com/shang-qian/SCSit
- name: Python 3.6
  role: Required runtime dependency for running SCSit
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:a83555a949c7e8df7b24e768808a6f96ea738397872a7a4b3f61ad82bd33d16b
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
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# indel-tolerant-oligo-position-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Locates short oligonucleotide sequences (e.g., SPLiT-seq combinatorial barcodes and their 30-nt index sequences) within reads using an 8-nt sliding window with tolerance for exactly one mismatch or one insertion/deletion, instead of discarding reads whose barcode regions carry indels.

## When to use

Use when preprocessing SPLiT-seq or similar combinatorial-barcode single-cell FASTQ data and reads may contain a single mismatch or INDEL inside the barcode/index regions, so naive exact matching would discard them.

## When NOT to use

- The barcodes are longer than ~8 nt, where general aligners like BWA are no longer disadvantaged (the paper notes 8 bp is too short for BWA to be effective)
- Input is not SPLiT-seq-style combinatorial barcode data; indel-tolerant alignment was proposed only for SPLiT-seq and extension to other platforms is potential, not demonstrated
- Reads need tolerance beyond one mismatch and one indel, which is not the SCSit default and yields only 4-5% more identified reads at 20-40% more runtime

## Inputs

- Paired-end SPLiT-seq FASTQ files (Read1 and Read2, Illumina 150-nt kits)
- barcodeList TXT file of 96-well plate oligonucleotides per barcode round (e.g., Table S1)
- primerList TXT file of forward/reverse oligonucleotide sequences (e.g., Table S2)

## Outputs

- Per-read calls of the three 8-nt combinatorial barcodes and the UMI
- Labeled-read FASTQ pair: Read1 composed of combinatorial barcodes plus UMI, with corresponding Read2

## How to apply

Parse the barcodeList (e.g., Table S1) to build joined index sequences index21/index32, split them into 23 segments by 8-nt k-mer, and encode indicator sequences via quad-to-decimal conversion (0,1,2,3 = A,T,G,C; intSeq = sum(seq[i]*4^(n-i-1))) for O(1) lookup. Scan each Read2 with an 8-nt sliding window to locate the 30-nt index sequences, accepting a complete match (Me-Ms = 30), one mismatch at position m (shift the start to Ms-m for m in 1-8, extend the end to Me+m for m in 22-30, keep unchanged for m in 9-21), or one INDEL (matched length 29 nt for deletion or 31 nt for insertion, with analogous boundary adjustments). Extract the UMI and the three 8-nt indicator barcodes from the located regions, validate each with the distance function Dist(x,y) requiring Dist < 2, and abandon Read2 otherwise. The rationale is that SCSit's default tolerance of one mismatch and one indel balances efficiency: relaxing to more errors only adds 4-5% identified reads at 20-40% more runtime.

## Related tools

- **SCSit** (Compiled C program (multithreaded) that executes the sliding-window, mismatch- and INDEL-tolerant barcode/UMI identification on SPLiT-seq reads) — https://github.com/shang-qian/SCSit
- **Python 3.6** (Required runtime dependency for running SCSit)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- Per-read agreement of the three barcodes and UMI with SCSit reference calls reaches the paper's 97% consistency level
- All accepted barcode indicator matches satisfy Dist(x,y) < 2; reads failing this are abandoned rather than force-called
- Located index sequences have expected matched lengths: 30 nt for complete match, 29 nt for one deletion, 31 nt for one insertion
- Labeled-read output FASTQ contains combinatorial barcodes plus UMI in Read1 with the corresponding Read2 retained
- Mapped reads should be roughly twice those of the original BLAST/BWA-based alignment method

## Limitations

- Default tolerance is restricted to one mismatch and one indel; higher tolerance was evaluated but not adopted because it increases runtime 20-40% for only 4-5% more identified reads (Table S4)
- Fault-tolerant and indel alignment were proposed for SPLiT-seq data; application to other barcode-based single-cell data is stated as potential, not demonstrated
- Quantitative comparisons are deferred to supplementary Table S3, cited without being provided in the text

## Evidence
