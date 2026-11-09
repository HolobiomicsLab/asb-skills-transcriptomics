---
name: peak-to-gene-strand-distance-matching
description: Use when MACS-called peaks from a 10x Chromium-like BAM must be filtered
  to gene-associated candidates — same strand as a gene and within a maximum distance
  — before designating 3' UTRs.
when_to_use_negative:
- The BAM comes from a protocol where signal is not concentrated at the distal ends
  of 3' or 5' UTRs, so strand/distance-matched peaks will not correspond to UTR ends.
- The goal is de novo peak calling or general ChIP/RIP peak analysis rather than annotating
  3' UTRs against a canonical GFF/GTF.
- Pre-existing 3' UTR annotations are already complete and must neither be extended
  nor overridden by new peak-based calls.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: peaks2utr
  role: Python 3 CLI implementing the strand/distance peak-to-gene matching filter
    and 3' UTR designation; installed via pip or built from source
  repo: https://github.com/haessar/peaks2utr
- name: MACS3
  role: Calls broad 'peaks' of significant read coverage in the BAM that are then
    filtered by the strand/distance criteria
- name: bedtools
  role: Required dependency for interval operations underlying peak-gene matching
  repo: https://bedtools.readthedocs.io/en/latest/content/installation.html
- name: GenomeTools
  role: Optional post-processing of the output gff3
  repo: https://github.com/genometools/genometools#building-and-installation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:53cf604ff971ae69d51f133111e6b25024e2cd8e6ac107059f1db45745c9bbf2
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# peak-to-gene-strand-distance-matching

## Summary

Filter stage of peaks2utr in which MACS3-called broad peaks of significant read coverage are matched to gene models (GFF/GTF) on the same strand within a user-defined maximum distance; only passing peaks are designated novel 3' UTRs and truncated at SPAT pile-up truncation points.

## When to use

Use when MACS-called peaks from a 10x Chromium-like BAM must be filtered to gene-associated candidates — same strand as a gene and within a maximum distance — before designating 3' UTRs.

## When NOT to use

- The BAM comes from a protocol where signal is not concentrated at the distal ends of 3' or 5' UTRs, so strand/distance-matched peaks will not correspond to UTR ends.
- The goal is de novo peak calling or general ChIP/RIP peak analysis rather than annotating 3' UTRs against a canonical GFF/GTF.
- Pre-existing 3' UTR annotations are already complete and must neither be extended nor overridden by new peak-based calls.

## Inputs

- Canonical gene annotation in GFF3 or GTF format (existing 3' UTRs optional)
- Aligned sequencing reads in BAM format (favouring 10x Chromium-style UTR-enriched signal)
- User-defined maximum peak-to-gene distance parameter
- bedtools (required dependency)

## Outputs

- GFF3 annotation file (default *.new.<ext> suffix, or path given with -o/--output) containing original features plus novel 3' UTR features with source=peaks2utr
- GTF output if --gtf is passed

## How to apply

Run peaks2utr with a canonical annotation (GFF3 or GTF, existing 3' UTRs optional) and an aligned BAM; MACS3 calls broad 'peaks' of significant read coverage in the BAM. Each peak is then matched against gene models using two criteria: it must lie on the same strand as a gene, and within the user-defined maximum distance of that gene; peaks failing either criterion are discarded and never become UTR candidates. Passing peaks form the basis of novel 3' UTR annotations, which are truncated at SPAT pile-up truncation points where possible — truncation points come from reads with soft-clipped bases and polyA-tails whose end bases are tallied and piled up to determine the precise UTR end base. The behaviour can be tuned to extend, override or ignore pre-existing 3' UTR annotations in the input GFF. Signal is expected to be concentrated at the distal ends of 3' or 5' UTRs, as in 10x Chromium runs.

## Related tools

- **peaks2utr** (Python 3 CLI implementing the strand/distance peak-to-gene matching filter and 3' UTR designation; installed via pip or built from source) — https://github.com/haessar/peaks2utr
- **MACS3** (Calls broad 'peaks' of significant read coverage in the BAM that are then filtered by the strand/distance criteria)
- **bedtools** (Required dependency for interval operations underlying peak-gene matching) — https://bedtools.readthedocs.io/en/latest/content/installation.html
- **GenomeTools** (Optional post-processing of the output gff3) — https://github.com/genometools/genometools#building-and-installation

## Examples

```
peaks2utr Tb927_01_v5.1.gff Tb927_01_v5.1.slice.bam -p 4 -o output.gff3
```

## Evaluation signals

- Output GFF3 exists with the *.new.<ext> suffix (e.g. Tb927_01_v5.1.new.gff) or the path given with -o
- Output contains the original annotations plus new 3' UTR features whose source field equals 'peaks2utr'
- peaks2utr-check demo run succeeds, producing Tb927_01_v5.1.new.gff from the bundled Tb927_01_v5.1.gff and Tb927_01_v5.1.slice.bam
- Retained peaks lie on the same strand as a gene and within the user-defined maximum distance; peaks failing these criteria do not appear as 3' UTR features
- Where SPAT pile-up truncation points exist, the called 3' UTR end base coincides with the piled-up co-occurring truncation point

## Limitations

- A 3' UTR end can only be set where reads yield SPAT pile-up truncation points; truncation is applied 'where possible'.
- Installation instructions assume a Debian/Ubuntu system with root privileges (bedtools required via apt-get).
- No software version numbers or changelog are stated in the context; the approach favours 10x Chromium-style BAMs with signal at UTR distal ends.

## Evidence

- [intro] peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file, and uses those peaks that pass a set of criteria as a basis to annotate novel: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file, and uses those peaks that pass a set of criteria as a basis to annotate novel"
- [intro] Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points".: "Reads containing soft-clipped bases and polyA-tails of a given length are detected, and their end bases tallied as "truncation points"."
- [intro] This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs.: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
- [readme] peaks2utr can be tuned to extend, override or ignore any pre-existing 3' UTR annotations in the input GFF file.: [evidence span withheld — anchor status: external_doc]
- [readme] When complete, you should see a file `Tb927_01_v5.1.new.gff` which contains original annotations as well as 3' UTRs with source "peaks2utr".: "When complete, you should see a file `Tb927_01_v5.1.new.gff` which contains original annotations as well as 3' UTRs with source "peaks2utr"."
