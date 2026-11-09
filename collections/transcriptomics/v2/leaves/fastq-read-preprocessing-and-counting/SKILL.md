---
name: fastq-read-preprocessing-and-counting
description: Use when raw paired-end SPLiT-seq FASTQ files need barcode/UMI identification
  and you must count identified labeled reads and compute identified-read ratios per
  sample.
when_to_use_negative:
- Input data is not from SPLiT-seq or a similar combinatorial barcoding platform —
  fault-tolerant and indel alignment was proposed for SPLiT-seq data; support for
  other single-cell platforms is only potential, not demonstrated
- Reads have already been barcode-identified and labeled (e.g., an existing labeled-read
  FASTQ) — recounting would duplicate the SCSit step
- You need tolerance beyond one mismatch and one indel without accepting the trade-off
  — the authors found more tolerance yields only 4–5% more identified reads at 20–40%
  more runtime
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: SCSit
  role: C-program, multithreaded preprocessing tool that identifies three rounds of
    barcode and UMI in SPLiT-seq reads and outputs labeled read pairs
  repo: https://github.com/shang-qian/SCSit
- name: Python 3.6
  role: Required working installation for running SCSit installed via conda or pip
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:5924677ed9a5b29403fcc6c8b33daca771f262ed2e7d0d7aee90c83ab5b2ee2a
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# fastq-read-preprocessing-and-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess SPLiT-seq paired-end FASTQ reads with SCSit to identify combinatorial barcodes and UMIs, output labeled reads, and quantify identified-read counts and ratios per sample against the original SPLiT-seq method.

## When to use

Use when raw paired-end SPLiT-seq FASTQ files need barcode/UMI identification and you must count identified labeled reads and compute identified-read ratios per sample.

## When NOT to use

- Input data is not from SPLiT-seq or a similar combinatorial barcoding platform — fault-tolerant and indel alignment was proposed for SPLiT-seq data; support for other single-cell platforms is only potential, not demonstrated
- Reads have already been barcode-identified and labeled (e.g., an existing labeled-read FASTQ) — recounting would duplicate the SCSit step
- You need tolerance beyond one mismatch and one indel without accepting the trade-off — the authors found more tolerance yields only 4–5% more identified reads at 20–40% more runtime

## Inputs

- Paired-end FASTQ files from SPLiT-seq (Read1 and Read2, any quality encoding variant)
- primerList TXT file with forward and reverse oligonucleotide primer sequences
- barcodeList TXT file with the 96 well plate oligonucleotides for each round of barcodes

## Outputs

- Output Read1 FASTQ file composed of combinatorial barcodes and UMI for labeled reads
- Corresponding output Read2 FASTQ file of sequencing data
- Per-sample identified-read counts and ratios (e.g., scsit_identified_reads_comparison.csv)

## How to apply

Download raw paired-end FASTQ files for each sample (e.g., SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264) from their SRA/BioSample accessions. Install SCSit via conda ('conda install -c susiew01 scsit'), pip ('pip install scsit'), or compile the C binary from GitHub, ensuring Python 3.6 is available. Run SCSit on each read pair with the primer list and 96-well plate barcode list, keeping the default tolerance of one mismatch and one indel, which the authors chose as the optimal balance of efficiency. For each sample, count the labeled reads in the output Read1 FASTQ (composed of combinatorial barcodes and UMI) and compute the identified-read ratio relative to total input reads; validate that ratios closely match the reported values (66.98%, 75.84%, 74.89%, 73.22%, 66.26%), each above 65%.

## Related tools

- **SCSit** (C-program, multithreaded preprocessing tool that identifies three rounds of barcode and UMI in SPLiT-seq reads and outputs labeled read pairs) — https://github.com/shang-qian/SCSit
- **Python 3.6** (Required working installation for running SCSit installed via conda or pip)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- Identified-read ratios for the five SPLiT-seq samples closely match the reported SCSit values (73.22%, 74.89%, 75.84%, 66.98%, 66.26%) within a small deviation tolerance
- Every per-sample identified-read ratio exceeds 65%
- Identified-read counts reach the reported magnitudes: SAMN08567263 = 56,833,343; SAMN08567261 = 163,780,622; SAMN08567260 = 163,502,984; SAMN08567259 = 148,423,169; SAMN08567264 = 160,257,239
- An output FASTQ pair (labeled Read1 with combinatorial barcodes and UMI, plus corresponding Read2) is produced for every sample
- Counts exceed those of the original SPLiT-seq method for all five samples

## Limitations

- SCSit default configuration is restricted to one mismatch and one indel; higher mismatch/indel tolerance is not enabled by default
- Loosening tolerance beyond one mismatch and one indel only increases identified reads by 4–5% but takes 20–40% more runtime (Table S4)
- Support for additional single-cell sequencing platforms is deferred to future updates and not yet available
- Quantitative comparison details are deferred to supplementary Table S3, cited without being provided or linked in the text

## Evidence
