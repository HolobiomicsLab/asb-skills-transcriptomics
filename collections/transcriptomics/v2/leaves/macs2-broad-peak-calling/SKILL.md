---
name: macs2-broad-peak-calling
description: Use when the analysis goal is to identify regions of significant read
  enrichment from BAM-format aligned reads (e.g. 10x Chromium 3' end-enriched signal)
  as candidate features, prior to downstream filtering or annotation.
when_to_use_negative:
- The goal is narrow punctate peak detection (e.g. TF binding sites) rather than broad
  regions of enriched coverage — peaks2utr explicitly calls broad peaks
- Reads are not 3' end-enriched; the approach favours signal concentrated at UTR distal
  ends such as 10x Chromium runs
- Downstream UTR designation is already done — this stage produces raw candidate regions
  only, with no gene-search or UTR annotation applied
edam_operation: http://edamontology.org/operation_0367
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: MACS3
  role: Peak-calling engine invoked by peaks2utr to call broad 'peaks' of significant
    read coverage in the BAM file
  repo: https://pypi.org/project/MACS3/
- name: peaks2utr
  role: Python CLI that orchestrates the MACS broad-peak call as the first stage of
    3' UTR annotation
  repo: https://github.com/haessar/peaks2utr
- name: bedtools
  role: Required dependency for peaks2utr's interval-based processing around the peak-calling
    stage
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:8a8aabdbd1386a1261c8f87c7e4b88456d7aa7d0ac6aa6e7427a379e06e94369
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MACS3
  - Python 3
  - bedtools
  - peaks2utr
  edam_operation: http://edamontology.org/operation_0367
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# macs2-broad-peak-calling

## Summary

Calls broad 'peaks' of significant read coverage in an aligned BAM file using MACS (MACS3 from PyPI), producing per-strand broadPeak intervals that peaks2utr uses as candidate regions for annotating novel 3' UTRs.

## When to use

Use when the analysis goal is to identify regions of significant read enrichment from BAM-format aligned reads (e.g. 10x Chromium 3' end-enriched signal) as candidate features, prior to downstream filtering or annotation.

## When NOT to use

- The goal is narrow punctate peak detection (e.g. TF binding sites) rather than broad regions of enriched coverage — peaks2utr explicitly calls broad peaks
- Reads are not 3' end-enriched; the approach favours signal concentrated at UTR distal ends such as 10x Chromium runs
- Downstream UTR designation is already done — this stage produces raw candidate regions only, with no gene-search or UTR annotation applied

## Inputs

- Aligned sequencing reads in BAM format (e.g. the demo Tb927_01_v5.1.slice.bam)
- Optionally strand-split BAM files (plus-strand and minus-strand reads) for per-strand peak calling

## Outputs

- Broad 'peaks' of significant read coverage as broadPeak BED-format files, one per strand

## How to apply

Run MACS3 in broad-peak mode on the aligned reads, separating plus-strand and minus-strand reads so coverage signal from the two strands is never merged. The peaks2utr workflow invokes MACS (installed via https://pypi.org/project/MACS3/) as the first stage, before any gene-search or UTR designation; the resulting broadPeak BED-format intervals are the candidate regions that later pass through peaks2utr's set of criteria. Rationale for broad-peak mode: the input favours BAM files from runs like 10x Chromium where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs, so broad enriched regions — not narrow punctate peaks — are the correct unit of interest. Validate each output peak file as a parseable broadPeak BED file containing non-empty intervals of significant coverage.

## Related tools

- **MACS3** (Peak-calling engine invoked by peaks2utr to call broad 'peaks' of significant read coverage in the BAM file) — https://pypi.org/project/MACS3/
- **peaks2utr** (Python CLI that orchestrates the MACS broad-peak call as the first stage of 3' UTR annotation) — https://github.com/haessar/peaks2utr
- **bedtools** (Required dependency for peaks2utr's interval-based processing around the peak-calling stage) — https://bedtools.readthedocs.io/en/latest/content/installation.html

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Each per-strand peak file exists and is parseable as a broadPeak BED file
- Peak files contain non-empty intervals representing regions of significant read coverage called by MACS
- Plus-strand and minus-strand peak calls are produced as separate runs, never merged across strands
- Running peaks2utr-check with the demo inputs (Tb927_01_v5.1.gff + Tb927_01_v5.1.slice.bam) completes and yields Tb927_01_v5.1.new.gff with 3' UTR features of source 'peaks2utr'

## Limitations

- No software version numbers are stated in the paper's installation instructions ('Install latest release with:')
- Installation instructions assume a Debian / Ubuntu system with root privileges (e.g. apt-get install bedtools)
- Peak calling is only the first step: peaks must subsequently pass a set of criteria before being used to annotate novel 3' UTRs

## Evidence

- [intro] MACS broad-peak call as stage 1 of peaks2utr: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file"
- [intro] Signal concentration rationale for broad peaks: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [intro] Peaks are filtered by criteria before UTR annotation: "uses those peaks that pass a set of criteria as a basis to annotate novel"
- [readme] Installation context and system assumptions: "Installation instructions assume a Debian / Ubuntu system with root privileges."
- [readme] Demo invocation with BAM input: "peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3"
