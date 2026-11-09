---
name: utr-enrichment-peak-detection
description: Use when annotating 3' UTRs from aligned reads (e.g. 10x Chromium BAM
  signal concentrated at UTR distal ends) and you need to identify candidate regions
  of significant read coverage before UTR end determination.
when_to_use_negative:
- The BAM signal is diffuse across transcript bodies rather than concentrated at UTR
  distal ends (the approach favours 10x Chromium-style runs)
- You need precise UTR end bases directly from peak calls — peaks are only the candidate
  regions; truncation-point pile-up is a separate downstream step
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: peaks2utr
  role: Python CLI wrapper that invokes MACS on the BAM file and applies peak-selection
    criteria toward 3' UTR annotation
  repo: https://github.com/haessar/peaks2utr
- name: MACS3
  role: Peak caller invoked to find broad 'peaks' of significant read coverage in
    the BAM file
- name: bedtools
  role: Required dependency of the peaks2utr installation used in this workflow
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:e2be5c265ea0ed7c4c75d96bda4214e90e37848ff8cb5be9f96405fec6a96c68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - MACS3
  - Python 3
  - bedtools
  - peaks2utr
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# utr-enrichment-peak-detection

## Summary

Call broad peaks of significant read coverage from an aligned BAM file using MACS (MACS3), as the first stage of peaks2utr's pipeline for annotating novel 3' UTRs. The peaks are the candidate regions on which downstream UTR-extension criteria are applied.

## When to use

Use when annotating 3' UTRs from aligned reads (e.g. 10x Chromium BAM signal concentrated at UTR distal ends) and you need to identify candidate regions of significant read coverage before UTR end determination.

## When NOT to use

- The BAM signal is diffuse across transcript bodies rather than concentrated at UTR distal ends (the approach favours 10x Chromium-style runs)
- You need precise UTR end bases directly from peak calls — peaks are only the candidate regions; truncation-point pile-up is a separate downstream step

## Inputs

- Aligned sequencing reads in BAM format (e.g. Tb927_01_v5.1.slice.bam)
- Canonical annotation in GFF3 or GTF format (GFF_IN, existing 3' UTRs optional)

## Outputs

- Broad 'peaks' of significant read coverage passing a set of criteria, used as the basis for annotating novel 3' UTRs
- Downstream GFF3 (or GTF with --gtf) annotation file containing original features plus 3' UTR features with source=peaks2utr

## How to apply

Take the aligned BAM input (e.g. the demo Tb927_01_v5.1.slice.bam) and run MACS (MACS3, installed via PyPI) in broad-peak mode to call broad 'peaks' of significant read coverage; peaks2utr performs this step internally as part of `peaks2utr <GFF_IN> <BAM_IN>`. Only peaks that pass a set of criteria are retained as a basis for annotating novel 3' UTRs, so the peak call should be evaluated as a candidate-generation step rather than a final result. The approach favours BAM files from 10x Chromium-style runs where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs. After peak calling, peaks2utr proceeds to detect soft-clipped bases and polyA-tails of a given length and tally truncation points to determine precise UTR end bases.

## Related tools

- **peaks2utr** (Python CLI wrapper that invokes MACS on the BAM file and applies peak-selection criteria toward 3' UTR annotation) — https://github.com/haessar/peaks2utr
- **MACS3** (Peak caller invoked to find broad 'peaks' of significant read coverage in the BAM file)
- **bedtools** (Required dependency of the peaks2utr installation used in this workflow) — https://bedtools.readthedocs.io/en/latest/content/installation.html

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- The run completes and `peaks2utr-check` passes, confirming correct installation and a short run with default parameters
- Output GFF3/GTF contains the original features plus 3' UTR features with source=peaks2utr (default output is the original filename with a *.new.<ext> suffix)
- Peaks retained are those passing the tool's set of criteria; peaks failing the criteria are excluded from UTR annotation

## Limitations

- Approach favours BAM files from runs like 10x Chromium where signal is concentrated at UTR distal ends; other read distributions may not yield informative peaks
- No software version numbers are stated for the dependencies ('Install latest release with:')
- Installation instructions assume a Debian / Ubuntu system with root privileges

## Evidence

- [intro] MACS calls broad peaks on BAM: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file"
- [intro] peaks passing criteria basis for novel UTRs: "and uses those peaks that pass a set of criteria as a basis to annotate novel 3' UTRs"
- [readme] Favours 10x Chromium BAM signal: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [readme] Truncation points determine UTR end: "Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points"."
