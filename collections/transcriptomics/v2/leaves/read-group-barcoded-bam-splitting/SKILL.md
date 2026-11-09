---
name: read-group-barcoded-bam-splitting
description: Use when a BAM of aligned sequencing reads (e.g. from 10x Chromium single-cell
  runs) must be split by SAM FLAG reverse-strand bit and RG tag into per-strand, per-read-group
  BAMs before peak calling or coverage analysis.
when_to_use_negative:
- The input is not a BAM file of aligned reads but already a peak bedGraph, GFF/GTF
  annotation, or other non-alignment format
- Reads are single-end/unstranded with no usable FLAG reverse-strand bit or no RG
  tags, so strand/read-group partitioning would be meaningless
- The downstream tool accepts the merged BAM directly and performs its own strand/read-group
  handling
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: pysam
  role: Python wrapper for htslib/SAMtools used to open the BAM, read FLAG and RG
    tags per alignment, and write the strand- and read-group-partitioned BAM files
    with the original header
  repo: https://github.com/haessar/peaks2utr
- name: peaks2utr
  role: Pipeline whose preprocessing stage performs the strand/read-group BAM splitting
    before applying the SPAT algorithm and MACS peak calling
  repo: https://github.com/haessar/peaks2utr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:b63be45f966707a56ea1aab1287929f1d15296244aad88819f65af9676aa368e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - peaks2utr
  - pysam
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# read-group-barcoded-bam-splitting

## Summary

Partitions an input BAM file into forward- and reverse-strand files, each further subdivided by read group (RG) tag, using pysam so that downstream 3′ UTR peak calling (SPAT and MACS) operates on strand- and read-group-consistent read piles.

## When to use

Use when a BAM of aligned sequencing reads (e.g. from 10x Chromium single-cell runs) must be split by SAM FLAG reverse-strand bit and RG tag into per-strand, per-read-group BAMs before peak calling or coverage analysis.

## When NOT to use

- The input is not a BAM file of aligned reads but already a peak bedGraph, GFF/GTF annotation, or other non-alignment format
- Reads are single-end/unstranded with no usable FLAG reverse-strand bit or no RG tags, so strand/read-group partitioning would be meaningless
- The downstream tool accepts the merged BAM directly and performs its own strand/read-group handling

## Inputs

- Aligned reads in BAM format (e.g. peaks2utr demo file Tb927_01_v5.1.slice.bam)
- BAM header reused for output files (sort order, reference sequences)
- SAM FLAG field reverse-strand bit per read
- Read group (RG) tag per read

## Outputs

- One BAM file per strand per read group, each carrying a copy of the original header
- Validation report confirming per-read strand flag and RG tag consistency

## How to apply

Open the input BAM (e.g. Tb927_01_v5.1.slice.bam) with pysam's AlignmentFile, preserving its header for reuse in the output partitions. Iterate over aligned reads and inspect the SAM FLAG field's reverse-strand bit to route each read into a forward- or reverse-strand pile, then subdivide each strand pile by the read's read group (RG) tag. Write each strand × read-group bucket as its own BAM file with pysam, copying the original header so sort order and reference sequences are retained. The rationale is that 3′ UTR signal (truncation points from soft-clipped bases and polyA-tails) is strand-specific and read-group-dependent, so merging strands or read groups would blur the coverage peaks that MACS and the SPAT algorithm rely on.

## Related tools

- **pysam** (Python wrapper for htslib/SAMtools used to open the BAM, read FLAG and RG tags per alignment, and write the strand- and read-group-partitioned BAM files with the original header) — https://github.com/haessar/peaks2utr
- **peaks2utr** (Pipeline whose preprocessing stage performs the strand/read-group BAM splitting before applying the SPAT algorithm and MACS peak calling) — https://github.com/haessar/peaks2utr

## Examples

```
from pysam import AlignmentFile; bam = AlignmentFile('Tb927_01_v5.1.slice.bam', 'rb'); [r for r in bam.fetch(until_eof=True) if r.is_reverse]
```

## Evaluation signals

- Reopen every output BAM with pysam and confirm each contained read carries the expected strand flag (reverse-strand bit consistent with the partition it is in)
- Confirm each read in a read-group partition carries the expected RG tag
- Confirm the summed read counts across all strand × read-group partitions equal the total read count of the input demo BAM (exact-sum consistency is the acceptance criterion)
- Verify each output BAM's header matches the original header (sort order and reference sequences retained)

## Limitations

- No quantitative read-count threshold is reported for the partitioning check; only the exact-sum consistency criterion is specified
- No software version numbers are stated for the tools in the workflow
- Installation of dependencies (e.g. bedtools) assumes a Debian/Ubuntu system with root privileges, which may block reproducibility elsewhere

## Evidence

- [other] BAM partitioning by strand and read group: [evidence span withheld — anchor status: not_found]
- [intro] Downstream peak calling on the BAM: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file"
- [readme] Input format for the pipeline: "peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format"
- [intro] Signal concentration rationale: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
