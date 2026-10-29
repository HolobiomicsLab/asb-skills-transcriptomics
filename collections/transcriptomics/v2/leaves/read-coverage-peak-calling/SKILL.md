---
name: read-coverage-peak-calling
description: Use when a BAM of aligned 3'-end-enriched reads (e.g. 10x Chromium scRNA-seq)
  must be scanned for broad coverage peaks to serve as candidate 3' UTRs against an
  existing GFF3/GTF gene model.
when_to_use_negative:
- The sequencing data is not 3'-end enriched (e.g. standard full-transcript RNA-seq),
  since the method relies on signal concentrated at UTR distal ends as in 10x Chromium
  runs
- Input is raw reads rather than an aligned BAM, or no canonical GFF/GTF annotation
  exists to attach UTRs to
- The goal is narrow ChIP-seq-style sharp peak calling rather than broad read-coverage
  regions for UTR annotation
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: peaks2utr
  role: Python 3 CLI that orchestrates the peak calling and converts passing peaks
    into 3' UTR annotations
  repo: https://github.com/haessar/peaks2utr
- name: MACS3
  role: Called by peaks2utr to detect broad peaks of significant read coverage in
    the BAM
  repo: https://pypi.org/project/MACS3/
- name: bedtools
  role: Required system dependency for peak/gene interval operations
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
- name: GenomeTools
  role: Optional post-processing of the output GFF3
  repo: https://github.com/genometools/genometools
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:fc2e7fa1d7358eb33722925bd7a2921e8e19e546a2d4a5a81a97253b576c3cfe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - GenomeTools
  - MACS3
  - Python
  - bedtools
  - peaks2utr
  - pip
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# read-coverage-peak-calling

## Summary

Call broad peaks of significant read coverage from an aligned BAM file using MACS3, as the first stage of peaks2utr, to identify candidate 3' UTR regions from single-cell (e.g. 10x Chromium) read enrichment.

## When to use

Use when a BAM of aligned 3'-end-enriched reads (e.g. 10x Chromium scRNA-seq) must be scanned for broad coverage peaks to serve as candidate 3' UTRs against an existing GFF3/GTF gene model.

## When NOT to use

- The sequencing data is not 3'-end enriched (e.g. standard full-transcript RNA-seq), since the method relies on signal concentrated at UTR distal ends as in 10x Chromium runs
- Input is raw reads rather than an aligned BAM, or no canonical GFF/GTF annotation exists to attach UTRs to
- The goal is narrow ChIP-seq-style sharp peak calling rather than broad read-coverage regions for UTR annotation

## Inputs

- BAM file of aligned sequencing reads (e.g. a 10x Chromium scRNA-seq BAM; demo: Tb927_01_v5.1.slice.bam)
- Canonical gene model annotation in GFF3 or GTF format (existing 3' UTRs optional; demo: Tb927_01_v5.1.gff)
- Optional flags: parallel workers (-p), output path (-o), --gtf, options to extend/override/ignore pre-existing 3' UTRs

## Outputs

- GFF3 annotation file (default suffix *.new.<ext>) containing original features plus 3' UTR features with source=peaks2utr
- GTF output instead of GFF3 when --gtf is passed

## How to apply

Run peaks2utr with a canonical annotation (GFF3 or GTF) and a BAM of aligned reads; internally it invokes MACS (MACS3 via PyPI) to call broad peaks of significant read coverage in the BAM. Because 10x Chromium libraries concentrate signal at the distal ends of UTRs, these broad peaks delineate candidate UTR regions. Peaks passing the downstream criteria — same strand as a gene within a user-defined maximum distance — are retained as 3' UTR candidates, while reads with soft-clipped bases and polyA tails of a given length supply 'truncation points' to set the precise UTR end base. Pre-existing 3' UTR annotations in the input GFF can be tuned to be extended, overridden or ignored. Results are written as 3' UTR features with source=peaks2utr into an output GFF3 (or GTF with --gtf), with parallel workers (e.g. -p 4) for speed.

## Related tools

- **peaks2utr** (Python 3 CLI that orchestrates the peak calling and converts passing peaks into 3' UTR annotations) — https://github.com/haessar/peaks2utr
- **MACS3** (Called by peaks2utr to detect broad peaks of significant read coverage in the BAM) — https://pypi.org/project/MACS3/
- **bedtools** (Required system dependency for peak/gene interval operations) — https://bedtools.readthedocs.io/en/latest/content/installation.html
- **GenomeTools** (Optional post-processing of the output GFF3) — https://github.com/genometools/genometools

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Output file (default *.new.<ext>) contains all original input annotation features plus new 3' UTR features whose source attribute is 'peaks2utr'
- peaks2utr-check demo run on Tb927_01_v5.1.gff / Tb927_01_v5.1.slice.bam produces Tb927_01_v5.1.new.gff as expected
- On the full T. brucei Briggs et al. 2021 scRNA-seq BAM (~140M reads, 12.4 GB), the pipeline reproduces 9888 total 3' UTRs (3709 new / 5347 altered / 832 unchanged) covering ~84% of 11703 genes
- Every retained peak maps to the same strand as a gene within the user-defined maximum distance; UTR end bases coincide with piled-up SPAT truncation points
- Optional GenomeTools post-processing validates/normalizes the emitted GFF3

## Limitations

- Assumes a Debian/Ubuntu system with root privileges for dependency installation (apt-get install bedtools)
- Method favours BAMs from single-cell 3'-end-enriched protocols (10x Chromium); read enrichment at UTR distal ends is what makes broad coverage peaks informative
- No software version numbers stated in the installation instructions and no changelog available

## Evidence

- [intro] peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file"
- [abstract] uses the UTR enrichment of single-cell technologies, such as 10× Chromium, to accurately annotate 3' UTRs: "uses the UTR enrichment of single-cell technologies, such as 10× Chromium, to accurately annotate 3' UTRs"
- [readme] This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs.: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [methods] peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3: "peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3"
- [intro] Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points".: "Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points"."
