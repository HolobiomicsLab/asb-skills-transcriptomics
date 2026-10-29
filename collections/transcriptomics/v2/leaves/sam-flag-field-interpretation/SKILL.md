---
name: sam-flag-field-interpretation
description: 'Use when a BAM-handling step needs to route aligned reads by strand
  or read-group: e.g. building strand-specific or per-RG BAM partitions from a single
  input BAM prior to downstream peak calling or pileup analyses.'
when_to_use_negative:
- Input is not aligned reads in BAM format (e.g. you only have GFF/GTF annotation
  or raw reads) — the FLAG and RG fields do not exist there
- You need peak calling or UTR annotation itself rather than BAM partitioning — use
  peaks2utr's MACS-based peak calling and truncation-point pipeline instead
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0080
tools:
- name: pysam
  role: Python wrapper over htslib/SAMtools used to open the BAM, read the FLAG field's
    reverse-strand bit and RG tags, and write strand/RG-partitioned BAM files with
    the copied header
- name: peaks2utr
  role: The pipeline whose preprocessing stage performs the strand/read-group BAM
    splitting with pysam before SPAT and MACS-based peak calling
  repo: https://github.com/haessar/peaks2utr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:6df8bd1167482e247093196276d115bcdf82d642cbaa6f271110b51ec7987a94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - peaks2utr
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# SAM FLAG field interpretation

## Summary

Interpreting the SAM/BAM FLAG field's reverse-strand bit and the RG read-group tag to partition aligned reads, as done by peaks2utr's preprocessing stage, which splits each input BAM by strand and read group using pysam before SPAT and peak calling.

## When to use

Use when a BAM-handling step needs to route aligned reads by strand or read-group: e.g. building strand-specific or per-RG BAM partitions from a single input BAM prior to downstream peak calling or pileup analyses.

## When NOT to use

- Input is not aligned reads in BAM format (e.g. you only have GFF/GTF annotation or raw reads) — the FLAG and RG fields do not exist there
- You need peak calling or UTR annotation itself rather than BAM partitioning — use peaks2utr's MACS-based peak calling and truncation-point pipeline instead

## Inputs

- Aligned reads in BAM format (e.g. the peaks2utr demo BAM Tb927_01_v5.1.slice.bam)
- BAM header defining references and sort order

## Outputs

- Per-strand, per-read-group BAM files (forward-strand and reverse-strand partitions, each subdivided by RG tag)
- Validation report: per-partition strand-flag and RG-tag checks plus a read-count conservation check against the input BAM

## How to apply

Open the input BAM with pysam's AlignmentFile, preserving its header for reuse. Iterate over aligned reads and inspect the SAM FLAG field's reverse-strand bit to route each read into a forward-strand or reverse-strand pile. Within each strand pile, subdivide reads by their RG (read-group) tag into per-read-group buckets. Write each bucket as its own BAM with pysam, copying the original header so sort order and references are retained. Validate by reopening every output BAM with pysam, checking each read carries the expected strand flag and read-group tag, and confirming summed read counts across partitions equal the input BAM's total read count (exact-sum consistency is the acceptance criterion).

## Related tools

- **pysam** (Python wrapper over htslib/SAMtools used to open the BAM, read the FLAG field's reverse-strand bit and RG tags, and write strand/RG-partitioned BAM files with the copied header)
- **peaks2utr** (The pipeline whose preprocessing stage performs the strand/read-group BAM splitting with pysam before SPAT and MACS-based peak calling) — https://github.com/haessar/peaks2utr

## Examples

```
from pysam import AlignmentFile; bam = AlignmentFile('Tb927_01_v5.1.slice.bam', 'rb'); fwd = [r for r in bam if not r.is_reverse]; rev = [r for r in bam if r.is_reverse]
```

## Evaluation signals

- Reopen each output BAM with pysam and confirm every contained read carries the expected strand flag (forward or reverse) for its partition
- Reopen each output BAM and confirm every read carries the expected read-group (RG) tag for its bucket
- Sum of read counts across all strand/RG partitions equals the total read count of the input demo BAM (no quantitative threshold is reported; exact-sum consistency is the acceptance criterion)
- Output BAM headers preserve the original header, retaining sort order and references

## Limitations

- Acceptance is an exact-sum read-count consistency check; the source text reports no quantitative threshold for partition quality
- Interpretation depends on correct encoding of the FLAG reverse-strand bit and RG tags in the input BAM; malformed or missing RG tags will misroute reads into per-read-group buckets
- No software version numbers are stated for the tools involved in the source documentation

## Evidence

- [intro] BAM format, and canonical annotation in GFF or GTF format.: "peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format, and canonical annotation in GFF or GTF format."
- [readme] BAM header reuse in the demo verification run: "This uses a small demo set of input files contained in the repository: <a href="https://github.com/haessar/peaks2utr/blob/master/peaks2utr/demo/Tb927_01_v5.1.gff" target="_blank""
- [intro] BAM coverage peaks feeding the partitioned reads: "peaks2utr uses MACS (https://pypi.org/project/MACS3/) to call broad "peaks" of significant read coverage in the BAM file"
- [methods] Tool invocation context for the BAM inputs: "peaks2utr <GFF_IN> <BAM_IN> [options]"
