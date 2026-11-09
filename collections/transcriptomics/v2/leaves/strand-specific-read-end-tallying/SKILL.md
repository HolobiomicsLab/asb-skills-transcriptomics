---
name: strand-specific-read-end-tallying
description: Use when a BAM of aligned reads (e.g. 10x Chromium) contains soft-clipped
  bases with polyA-tails, and precise strand-specific UTR end-base coordinates are
  needed for downstream 3' UTR annotation.
when_to_use_negative:
- The BAM contains no soft-clipped reads or terminal poly(A)/poly(T) tails (e.g. non-3'-enriched
  data such as random-primed RNA-seq), since no truncation points can be tallied.
- The goal is peak calling or UTR annotation without read-level end evidence — MACS
  peak calling or the downstream peaks2utr annotation stage, not read-end tallying,
  is the appropriate step.
- Input is already a GFF/GTF annotation of UTRs; this skill consumes aligned reads,
  not annotation files.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0080
tools:
- name: peaks2utr
  role: Hosts the SPAT stage that detects soft-clipped reads with polyA-tails and
    tallies their end bases as truncation points
  repo: https://github.com/haessar/peaks2utr
- name: Python 3
  role: Implementation language (>=3.8) used to iterate over BAM alignments and parse
    CIGAR strings per strand
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:879b7301f34f053951867ec3d3f16c0631237c6310c0f81f4268d824f2f84dbb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - Python 3
  - peaks2utr
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Strand-specific read-end tallying (SPAT truncation points)

## Summary

Tallies the genomic coordinate of the final aligned end base of every read containing soft-clipped bases and a poly(A)-tail of a given minimum length, per chromosome and strand, to produce truncation-point counts used to pinpoint the precise 3' UTR end base.

## When to use

Use when a BAM of aligned reads (e.g. 10x Chromium) contains soft-clipped bases with polyA-tails, and precise strand-specific UTR end-base coordinates are needed for downstream 3' UTR annotation.

## When NOT to use

- The BAM contains no soft-clipped reads or terminal poly(A)/poly(T) tails (e.g. non-3'-enriched data such as random-primed RNA-seq), since no truncation points can be tallied.
- The goal is peak calling or UTR annotation without read-level end evidence — MACS peak calling or the downstream peaks2utr annotation stage, not read-end tallying, is the appropriate step.
- Input is already a GFF/GTF annotation of UTRs; this skill consumes aligned reads, not annotation files.

## Inputs

- Strand-split BAM file of aligned sequencing reads (e.g. Tb927_01_v5.1.slice.bam from the peaks2utr demo set)
- Configured minimum poly(A)-tail length (a reverse-complement poly(T) stretch on the reverse strand)

## Outputs

- truncation_points.tsv — one row per end-base coordinate with its read count, tallied per chromosome and strand
- Piled-up co-occurring truncation points used downstream to determine the precise UTR end base

## How to apply

Load a strand-split BAM (the repository demo file Tb927_01_v5.1.slice.bam) and iterate over alignments per reference sequence and strand. For each read, parse the CIGAR string to flag soft-clipped base operations, then screen the terminal unaligned sequence for a poly(A)-tail of the configured minimum length (a reverse-complement poly(T) stretch on the reverse strand). Record the genomic coordinate of the final aligned end base of every qualifying read. Tally co-occurring end-base coordinates per chromosome and strand; these piled-up truncation points are then used by the downstream stage to determine the precise end base of a given UTR.

## Related tools

- **peaks2utr** (Hosts the SPAT stage that detects soft-clipped reads with polyA-tails and tallies their end bases as truncation points) — https://github.com/haessar/peaks2utr
- **Python 3** (Implementation language (>=3.8) used to iterate over BAM alignments and parse CIGAR strings per strand)

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Each read containing soft-clipped bases and a poly(A)-tail of the given length contributes exactly one end base to the tally (one-to-one read-to-truncation-point invariant).
- Tallies are split per chromosome and strand; reverse-strand truncation points must come from reads with a reverse-complement poly(T) terminal stretch.
- Every recorded coordinate corresponds to the final aligned end base (not the soft-clipped sequence) of a qualifying read.
- Output truncation_points.tsv has exactly one row per end-base position with its read count, and piled-up co-occurring coordinates agree with the UTR end bases annotated downstream.
- Reproducing the SPAT stage on the demo file Tb927_01_v5.1.slice.bam yields truncation points consistent with the 3' UTRs in the demo output Tb927_01_v5.1.new.gff.

## Limitations

- Signal depends on reads with soft-clipped bases and a poly(A)-tail of the configured minimum length; shorter tails or fully aligned reads yield no truncation points.
- The method favours BAM files from runs like 10x Chromium, where signal is inherently concentrated at the distal ends of the 3' UTRs; other library types may not concentrate truncation points.
- Truncation-point tallies alone do not define UTR ends — a downstream step must pile up co-occurring truncation points to determine the precise end base of a given UTR.

## Evidence

- [intro] Truncation-point tallying mechanism: "Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points"."
- [intro] Co-occurring truncation points define UTR end: "When piled up, each co-occurring truncation point is used to determine the precise end base of a given UTR."
- [readme] Demo BAM input for validation: "This uses a small demo set of input files contained in the repository"
- [intro] Read-end signal enrichment assumption: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [intro] peaks2utr consumes BAM-aligned reads: "peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format"
