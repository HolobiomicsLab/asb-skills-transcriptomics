---
name: modification-classification-precision-recall-assessment
description: Use when a modification-aware basecaller has been run on reads with ground-truth
  modified positions (e.g., synthetic IVT constructs) and you must quantify per-class
  precision/recall and cross-modification misclassification.
when_to_use_negative:
- Reads lack ground-truth modification positions (e.g., native biological samples),
  so per-class precision/recall cannot be computed and only qualitative comparison
  is possible
- The basecaller output is a genome-space aggregate only, with no per-read call coordinates,
  preventing per-read identity benchmarking against Guppy
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: NanoSpeech
  role: Modification-aware transformer basecaller whose calls (canonical, I, m6A)
    are evaluated for per-class precision/recall
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: minimap2
  role: Maps basecalled fasta/fastq reads to the reference with -ax map-ont -k 14
    --secondary=no --MD presets to localize modification calls
- name: samtools
  role: Filters (view -b -F 2304), sorts, and indexes the SAM into a BAM consumed
    by modification_detector.py
- name: Guppy
  role: Standard modification-unaware basecaller used as the per-read identity baseline
    for comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:1eeabadb700f7912a65957765c1698dcf98bcbf04d7f03f18b8114cb45c13c91
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/F0nz0/NanoSpeech_basecaller
  repo_url: https://github.com/F0nz0/NanoSpeech_basecaller
  tools:
  - Guppy
  - NanoSpeech
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# modification-classification-precision-recall-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a modification-aware basecaller (e.g., NanoSpeech) by computing per-class precision and recall for each modification class (canonical, inosine, m6A) against known modified positions on synthetic dRNA reads, checking for cross-class confusion and benchmarking per-read identity against a standard basecaller baseline.

## When to use

Use when a modification-aware basecaller has been run on reads with ground-truth modified positions (e.g., synthetic IVT constructs) and you must quantify per-class precision/recall and cross-modification misclassification.

## When NOT to use

- Reads lack ground-truth modification positions (e.g., native biological samples), so per-class precision/recall cannot be computed and only qualitative comparison is possible
- The basecaller output is a genome-space aggregate only, with no per-read call coordinates, preventing per-read identity benchmarking against Guppy

## Inputs

- Raw dRNA pod5/fast5 reads from synthetic IVT constructs with known modified positions (inosine, m6A)
- Basecalled fasta/fastq from a modification-aware basecaller (e.g., NanoSpeech with expanded A,C,G,U,I,m6A vocabulary)
- Reference transcriptome or genome fasta for mapping
- Sorted, indexed BAM of alignments
- {bam}.per_read.bed and {bam}.genome_space.tsv modification call files from modification_detector.py

## Outputs

- Per-class precision values for canonical, inosine, and m6A sites
- Per-class recall values for canonical, inosine, and m6A sites
- I-to-m6A and m6A-to-I cross-misclassification counts
- Per-read identity/consensus accuracy compared to a Guppy baseline

## How to apply

After mapping basecalled reads (minimap2 -ax map-ont -k 14 --secondary=no --MD for a transcriptome, or -ax splice -uf -k 14 --secondary=no --MD for a genome) and converting to a sorted, indexed BAM (samtools view -b -F 2304 | samtools sort -O BAM; samtools index), run modification_detector.py with a minimum base quality threshold to obtain per-read BED and genome-space TSV coordinates for every called modified nucleotide. Match these calls against the known modified positions of the synthetic constructs to compute precision and recall separately for each class (canonical, I, m6A), and specifically count I-to-m6A and m6A-to-I cross-misclassifications, which should be zero. Additionally compute per-read identity/consensus accuracy and compare it to a modification-unaware baseline (Guppy) on the same modified sequences. On the m6A-I-aware NanoSpeech model, validation was passing when per-class precision reached at least 0.98, recall was approximately 0.99, and no I and m6A sites were confused with each other, with per-read identity exceeding the Guppy baseline.

## Related tools

- **NanoSpeech** (Modification-aware transformer basecaller whose calls (canonical, I, m6A) are evaluated for per-class precision/recall) — https://github.com/F0nz0/NanoSpeech_basecaller
- **minimap2** (Maps basecalled fasta/fastq reads to the reference with -ax map-ont -k 14 --secondary=no --MD presets to localize modification calls)
- **samtools** (Filters (view -b -F 2304), sorts, and indexes the SAM into a BAM consumed by modification_detector.py)
- **Guppy** (Standard modification-unaware basecaller used as the per-read identity baseline for comparison)

## Examples

```
samtools view -b -F 2304 calls.sam | samtools sort -O BAM calls.sorted.bam && samtools index calls.sorted.bam && python3 modification_detector.py -b calls.sorted.bam -f basecalls.fastq -q 7
```

## Evaluation signals

- Per-class precision for canonical, inosine, and m6A each reaches at least 0.98
- Per-class recall for inosine and m6A is approximately 0.99
- Zero I-to-m6A and m6A-to-I cross-misclassification on unseen synthetic reads
- False positive rate remains low across called modification sites
- Per-read identity/consensus accuracy exceeds the Guppy baseline on the modified sequences

## Limitations

- Performance was primarily demonstrated on synthetic IVT constructs as a proof of concept; native-sample transfer may be impaired by questionable quality of public RNA-modification data and lack of reliable benchmarks
- Training-set quality issues (read number, quality, contamination, unknown IVT conditions) can lead to inference artefacts or 'hallucinations' that affect measured precision/recall
- Streamlined models for detection of highly similar chemical structures are still under development, so adjacent-modification confusion remains a risk

## Evidence

- [methods] per-class precision and recall of I and m6A calls against the known modified positions of the synthetic constructs: [evidence span withheld — anchor status: not_found]
- [results] precision of at least 0.98 for each class of sites: [evidence span withheld — anchor status: not_found]
- [methods] Run modification_detector.py: [evidence span withheld — anchor status: not_found]
- [methods] validation threshold statement: [evidence span withheld — anchor status: not_found]
