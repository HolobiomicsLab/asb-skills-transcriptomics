---
name: distance-function-indicator-sequence-validation
description: 'Use when validating per-read barcode calls from SPLiT-seq reads after
  sliding-window index location: each of the three 8-nt indicator barcodes must satisfy
  Dist < 2 against its whitelist entry, otherwise the Read2 must be abandoned.'
when_to_use_negative:
- The barcode platform is not SPLiT-seq-style combinatorial 8-nt barcodes — the article
  states the fault-tolerant/indel alignment was proposed for SPLiT-seq data, and broader
  application is potential, not demonstrated
- Reads still need index-region location — this validation step presumes the sliding-window
  scan has already identified the index positions and extracted the indicator sequences
- Higher tolerance (>1 mismatch or >1 indel) is required — SCSit's default uses one
  mismatch and one indel, which increases identified reads only 4–5% while costing
  20–40% more runtime
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: SCSit
  role: C program (multithreaded) that performs the quad-to-decimal encoding, Dist(x,y)
    threshold check (Dist < 2), and emits validated barcode/UMI calls; compiled from
    the GitHub release
  repo: https://github.com/shang-qian/SCSit
- name: Python
  role: Python 3.6 dependency required for running SCSit; used in the skill's reimplementation
    for encoding indicator sequences and computing Dist per read
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:e7b05526cd8d8826f0673cd7185783ed074f0cb076a7b60bce374858e91fe44b
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# distance-function-indicator-sequence-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates the three 8-nt indicator barcodes extracted from each SPLiT-seq Read2 by encoding them via quad-to-decimal conversion (0,1,2,3 = A,T,G,C) and computing the integer distance function Dist(x,y), accepting Read2 only when Dist < 2 for every round. This is the final correctness gate in SCSit's indel-tolerant barcode identification pipeline.

## When to use

Use when validating per-read barcode calls from SPLiT-seq reads after sliding-window index location: each of the three 8-nt indicator barcodes must satisfy Dist < 2 against its whitelist entry, otherwise the Read2 must be abandoned.

## When NOT to use

- The barcode platform is not SPLiT-seq-style combinatorial 8-nt barcodes — the article states the fault-tolerant/indel alignment was proposed for SPLiT-seq data, and broader application is potential, not demonstrated
- Reads still need index-region location — this validation step presumes the sliding-window scan has already identified the index positions and extracted the indicator sequences
- Higher tolerance (>1 mismatch or >1 indel) is required — SCSit's default uses one mismatch and one indel, which increases identified reads only 4–5% while costing 20–40% more runtime

## Inputs

- Read2 sequence with located 30-nt index regions (complete match, one mismatch, or one INDEL accepted)
- Three extracted 8-nt indicator barcode sequences per read
- barcodeList (Table S1) encoded as integer hash of quad-to-decimal converted 8-nt segments

## Outputs

- Per-read validation verdict: accept (Dist < 2 for all three indicator barcodes) or abandon Read2
- Combinatorial three-round barcode call and UMI in Read1 for accepted reads, emitted to labeled-read FASTQ

## How to apply

After locating the 30-nt index regions in Read2 with the 8-nt sliding window (including one-mismatch and one-INDEL tolerance), extract the three 8-nt indicator barcodes and the UMI. Encode each indicator sequence as an integer using the quad-to-decimal scheme where 0, 1, 2, and 3 represent A, T, G, and C (intSeq = sum(seq[i]*4^(n-i-1))), enabling O(1) lookup against the barcodeList-derived hash. Compute Dist(x,y) between each encoded indicator and its whitelist entry; if Dist < 2 for all three rounds, emit the combinatorial barcode/UMI call, otherwise abandon Read2. This integer-distance gate is what bounds SCSit to its default tolerance of one mismatch and one indel per barcode, balancing efficiency against read retention.

## Related tools

- **SCSit** (C program (multithreaded) that performs the quad-to-decimal encoding, Dist(x,y) threshold check (Dist < 2), and emits validated barcode/UMI calls; compiled from the GitHub release) — https://github.com/shang-qian/SCSit
- **Python** (Python 3.6 dependency required for running SCSit; used in the skill's reimplementation for encoding indicator sequences and computing Dist per read)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- For every accepted Read2, all three indicator barcodes satisfy Dist < 2 against their barcodeList whitelist entries; any read with a round at Dist >= 2 must be abandoned
- Per-read agreement with SCSit reference barcode/UMI calls reaches the paper's 97% consistency level (barcode_agreement_report.csv)
- No accepted call contains an indicator barcode differing by more than one substitution from a whitelist entry
- Abandoned Read2 fraction is consistent with the one-mismatch/one-INDEL default tolerance; relaxing tolerance should increase identified reads only ~4–5% per Table S4

## Limitations

- Default tolerance is restricted to one mismatch and one indel; higher tolerance increases identified reads only 4–5% at 20–40% more runtime (Table S4) and is not enabled by default
- Reads with indicator distance Dist >= 2 in any round are abandoned outright, so reads with two or more mismatches in one 8-nt barcode are lost
- Fault-tolerant and indel alignment was proposed for SPLiT-seq data; application to other barcode-based single-cell data is stated as potential, not demonstrated

## Evidence
