---
name: mismatch-indel-tolerant-read2-index-alignment
description: Decision support for choosing SCSit's mismatch/indel tolerance parameters
  when preprocessing SPLiT-seq paired-end FASTQ data and questioning whether the default
  one-mismatch/one-indel setting sacrifices too many reads.
when_to_use_negative:
- Input data is not SPLiT-seq combinatorial-barcode data; SCSit's fault-tolerant and
  indel alignment was proposed for SPLiT-seq, and support for other platforms is only
  potential, not demonstrated
- You need a tool that supports other single-cell sequencing platforms out of the
  box — SCSit states it 'will be updated and improved in time' for more platforms,
  so this is not yet available
- You only need the default preprocessing result and have no question about the tolerance
  trade-off; the paper already concludes the default one-mismatch/one-indel setting
  is optimal
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: SCSit
  role: Preprocessing tool run under default and relaxed mismatch/indel tolerance
    settings on the same paired-end FASTQ inputs to measure the read-retention vs.
    runtime trade-off
  repo: https://github.com/shang-qian/SCSit
- name: Python 3.6
  role: Required working installation for SCSit; used to assemble the per-sample percent-change
    comparison table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:708bf755b2723a72d53cf3dbd2abacde6a4209614a28bb03a4e0b7ae6830683b
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# mismatch-indel-tolerant-read2-index-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for SCSit that evaluates whether relaxing the default Read2 index/indicator-sequence identification tolerance (one mismatch and one indel) to more than one mismatch and one indel is worthwhile. The paper found this yields only ~4–5% more identified reads at 20–40% more runtime (Table S4), so the default setting is retained.

## When to use

Decision support for choosing SCSit's mismatch/indel tolerance parameters when preprocessing SPLiT-seq paired-end FASTQ data and questioning whether the default one-mismatch/one-indel setting sacrifices too many reads.

## When NOT to use

- Input data is not SPLiT-seq combinatorial-barcode data; SCSit's fault-tolerant and indel alignment was proposed for SPLiT-seq, and support for other platforms is only potential, not demonstrated
- You need a tool that supports other single-cell sequencing platforms out of the box — SCSit states it 'will be updated and improved in time' for more platforms, so this is not yet available
- You only need the default preprocessing result and have no question about the tolerance trade-off; the paper already concludes the default one-mismatch/one-indel setting is optimal

## Inputs

- Paired-end Illumina FASTQ files (R1/R2) for SPLiT-seq samples SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264
- barcodeList TXT file (96-well plate oligonucleotides for each round of barcodes)
- primerList TXT file (forward and reverse primer sequences, Table S2)

## Outputs

- Per-sample identified-read counts and runtime under default (1 mismatch/1 indel) and relaxed tolerance settings
- Percent-change metrics for identified reads and runtime (relaxed vs. default)
- Comparison table mirroring Table S4 (e.g., table_s4_relaxed_tolerance_comparison.csv)

## How to apply

Obtain the five SPLiT-seq paired-end Illumina FASTQ samples (SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264) with their barcodeList and primerList TXT files. Run SCSit under the default tolerance of one mismatch and one indel (the setting chosen for 'the optimal balance of efficiency'), recording identified-read counts and runtime under 4 CPU cores. Re-run each sample with relaxed settings permitting more than one mismatch and one indel during Read2 index/indicator-sequence identification under identical 4-core conditions. For each sample, compute the percent increase in identified reads and runtime relative to the default run, assemble them into a comparison table mirroring Table S4, and flag any sample whose deviation from the reported ~4–5% read gain / ~20–40% runtime increase ranges is substantial.

## Related tools

- **SCSit** (Preprocessing tool run under default and relaxed mismatch/indel tolerance settings on the same paired-end FASTQ inputs to measure the read-retention vs. runtime trade-off) — https://github.com/shang-qian/SCSit
- **Python 3.6** (Required working installation for SCSit; used to assemble the per-sample percent-change comparison table)

## Examples

```
scsit -r1 <R1.fastq> -r2 <R2.fastq> -p primer.list -b barcode.list -t 4 -o default_<sample>
```

## Evaluation signals

- Relaxing tolerance beyond one mismatch and one indel increases identified reads by only ~4–5% relative to the default run, matching the paper's Table S4 result
- Relaxed runs take ~20–40% more runtime than the default runs under the same 4 CPU cores
- Per-sample percent changes cluster within the reported ranges; samples deviating substantially are flagged for review
- All runs use the same inputs (paired-end FASTQ, primerList, barcodeList) and the same thread count (-t 4) so runtime comparisons are internally consistent
- Identified-read counts from default runs should be consistent with the paper's reported ~97% consistency of identified reads

## Limitations

- Quantitative per-sample results are in supplementary Table S4, cited without being provided or linked in the main text, so exact per-sample values may need to be regenerated
- The default is deliberately restricted to one mismatch and one indel because larger tolerance yields marginal gains (4–5% reads) at high cost (20–40% runtime)
- Runtime comparisons depend on the 4-CPU-core measurement conditions used in the paper's assessment; other hardware may shift the runtime percentages
- SCSit requires Python 3.6 and is optimized for x86-64 CPUs; it operates on a pair of paired-end FASTQ files at a time

## Evidence
