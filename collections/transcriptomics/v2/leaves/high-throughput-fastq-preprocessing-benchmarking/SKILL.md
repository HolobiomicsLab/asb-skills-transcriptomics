---
name: high-throughput-fastq-preprocessing-benchmarking
description: Use when you must quantify the read-recovery vs. runtime trade-off of
  a preprocessing tool on raw paired-end SPLiT-seq FASTQ files, or justify/validate
  a default tolerance setting against a relaxed alternative.
when_to_use_negative:
- Input is not SPLiT-seq combinatorial-barcoded data with Read2 carrying index/indicator
  sequences — SCSit's fault-tolerant and indel alignment was proposed specifically
  for SPLiT-seq data
- The goal is read identification itself rather than measuring tool performance —
  use SCSit normally with its default one-mismatch/one-indel setting instead of benchmarking
- FASTQ files are already preprocessed/labeled; benchmarking requires raw paired-end
  input FASTQ
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: SCSit
  role: C-multithreaded preprocessing tool whose identified-read counts and runtime
    are benchmarked under default vs. relaxed mismatch/indel tolerance
  repo: https://github.com/shang-qian/SCSit
- name: Python 3.6
  role: Required runtime for SCSit installation; also usable to compute percent changes
    and assemble the comparison table
- name: BLAST
  role: Slower baseline alignment method against which SCSit's preprocessing efficiency
    is contextualized
- name: BWA
  role: Baseline aligner; unsuitable for the 8 bp barcodes of SPLiT-seq, motivating
    SCSit and the benchmark comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:a9b029781574c842e5977ac352c80ed211796bb5ec8abc3e4c258b9ff2f49743
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# high-throughput-fastq-preprocessing-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark SPLiT-seq FASTQ preprocessing (SCSit) by running paired-end Illumina reads through the tool under controlled CPU conditions and comparing identified-read counts and runtime across parameter settings (e.g., default one-mismatch/one-indel tolerance vs. relaxed tolerance).

## When to use

Use when you must quantify the read-recovery vs. runtime trade-off of a preprocessing tool on raw paired-end SPLiT-seq FASTQ files, or justify/validate a default tolerance setting against a relaxed alternative.

## When NOT to use

- Input is not SPLiT-seq combinatorial-barcoded data with Read2 carrying index/indicator sequences — SCSit's fault-tolerant and indel alignment was proposed specifically for SPLiT-seq data
- The goal is read identification itself rather than measuring tool performance — use SCSit normally with its default one-mismatch/one-indel setting instead of benchmarking
- FASTQ files are already preprocessed/labeled; benchmarking requires raw paired-end input FASTQ

## Inputs

- Paired-end Illumina SPLiT-seq FASTQ files (R1 and R2, e.g., SRA samples SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264)
- barcodeList TXT file (96-well plate oligonucleotides used for each round of barcodes)
- primerList TXT file (forward and reverse primer oligonucleotide sequences)
- Machine with ≥4 CPU cores for consistent runtime measurement

## Outputs

- Per-sample identified-read counts and runtimes under default and relaxed tolerance settings
- Per-sample percent increase in identified reads and runtime (relaxed vs. default)
- Comparison CSV (e.g., table_s4_relaxed_tolerance_comparison.csv) with columns for identified reads and runtime under both settings plus percent changes

## How to apply

Run SCSit on each paired-end FASTQ sample with its default tolerance of one mismatch and one indel (scsit -r1 <R1.fastq> -r2 <R2.fastq> -p primer.list -b barcode.list -t 4 -o <prefix>), recording identified-read counts and wall-clock time under a fixed 4-CPU-core budget. Re-run each sample with tolerance relaxed beyond one mismatch and one indel during Read2 index/indicator-sequence identification, keeping cores constant so runtime differences are attributable to the tolerance setting. Compute per-sample percent increases in identified reads and runtime of the relaxed run relative to the default, and assemble them into a comparison table (Table S4-style). The rationale is the paper's own efficiency-efficiency trade-off argument: relaxation must yield read gains large enough to justify added compute, otherwise the default stands.

## Related tools

- **SCSit** (C-multithreaded preprocessing tool whose identified-read counts and runtime are benchmarked under default vs. relaxed mismatch/indel tolerance) — https://github.com/shang-qian/SCSit
- **Python 3.6** (Required runtime for SCSit installation; also usable to compute percent changes and assemble the comparison table)
- **BLAST** (Slower baseline alignment method against which SCSit's preprocessing efficiency is contextualized)
- **BWA** (Baseline aligner; unsuitable for the 8 bp barcodes of SPLiT-seq, motivating SCSit and the benchmark comparison)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o default_SAMN08567259
```

## Evaluation signals

- Relaxed-tolerance runs should identify only ~4-5% more reads than the default one-mismatch/one-indel setting; flag any sample deviating substantially
- Relaxed-tolerance runs should take ~20-40% more runtime than default runs under the same 4 CPU cores
- Both runs per sample must use identical thread counts (-t 4) so runtime deltas reflect tolerance, not parallelism
- Comparison table must contain, per sample, identified reads and runtime under both settings plus their percent changes, mirroring Table S4 structure
- Output must be valid paired FASTQ with Read1 carrying combinatorial barcodes and UMI and Read2 carrying corresponding sequence data

## Limitations

- Tolerance beyond one mismatch and one indel was evaluated but not adopted: gains are only 4-5% more identified reads at 20-40% more runtime (Table S4)
- Runtime figures depend on the 4-core CPU environment; numbers are not portable across hardware
- The original SPLiT-seq method discarded reads whose barcode had more than one mismatch or low-quality UMI bases (phred ≤ 10), so benchmark comparisons against it reflect different filter chains, not only speed
- Quantitative comparisons are deferred to supplementary Tables S3/S4, which are cited but not provided or linked in the main text

## Evidence
