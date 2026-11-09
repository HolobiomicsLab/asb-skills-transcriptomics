---
name: bam-cigar-softclip-parsing
description: Use when a BAM of aligned sequencing reads (e.g. 10x Chromium 3'-enriched
  data) must be mined for soft-clipped reads bearing polyA-tails to derive per-position
  truncation-point tallies for 3' UTR end determination.
when_to_use_negative:
- Input reads lack soft-clipped bases or polyA/polyT tails (e.g. non-3'-enriched or
  long-read data without template-switch artifacts) — no truncation points can be
  tallied.
- The goal is peak calling or UTR annotation itself rather than the truncation-point
  tallying step — use the full peaks2utr pipeline with MACS peak calling instead of
  this single SPAT stage.
- The BAM is not strand-resolved and per-strand truncation points are required, since
  tallying is performed per chromosome and strand.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: peaks2utr
  role: Source implementation of the SPAT truncation-point tallying stage being reconstructed
  repo: https://github.com/haessar/peaks2utr
- name: Python 3
  role: Language used to iterate over BAM alignments, parse CIGAR strings, screen
    polyA-tails, and write the tally TSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:53fc788f8eb9c79bd09ecf08fc1fce438dac862160f8f69c85ac713931e9191b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - Python 3
  - peaks2utr
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bam-cigar-softclip-parsing

## Summary

Reconstruction of the SPAT stage of peaks2utr: parse CIGAR strings from a strand-split BAM to flag soft-clipped reads, screen terminal unaligned sequence for polyA/polyT tails of a minimum length, and tally each qualifying read's final aligned end base as a truncation point.

## When to use

Use when a BAM of aligned sequencing reads (e.g. 10x Chromium 3'-enriched data) must be mined for soft-clipped reads bearing polyA-tails to derive per-position truncation-point tallies for 3' UTR end determination.

## When NOT to use

- Input reads lack soft-clipped bases or polyA/polyT tails (e.g. non-3'-enriched or long-read data without template-switch artifacts) — no truncation points can be tallied.
- The goal is peak calling or UTR annotation itself rather than the truncation-point tallying step — use the full peaks2utr pipeline with MACS peak calling instead of this single SPAT stage.
- The BAM is not strand-resolved and per-strand truncation points are required, since tallying is performed per chromosome and strand.

## Inputs

- Strand-split BAM file of aligned sequencing reads (e.g. Tb927_01_v5.1.slice.bam)
- Configured minimum polyA-tail length threshold

## Outputs

- truncation_points.tsv with one row per end-base genomic coordinate and its read count, split by chromosome and strand
- Per-read truncation-point coordinates (genomic position of the final aligned end base of each soft-clipped, polyA-bearing read)

## How to apply

Load the strand-split BAM (e.g. the repository demo Tb927_01_v5.1.slice.bam) and iterate over alignments per reference sequence and strand in Python 3, mirroring the SPAT stage of peaks2utr. For each read, parse the CIGAR string to flag soft-clipped base operations, and screen the terminal unaligned sequence for a poly(A) tail of the configured minimum length (the reverse-complement poly(T) stretch on the reverse strand). Record the genomic coordinate of the final aligned end base of every read that passes both the soft-clip and polyA-tail screens. Tally co-occurring end-base coordinates per chromosome and strand and write the counts to truncation_points.tsv, one row per end-base position with its read count. Each qualifying read must contribute exactly one end base, so piled-up co-occurring truncation points can determine the precise end base of a given UTR downstream.

## Related tools

- **peaks2utr** (Source implementation of the SPAT truncation-point tallying stage being reconstructed) — https://github.com/haessar/peaks2utr
- **Python 3** (Language used to iterate over BAM alignments, parse CIGAR strings, screen polyA-tails, and write the tally TSV)

## Examples

```
python -c "import pysam; bam=pysam.AlignmentFile('Tb927_01_v5.1.slice.bam','rb'); [r for r in bam.fetch() if 'S' in r.cigarstring]"
```

## Evaluation signals

- Each read containing soft-clipped bases and a polyA-tail of the configured minimum length contributes exactly one end base to the tally (no double-counting).
- Reverse-strand reads are screened for the reverse-complement poly(T) stretch rather than poly(A), and tallies are split per chromosome and strand.
- truncation_points.tsv contains one row per end-base position with its read count, with coordinates matching the final aligned base implied by the CIGAR (not the soft-clipped region).
- Runs on the demo Tb927_01_v5.1.slice.bam should yield piled-up co-occurring truncation points at 3' UTR distal ends consistent with downstream UTR end determination.
- Reads failing either screen (no soft-clip, or tail shorter than the minimum length) appear nowhere in the tally.

## Limitations

- Detection depends on a configured minimum polyA-tail length; reads with shorter tails are silently missed, and the article/README does not state version numbers or a default threshold value.
- The method favours BAM files from 10x Chromium-style runs where signal is concentrated at UTR distal ends; other library types may yield sparse or misleading tallies.
- Tallied truncation points are only a precursor signal — they must be piled up co-occurring per position before the precise UTR end base can be determined.

## Evidence

- [intro] Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points".: "Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points"."
- [intro] This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs.: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [readme] peaks2utr <GFF_IN> <BAM_IN> [options]: "peaks2utr is called from the command line as:
```
peaks2utr <GFF_IN> <BAM_IN> [options]
```
### Inputs
* `GFF_IN` - gene models"
- [readme] When piled up, each co-occurring truncation point is used to determine the precise end base of a given UTR.: "When piled up, each co-occurring truncation point is used to determine the precise end base of a given UTR."
