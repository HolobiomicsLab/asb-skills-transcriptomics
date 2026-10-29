---
name: single-cell-read-labeling-qc
description: Use when raw paired-end SPLiT-seq FASTQ files need preprocessing to label
  reads with combinatorial barcodes and UMI, and identified-read counts/ratios must
  be reproduced or compared to published values.
when_to_use_negative:
- Input is not SPLiT-seq combinatorial-barcoding data — fault-tolerant and indel alignment
  were proposed for SPLiT-seq data, with application to other barcode-based single-cell
  data only potential, not demonstrated
- Barcodes are longer than 8 nt and require general alignment — the article notes
  BWA cannot take advantage of 8 bp barcode length, but SCSit is purpose-built for
  SPLiT-seq's three 8 nt barcodes, not generic adapter or alignment tasks
- Reads are already labeled with cell barcodes and UMI (post-preprocessing BAM/FASTQ)
  — rerunning SCSit would only re-identify existing labels
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: SCSit
  role: Primary C multithreaded tool that identifies three rounds of barcode and UMI
    from SPLiT-seq paired-end reads and outputs labeled reads, using one-mismatch
    and one-indel tolerance by default
  repo: https://github.com/shang-qian/SCSit
- name: Python 3.6
  role: Required runtime for installing and running SCSit via conda or pip
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:08f27de90d67d1afc32eb35994b47a82835899013f7b803e67fd8ec152aa7d07
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
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-read-labeling-qc

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce and quality-control SPLiT-seq read labeling by running SCSit on paired-end FASTQ data to identify combinatorial barcodes and UMIs, then comparing identified-read counts and ratios (e.g. 66.26%–75.84% across five SPLiT-seq samples) against the original method.

## When to use

Use when raw paired-end SPLiT-seq FASTQ files need preprocessing to label reads with combinatorial barcodes and UMI, and identified-read counts/ratios must be reproduced or compared to published values.

## When NOT to use

- Input is not SPLiT-seq combinatorial-barcoding data — fault-tolerant and indel alignment were proposed for SPLiT-seq data, with application to other barcode-based single-cell data only potential, not demonstrated
- Barcodes are longer than 8 nt and require general alignment — the article notes BWA cannot take advantage of 8 bp barcode length, but SCSit is purpose-built for SPLiT-seq's three 8 nt barcodes, not generic adapter or alignment tasks
- Reads are already labeled with cell barcodes and UMI (post-preprocessing BAM/FASTQ) — rerunning SCSit would only re-identify existing labels

## Inputs

- Paired-end FASTQ files (Read1/Read2) from SPLiT-seq libraries sequenced on Illumina with 150 nt kits
- Primer list file (TXT) of all oligonucleotide sequences used
- Barcode list file (TXT) of the 96 well plate oligonucleotides for each round of barcodes

## Outputs

- Output Read1 FASTQ composed of combinatorial barcodes and UMI, plus corresponding Read2 FASTQ of sequencing data
- Per-sample identified-read counts and identified-read ratios (e.g. saved as scsit_identified_reads_comparison.csv)

## How to apply

Download the raw paired-end FASTQ files for the target samples (e.g. SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264) from their SRA/BioSample accessions. Install SCSit via conda ('conda install -c susiew01 scsit'), pip ('pip install scsit'), or by compiling the C binary from GitHub. Run SCSit on each read pair with the primer list and 96-well barcode list, keeping the default tolerance of one mismatch and one indel, which the authors chose as the optimal efficiency balance. Count the labeled reads in the output Read1 FASTQ (which contains combinatorial barcodes and UMI) and compute the identified-read ratio relative to total input reads. Validate by comparing per-sample ratios against the reported values (73.22%, 74.89%, 75.84%, 66.98%, 66.26%), all of which exceed the original SPLiT-seq method, with every rate above 65%.

## Related tools

- **SCSit** (Primary C multithreaded tool that identifies three rounds of barcode and UMI from SPLiT-seq paired-end reads and outputs labeled reads, using one-mismatch and one-indel tolerance by default) — https://github.com/shang-qian/SCSit
- **Python 3.6** (Required runtime for installing and running SCSit via conda or pip)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- Identified-read ratios for the five SPLiT-seq samples should closely match reported SCSit values: 66.98% (SAMN08567263), 75.84% (SAMN08567261), 74.89% (SAMN08567260), 73.22% (SAMN08567259), 66.26% (SAMN08567264), all above 65%
- Identified-read counts should reach approximately 56,833,343 (SAMN08567263), 163,780,622 (SAMN08567261), 163,502,984 (SAMN08567260), 148,423,169 (SAMN08567259), and 160,257,239 (SAMN08567264), all exceeding the original method
- An output Read1 FASTQ containing combinatorial barcodes and UMI plus a corresponding Read2 FASTQ must be produced for every input sample pair
- Consistency of identified reads should increase to about 97% relative to the original alignment approach

## Limitations

- Default SCSit configuration is restricted to one mismatch and one indel; higher tolerance was evaluated but not adopted because identified reads only increase 4–5% while runtime increases 20–40%
- Fault-tolerant and indel alignment were proposed for SPLiT-seq data; use on other single-cell sequencing platforms is stated as potential, not demonstrated, and broader platform support is deferred to future updates
- SCSit was developed in C with multithreaded parallelism and optimized for x86-64 CPUs; runtime was assessed with 4 CPU cores, so behavior under other configurations is not characterized

## Evidence
