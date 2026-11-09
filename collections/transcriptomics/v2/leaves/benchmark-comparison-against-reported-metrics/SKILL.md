---
name: benchmark-comparison-against-reported-metrics
description: Use when a paper reports concrete per-sample quantitative results (e.g.,
  identified-read counts/ratios) on named public datasets (e.g., SAMN accessions)
  and you must independently reproduce and compare them.
when_to_use_negative:
- Input is not SPLiT-seq combinatorial-barcoded data — the article proposes the fault-tolerant
  and indel alignment only for SPLiT-seq data, with other platforms stated as potential
  but not demonstrated
- No reported quantitative metrics or public raw datasets exist to compare against,
  so there is nothing to benchmark
- The goal is biological downstream analysis (e.g., clustering or cell typing) rather
  than tool reproduction/validation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: SCSit
  role: Preprocessing tool re-run on the five SPLiT-seq datasets with default one-mismatch/one-indel
    tolerance to reproduce identified-read counts and ratios
  repo: https://github.com/shang-qian/SCSit
- name: Python 3.6
  role: Required working environment for installing and running SCSit (pip/conda install)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:8c8cc3a6e3c0bb57212b0e00bcfa76f20624111eb1b045fa93d08705859748a9
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# benchmark-comparison-against-reported-metrics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Re-run a published tool (here SCSit, a SPLiT-seq preprocessing tool) on its original public datasets and assemble per-sample metrics into a table alongside the paper's reported values to verify reproduction within a small deviation tolerance.

## When to use

Use when a paper reports concrete per-sample quantitative results (e.g., identified-read counts/ratios) on named public datasets (e.g., SAMN accessions) and you must independently reproduce and compare them.

## When NOT to use

- Input is not SPLiT-seq combinatorial-barcoded data — the article proposes the fault-tolerant and indel alignment only for SPLiT-seq data, with other platforms stated as potential but not demonstrated
- No reported quantitative metrics or public raw datasets exist to compare against, so there is nothing to benchmark
- The goal is biological downstream analysis (e.g., clustering or cell typing) rather than tool reproduction/validation

## Inputs

- Raw paired-end FASTQ files for five SPLiT-seq samples (SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264) from SRA/BioSample
- SCSit primer.list (oligonucleotide primer sequences, Table S2)
- SCSit barcode.list (96-well plate oligonucleotides for each round of barcodes)

## Outputs

- Per-sample identified-read counts and identified-read ratios
- Comparison CSV (scsit_identified_reads_comparison.csv) with reproduced vs. reported values (e.g., 73.22%, 74.89%, 75.84%, 66.98%, 66.26%)
- Output FASTQ pair (labeled Read1 with combinatorial barcodes and UMI, plus corresponding Read2) for each sample

## How to apply

Download the raw paired-end Illumina FASTQ files for each named BioSample accession (here SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264) from SRA. Install the tool exactly as documented (conda via 'conda install -c susiew01 scsit', pip, or compile the C binary with ./make.sh) and run it on each read pair with the paper's default tolerance settings — for SCSit, one mismatch and one indel — using the primer.list and barcode.list supplied in the repository. For each sample, count labeled reads in the output Read1 FASTQ (which contains combinatorial barcodes and UMI) and compute the identified-read ratio relative to total input reads. Assemble per-sample counts and ratios into a comparison table (e.g., scsit_identified_reads_comparison.csv) alongside the reported values and check each reproduces within a small deviation tolerance.

## Related tools

- **SCSit** (Preprocessing tool re-run on the five SPLiT-seq datasets with default one-mismatch/one-indel tolerance to reproduce identified-read counts and ratios) — https://github.com/shang-qian/SCSit
- **Python 3.6** (Required working environment for installing and running SCSit (pip/conda install))

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- Reproduced identified-read ratios closely match the reported SCSit ratios for all five samples: 66.98% (SAMN08567259), 75.84% (SAMN08567260), 74.89% (SAMN08567261), 73.22% (SAMN08567263), 66.26% (SAMN08567264) within a small deviation tolerance
- Reproduced identified-read counts match reported values: 148,423,169 (SAMN08567259), 163,502,984 (SAMN08567260), 163,780,622 (SAMN08567261), 56,833,343 (SAMN08567263), 160,257,239 (SAMN08567264)
- Every sample's identified-read rate exceeds 65% and exceeds the original SPLiT-seq method
- An output FASTQ pair is produced for every sample, with output Read1 composed of combinatorial barcodes and UMI
- Comparison table (scsit_identified_reads_comparison.csv) contains per-sample counts and ratios for all five accessions

## Limitations

- SCSit's default tolerance is restricted to one mismatch and one indel; higher tolerance was evaluated but yields only 4–5% more identified reads at 20–40% more runtime (supplementary Table S4)
- Quantitative comparison details (e.g., UMI/barcode-unidentified reads, Table S3) are deferred to supplementary tables cited but not provided in the text
- Reproduction depends on exact barcode.list and primer.list (Table S2) being available from the repository; output files can take minutes to open given their size

## Evidence

- [results] Identified-read counts for all five samples: "SAMN08567261 (163,780,622), SAMN08567260 (163,502,984)"
- [discussion] Default tolerance setting used for reproduction: "Considering the optimal balance of efficiency, we use one mismatch and one indel as default setting in SCSit."
- [readme] SCSit invocation and inputs: "scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output"
- [readme] Output Read1 content: "The output Read1 FASTQ format file was composed of combinatorial barcodes and UMI, and Read2 file was corresponding sequencing data."
- [abstract] Runtime and mapped-read improvements used as context for benchmarking: "the consistency of identified reads from SCSit increases to 97%, and mapped reads are twice than the original"
