---
name: runtime-vs-sensitivity-tradeoff-analysis
description: 'Decision support for choosing SCSit tolerance settings: when you need
  to know whether enabling more than one mismatch and one indel in SPLiT-seq barcode
  identification gains enough reads to outweigh the extra runtime.'
when_to_use_negative:
- The task is routine SPLiT-seq preprocessing rather than parameter benchmarking —
  use SCSit's default one-mismatch/one-indel setting directly
- Input data is not SPLiT-seq combinatorial-barcoding data; SCSit's fault-tolerant/indel
  alignment was proposed specifically for SPLiT-seq, with other platforms only a stated
  future possibility
- Runtime cannot be controlled (variable CPU load or thread counts), since the trade-off
  conclusion depends on runtimes measured under equal 4-core conditions
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: SCSit
  role: Preprocessing tool run twice per sample (default vs. relaxed mismatch/indel
    tolerance) to measure identified reads and runtime
  repo: https://github.com/shang-qian/SCSit
- name: Python
  role: Used to compute percent changes and assemble the Table S4-style comparison
    CSV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:7ac574aef813908f0d694ee759b56663c097c15d82dfd11117b5c33103efa1ac
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
  - SCSit
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# runtime-vs-sensitivity-tradeoff-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies whether relaxing SCSit's default fault tolerance (one mismatch and one indel) to multiple mismatches/indels justifies its runtime cost, replicating the Table S4 evaluation on five SPLiT-seq samples.

## When to use

Decision support for choosing SCSit tolerance settings: when you need to know whether enabling more than one mismatch and one indel in SPLiT-seq barcode identification gains enough reads to outweigh the extra runtime.

## When NOT to use

- The task is routine SPLiT-seq preprocessing rather than parameter benchmarking — use SCSit's default one-mismatch/one-indel setting directly
- Input data is not SPLiT-seq combinatorial-barcoding data; SCSit's fault-tolerant/indel alignment was proposed specifically for SPLiT-seq, with other platforms only a stated future possibility
- Runtime cannot be controlled (variable CPU load or thread counts), since the trade-off conclusion depends on runtimes measured under equal 4-core conditions

## Inputs

- Paired-end Illumina FASTQ files (R1/R2) for the five SPLiT-seq samples (SRA accessions SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264)
- barcodeList TXT file (96-well plate oligonucleotides for each round of barcodes)
- primerList TXT file (forward and reverse primer oligonucleotide sequences)

## Outputs

- Per-sample identified-read counts and runtime (under 4 CPU cores) for the default and relaxed tolerance runs
- Percent-change metrics for identified reads and runtime per sample
- Comparison table (e.g. table_s4_relaxed_tolerance_comparison.csv) with columns for identified reads and runtime under both settings plus percent changes

## How to apply

Run SCSit on each of the five SPLiT-seq FASTQ sample pairs (SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264) with its default tolerance of one mismatch and one indel, fixing the thread count at 4 (-t 4) so runtimes are comparable. Then re-run each sample with relaxed settings permitting more than one mismatch and one indel during Read2 index/indicator-sequence identification. For each sample compute the percent increase in identified reads and in runtime of the relaxed run versus the default run, and assemble the per-sample metrics into a comparison table mirroring Table S4. The article's benchmark shows identified reads increase only ~4-5% while runtime grows ~20-40%, which is why one mismatch and one indel is kept as the default; treat deviations substantially outside these ranges as flags for re-examination.

## Related tools

- **SCSit** (Preprocessing tool run twice per sample (default vs. relaxed mismatch/indel tolerance) to measure identified reads and runtime) — https://github.com/shang-qian/SCSit
- **Python** (Used to compute percent changes and assemble the Table S4-style comparison CSV)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o default_SAMN08567259
```

## Evaluation signals

- Relaxed-tolerance runs identify only ~4-5% more reads than the default one-mismatch/one-indel runs
- Relaxed-tolerance runs require ~20-40% more runtime than default runs under the same 4-core conditions
- Both settings use identical inputs and thread count (-t 4), so only the tolerance parameter differs between runs
- Per-sample percent changes fall near the Table S4 ranges; samples deviating substantially are flagged
- Output table has columns for identified reads and runtime under both settings plus their percent changes, one row per sample

## Limitations

- The article does not expose a documented CLI flag for changing the mismatch/indel tolerance — the relaxed run may require recompiling or modifying the C source
- Quantitative results are deferred to supplementary Table S4, which is cited but not provided or linked in the text
- SCSit requires a working installation of Python 3.6 and is optimized for x86-64 CPUs

## Evidence
