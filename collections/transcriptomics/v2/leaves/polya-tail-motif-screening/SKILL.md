---
name: polya-tail-motif-screening
description: Use when a strand-split BAM of aligned reads must be interrogated for
  reads with soft-clipped polyA/polyT tails to build truncation-point tallies that
  localize precise 3' UTR end bases.
when_to_use_negative:
- The BAM contains long-read or full-length transcript alignments without soft-clipping
  — polyA tails will be embedded in aligned sequence rather than clipped, so the truncation-point
  signal will be absent or misleading
- You only need broad regions of UTR enrichment — MACS peak calling on the BAM is
  the upstream step for that, and this skill addresses the finer end-base localization
  stage
- The library chemistry does not enrich signal at transcript distal ends (not a 10x
  Chromium-like run), so soft-clipped polyA reads are too sparse to form piled-up
  truncation points
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: peaks2utr
  role: 'Hosts the SPAT stage whose logic this skill re-implements: detection of soft-clipped,
    polyA-tailed reads and tallying of their end bases as truncation points'
  repo: https://github.com/haessar/peaks2utr
- name: Python 3
  role: Language for iterating over BAM alignments, parsing CIGAR strings, and tallying
    truncation points (peaks2utr requires Python ≥3.8)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:f0777e6eb19ecc7142bab33b5e3b79440a343a851b4aeb437856ae36c10fd9c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - Python 3
  - peaks2utr
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# polya-tail-motif-screening

## Summary

Screens aligned sequencing reads in a BAM file for soft-clipped bases carrying a polyA-tail (or reverse-strand polyT stretch) of a configured minimum length, tallying each qualifying read's final aligned end base as a 'truncation point' for downstream 3' UTR end determination, as implemented in the SPAT stage of peaks2utr.

## When to use

Use when a strand-split BAM of aligned reads must be interrogated for reads with soft-clipped polyA/polyT tails to build truncation-point tallies that localize precise 3' UTR end bases.

## When NOT to use

- The BAM contains long-read or full-length transcript alignments without soft-clipping — polyA tails will be embedded in aligned sequence rather than clipped, so the truncation-point signal will be absent or misleading
- You only need broad regions of UTR enrichment — MACS peak calling on the BAM is the upstream step for that, and this skill addresses the finer end-base localization stage
- The library chemistry does not enrich signal at transcript distal ends (not a 10x Chromium-like run), so soft-clipped polyA reads are too sparse to form piled-up truncation points

## Inputs

- Strand-split BAM file of aligned sequencing reads (e.g. the repository demo file Tb927_01_v5.1.slice.bam)
- Configured minimum polyA-tail length threshold
- Optional canonical annotation in GFF/GTF format for context of the UTR being terminated

## Outputs

- Per-read truncation-point records: genomic coordinate of the final aligned end base of each read with soft-clipped bases and a qualifying polyA-tail
- Tally of co-occurring end-base coordinates per chromosome and strand (e.g. truncation_points.tsv, one row per end-base position with its read count)
- Piled-up truncation points used downstream to determine the precise end base of a given UTR

## How to apply

Iterate over alignments per reference sequence and strand in Python 3, mirroring the SPAT stage of the peaks2utr package (demo input: Tb927_01_v5.1.slice.bam). For each read, parse the CIGAR string to flag soft-clipped base operations, then screen the terminal unaligned sequence for a poly(A) tail of the configured minimum length — the reverse-complement poly(T) stretch on the reverse strand. Record the genomic coordinate of the final aligned end base of every read that contains both soft-clipped bases and a qualifying polyA-tail. Tally co-occurring end-base coordinates per chromosome and strand; when piled up, each co-occurring truncation point determines the precise end base of a given UTR. The rationale is that library preparation such as 10x Chromium concentrates signal inherently at the distal ends of 3' or 5' UTRs, so soft-clipped polyA reads mark true transcript termini.

## Related tools

- **peaks2utr** (Hosts the SPAT stage whose logic this skill re-implements: detection of soft-clipped, polyA-tailed reads and tallying of their end bases as truncation points) — https://github.com/haessar/peaks2utr
- **Python 3** (Language for iterating over BAM alignments, parsing CIGAR strings, and tallying truncation points (peaks2utr requires Python ≥3.8))

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Each read containing soft-clipped bases and a polyA-tail of the given length contributes exactly one end base to the tally — count of records equals count of qualifying reads
- Reverse-strand reads contribute the reverse-complement poly(T) stretch as their tail signal, and their end coordinates are tallied separately per strand
- Every tallied coordinate corresponds to a final aligned end base in the BAM (verifiable against the CIGAR soft-clip length and reference start)
- Piled-up co-occurring truncation points exist at positions where multiple independent reads agree, which downstream determines the precise UTR end base
- Running the full peaks2utr pipeline on the demo inputs (peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam) produces a GFF3 with 3' UTR features sourced as 'peaks2utr', consistent with the tallied truncation points

## Limitations

- Only reads with soft-clipped bases AND a polyA-tail of the configured minimum length contribute; reads with aligned polyA or tails shorter than the threshold are missed
- The method favours BAM files from runs like 10x Chromium where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs; other library types may yield weak signal
- A single truncation point is not sufficient — the precise UTR end base requires piled-up co-occurring truncation points
- No software version numbers are stated in the paper's methods, and installation assumes a Debian/Ubuntu system with root privileges

## Evidence

- [intro] Soft-clipped polyA detection and truncation-point tallying: "Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points"."
- [intro] Piled-up truncation points determine the precise UTR end base: "When piled up, each co-occurring truncation point is used to determine the precise end base of a given UTR."
- [intro] Library type rationale for concentrated distal signal: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [readme] Demo BAM input for verifying the SPAT stage: "Tb927_01_v5.1.slice.bam"
- [abstract] peaks2utr implemented in Python 3: "peaks2utr is implemented in Python 3 (≥3.8)"
