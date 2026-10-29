---
name: bam-file-strand-separation
description: Use when input is a strand-specific aligned BAM (e.g. Tb927_01_v5.1.slice.bam)
  and the next step is per-strand coverage/peak calling toward 3' UTR annotation.
when_to_use_negative:
- The goal is merged whole-genome coverage signal — peaks2utr's design keeps strand
  coverage separate, so this stage is wrong when cross-strand merging is intended
- The downstream goal is already satisfied — this stage must not apply gene-search
  or UTR designation; those belong to a separate downstream task
- The input is not aligned reads (e.g. GFF annotation only) — peaks2utr's MACS peak
  calling operates on the BAM file, so there is nothing to strand-split
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: peaks2utr
  role: Tool whose workflow this stage belongs to; it strand-splits the aligned BAM
    and drives per-strand broad-peak calling
  repo: https://github.com/haessar/peaks2utr
- name: MACS3
  role: Called in broad-peak mode separately on each strand-split BAM to produce per-strand
    significant-read-coverage peak calls
  repo: https://pypi.org/project/MACS3/
- name: Python 3
  role: Implementation language of peaks2utr, which orchestrates the strand-split
    and per-strand MACS calls
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:bbe2e68b7db411a15183afe2be59d0a6109dacc368feba43b23eb4b7b49d068c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - MACS3
  - Python 3
  - peaks2utr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bam-file-strand-separation

## Summary

Separate an aligned, strand-specific BAM file into plus-strand and minus-strand read files so downstream coverage analysis (e.g. MACS broad-peak calling in peaks2utr) is performed independently per strand, never merging coverage signal from the two strands.

## When to use

Use when input is a strand-specific aligned BAM (e.g. Tb927_01_v5.1.slice.bam) and the next step is per-strand coverage/peak calling toward 3' UTR annotation.

## When NOT to use

- The goal is merged whole-genome coverage signal — peaks2utr's design keeps strand coverage separate, so this stage is wrong when cross-strand merging is intended
- The downstream goal is already satisfied — this stage must not apply gene-search or UTR designation; those belong to a separate downstream task
- The input is not aligned reads (e.g. GFF annotation only) — peaks2utr's MACS peak calling operates on the BAM file, so there is nothing to strand-split

## Inputs

- Aligned sequencing reads in BAM format with strand information (e.g. demo Tb927_01_v5.1.slice.bam)

## Outputs

- Plus-strand BAM (plus-strand reads only)
- Minus-strand BAM (minus-strand reads only)
- Per-strand broadPeak BED-format files of MACS broad-peak intervals (no UTR designation applied)

## How to apply

Start from the aligned reads BAM such as the demo Tb927_01_v5.1.slice.bam and use the strand field of the alignments to produce two files: one containing plus-strand reads and one containing minus-strand reads. Keep the two files separate rather than merging them, because peaks2utr's peak-calling stage invokes MACS (MACS3, installable from PyPI) as two independent broad-peak runs — one on the plus-strand BAM and one on the minus-strand BAM — so coverage signal from the two strands is never combined. The rationale is that 3' UTR enrichment signal is inherently strand- and end-concentrated in data like 10x Chromium runs, so merged-strand coverage would blur strand-specific peak intervals. Each per-strand run emits a broadPeak BED-format file of significant-read-coverage intervals; no gene-search or UTR designation happens in this stage, that is handled downstream. Validate before proceeding: each per-strand BAM is non-empty and has correct flags, and each per-strand peak file exists and parses as a broadPeak BED file.

## Related tools

- **peaks2utr** (Tool whose workflow this stage belongs to; it strand-splits the aligned BAM and drives per-strand broad-peak calling) — https://github.com/haessar/peaks2utr
- **MACS3** (Called in broad-peak mode separately on each strand-split BAM to produce per-strand significant-read-coverage peak calls) — https://pypi.org/project/MACS3/
- **Python 3** (Implementation language of peaks2utr, which orchestrates the strand-split and per-strand MACS calls)

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Each per-strand peak file exists and is parseable as a broadPeak BED file
- Each per-strand peak file contains peak intervals representing regions of significant read coverage called by MACS
- Two distinct peak-call sets are produced (plus-strand and minus-strand) rather than a single merged set
- Both strand-split BAMs are non-empty and contain only reads of the expected strand
- The per-strand peak files contain no gene-search or UTR-designation output (that is downstream)

## Limitations

- Installation instructions assume a Debian / Ubuntu system with root privileges
- No software version numbers are stated for peaks2utr or its dependencies
- No changelog found for the repository

## Evidence

- [intro] peaks2utr uses MACS to call broad peaks: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file"
- [readme] peaks2utr annotates 3' UTRs for a given set of aligned sequencing reads in BAM format, and canonical annotation in GFF or GTF format.: "peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format"
- [intro] This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs.: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [readme] Installation instructions assume a Debian / Ubuntu system with root privileges.: "Installation instructions assume a Debian / Ubuntu system with root privileges. Follow the links for instructions for other systems."
