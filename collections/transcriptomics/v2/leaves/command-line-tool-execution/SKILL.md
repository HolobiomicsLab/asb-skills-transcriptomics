---
name: command-line-tool-execution
description: Use when raw paired-end FASTQ data from SPLiT-seq combinatorial barcoding
  must be preprocessed to identify cell barcodes and UMIs, and a fast command-line
  tool call (scsit -r1 ... -r2 ... -p ... -b ...) is the intended execution path.
when_to_use_negative:
- Input data is not from SPLiT-seq combinatorial barcoding — the fault-tolerant and
  indel alignment was proposed specifically for SPLiT-seq; support for other single-cell
  platforms is stated only as potential/future work
- Barcodes are long enough for standard aligners — the paper notes BWA fails because
  the 8 bp barcodes are too short, but for other datasets a generic alignment tool
  may be appropriate
- You need tolerance beyond one mismatch and one indel — this is not enabled by default
  and only gains 4–5% identified reads at 20–40% more runtime
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SCSit
  role: The command-line preprocessing tool being executed; identifies three rounds
    of combinatorial barcodes and UMI from SPLiT-seq paired-end FASTQ input
  repo: https://github.com/shang-qian/SCSit
- name: Python 3.6
  role: Runtime dependency required for installing/running SCSit via conda or pip
- name: C
  role: Implementation language of the precompiled multithreaded SCSit binary built
    via ./make.sh for x86-64 CPUs
  repo: https://github.com/shang-qian/SCSit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:05c1764e03eb201c73dd6f5603a2dee69f991895b338a2c355ef7d6077337d52
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
  - C
  - Python
  - Python 3.6
  - SCSit
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# command-line-tool-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run SCSit, a C-based multithreaded command-line preprocessing tool, on paired-end SPLiT-seq FASTQ files to identify the three rounds of combinatorial barcodes and UMI, yielding labeled Read1/Read2 FASTQ pairs. This skill covers installing SCSit (conda/pip or compiled C binary) and invoking it correctly with its required arguments.

## When to use

Use when raw paired-end FASTQ data from SPLiT-seq combinatorial barcoding must be preprocessed to identify cell barcodes and UMIs, and a fast command-line tool call (scsit -r1 ... -r2 ... -p ... -b ...) is the intended execution path.

## When NOT to use

- Input data is not from SPLiT-seq combinatorial barcoding — the fault-tolerant and indel alignment was proposed specifically for SPLiT-seq; support for other single-cell platforms is stated only as potential/future work
- Barcodes are long enough for standard aligners — the paper notes BWA fails because the 8 bp barcodes are too short, but for other datasets a generic alignment tool may be appropriate
- You need tolerance beyond one mismatch and one indel — this is not enabled by default and only gains 4–5% identified reads at 20–40% more runtime

## Inputs

- Paired-end SPLiT-seq Read1 and Read2 FASTQ files (any quality-encoding variant), e.g. from SRA/BioSample accessions SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264
- primerList TXT file containing all oligonucleotide primer sequences used
- barcodeList TXT file with the 96-well plate oligonucleotides for each round of barcodes

## Outputs

- Output Read1 FASTQ file composed of combinatorial barcodes and UMI
- Corresponding output Read2 FASTQ file
- Identified-read counts and ratios per sample for comparison with the original SPLiT-seq method

## How to apply

Install SCSit via conda (`conda install -c susiew01 scsit`), pip (`pip install scsit`), or by cloning https://github.com/shang-qian/SCSit and running ./make.sh for the precompiled x86-64 C binary; Python 3.6 is required. Prepare the two required TXT inputs: primerList (all oligonucleotide primer sequences, forward and reverse per Table S2) and barcodeList (the 96-well plate oligonucleotides for each round of barcodes). Run SCSit on each paired-end sample with the recommended invocation `scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output`, keeping the default tolerance of one mismatch and one indel for barcode alignment (the article's stated optimal balance of efficiency). SCSit operates on a pair of files multithreaded at a time and writes a FASTQ pair: Read1 composed of combinatorial barcodes and UMI, and the corresponding Read2 sequencing data. Verify execution by confirming both output FASTQ files appear for every sample and that the finished-percentage reporting completed for each large input pair.

## Related tools

- **SCSit** (The command-line preprocessing tool being executed; identifies three rounds of combinatorial barcodes and UMI from SPLiT-seq paired-end FASTQ input) — https://github.com/shang-qian/SCSit
- **Python 3.6** (Runtime dependency required for installing/running SCSit via conda or pip)
- **C** (Implementation language of the precompiled multithreaded SCSit binary built via ./make.sh for x86-64 CPUs) — https://github.com/shang-qian/SCSit

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- Both output FASTQ files (Read1 and Read2) are produced for every input sample pair
- Output Read1 reads contain combinatorial barcodes and UMI in the expected format
- Identified-read ratios closely match the reported SCSit values (66.98%, 75.84%, 74.89%, 73.22%, 66.26%) with small deviation tolerance, and every rate stays above 65%
- Identified-read counts (e.g. 163,780,622 for SAMN08567261; 148,423,169 for SAMN08567259) exceed the original SPLiT-seq method's counts for the same samples
- A per-sample comparison table (e.g. scsit_identified_reads_comparison.csv) assembles counts and ratios with no missing samples

## Limitations

- SCSit default setting is restricted to one mismatch and one indel; higher mismatch/indel tolerance is not enabled by default and evaluated variants only increase identified reads 4–5% while taking 20–40% more runtime (Table S4)
- Fault-tolerant and indel alignment was proposed for SPLiT-seq data; applicability to other barcode-based single-cell data is potential, not demonstrated, and support for additional single-cell sequencing platforms is deferred to future updates
- SCSit only supports FASTQ input files (all quality-encoding variants); other formats are rejected

## Evidence
