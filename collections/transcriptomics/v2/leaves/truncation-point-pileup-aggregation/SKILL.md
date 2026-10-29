---
name: truncation-point-pileup-aggregation
description: Use when you have a strand-split BAM of aligned reads (e.g. from 10x
  Chromium) and need per-position counts of polyA/soft-clip-driven read endpoints
  to locate precise UTR termination sites.
when_to_use_negative:
- Input reads lack soft-clipped bases or polyA/polyT tails (e.g. fully aligned reads
  without 3' enrichment) — no truncation points will be detectable
- BAM files whose signal is not concentrated at UTR distal ends (the method favours
  10x Chromium-style runs)
- You only need broad coverage peaks rather than precise UTR end bases — use MACS
  peak calling instead
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: peaks2utr
  role: Source package whose SPAT stage performs soft-clip/polyA detection and truncation-point
    tallying
  repo: https://github.com/haessar/peaks2utr
- name: Python 3
  role: Implementation language (>=3.8) for iterating BAM alignments and parsing CIGAR
    strings
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:9b7d2a94ac8a5e73e00e93551b8be054bc4a6f475b990753846ed4180b8e3bdc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - Python 3
  - peaks2utr
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# truncation-point-pileup-aggregation

## Summary

Reconstructs the SPAT stage of peaks2utr: detect aligned reads carrying soft-clipped bases and a polyA-tail of a given length, then tally each read's final aligned end base as a 'truncation point' so piled-up co-occurring truncation points can pinpoint the precise 3' UTR end.

## When to use

Use when you have a strand-split BAM of aligned reads (e.g. from 10x Chromium) and need per-position counts of polyA/soft-clip-driven read endpoints to locate precise UTR termination sites.

## When NOT to use

- Input reads lack soft-clipped bases or polyA/polyT tails (e.g. fully aligned reads without 3' enrichment) — no truncation points will be detectable
- BAM files whose signal is not concentrated at UTR distal ends (the method favours 10x Chromium-style runs)
- You only need broad coverage peaks rather than precise UTR end bases — use MACS peak calling instead

## Inputs

- Strand-split BAM file of aligned sequencing reads (e.g. Tb927_01_v5.1.slice.bam from the peaks2utr demo)
- Configured minimum polyA-tail length (poly(T) minimum length for reverse-strand reads)

## Outputs

- truncation_points.tsv — one row per end-base genomic position with its read count, tallied per chromosome and strand

## How to apply

Iterate over alignments in the BAM per reference sequence and strand (Python 3, mirroring the peaks2utr SPAT stage; the repo demo file Tb927_01_v5.1.slice.bam is a suitable test input). For each read, parse the CIGAR string to flag soft-clipped base operations and screen the terminal unaligned sequence for a poly(A) tail of the configured minimum length — a reverse-complement poly(T) stretch on the reverse strand. Only reads that contain BOTH soft-clipped bases and a qualifying polyA-tail contribute, and each contributes exactly one genomic coordinate: the final aligned end base. Tally co-occurring end-base coordinates per chromosome and strand and write counts to truncation_points.tsv, one row per end-base position with its read count; these piled-up co-occurring truncation points are what downstream stages use to determine the precise end base of a given UTR.

## Related tools

- **peaks2utr** (Source package whose SPAT stage performs soft-clip/polyA detection and truncation-point tallying) — https://github.com/haessar/peaks2utr
- **Python 3** (Implementation language (>=3.8) for iterating BAM alignments and parsing CIGAR strings)

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Each read containing soft-clipped bases and a polyA-tail of the given length contributes exactly one end base to the tally
- Reverse-strand reads must be screened for a reverse-complement poly(T) stretch of at least the configured minimum length
- Output truncation_points.tsv has one row per end-base coordinate with an integer read count, stratified per chromosome and strand
- Piled-up co-occurring truncation points should correspond to candidate precise UTR end bases consumed by the downstream annotation step
- Running the reconstructed stage on the repo demo Tb927_01_v5.1.slice.bam yields a consistent tally with the SPAT stage of the peaks2utr package

## Limitations

- Only reads with both soft-clipped bases and a polyA-tail of the given minimum length are tallied; shorter or absent tails yield no signal
- Signal quality depends on library type — the approach favours BAM files from runs like 10x Chromium where signal is inherently concentrated at UTR distal ends
- Tallied truncation points alone do not determine the UTR end; they must be piled up and interpreted downstream

## Evidence

- [intro] SPAT mechanism: "Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points"."
- [intro] pileup determines UTR end: "When piled up, each co-occurring truncation point is used to determine the precise end base of a given UTR."
- [intro] library-type dependency: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [readme] demo BAM input: "Tb927_01_v5.1.slice.bam"
