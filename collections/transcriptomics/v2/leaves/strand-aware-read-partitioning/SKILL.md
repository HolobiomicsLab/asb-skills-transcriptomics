---
name: strand-aware-read-partitioning
description: Use when a pipeline consumes a BAM of aligned reads (e.g. 10x Chromium
  output) and downstream analysis, such as truncation-point pileups or per-strand
  peak calling, requires reads to be separated by strand and read group before processing.
when_to_use_negative:
- The input is not a BAM of aligned reads but an already-processed feature table,
  GFF/GTF annotation, or normalized coverage track
- The downstream tool consumes a single unsplit BAM and performs its own strand or
  read-group handling internally
- The BAM lacks read-group (RG) tags or strand information, so no meaningful RG-level
  or strand-level partition can be made
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: pysam
  role: Python wrapper for htslib/SAMtools used to open the input BAM, read the FLAG
    reverse-strand bit and RG tags, and write each strand/read-group bucket as a new
    BAM with the original header
  repo: https://github.com/haessar/peaks2utr
- name: peaks2utr
  role: Pipeline whose preprocessing stage performs the strand and read-group splitting
    before SPAT truncation-point analysis and MACS peak calling
  repo: https://github.com/haessar/peaks2utr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:e1510383e4fa6d639ce5e9cf085d4ea27f9d5c746c2b1b4bb3f87f2a7dfb16fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - peaks2utr
  - pysam
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# strand-aware-read-partitioning

## Summary

Partitions an input BAM of aligned sequencing reads into forward- and reverse-strand files and further subdivides each strand pile by read group (RG) tag, using pysam (the Python wrapper for htslib/SAMtools). This preprocessing stage prepares the read data that peaks2utr feeds to the SPAT algorithm and MACS peak calling.

## When to use

Use when a pipeline consumes a BAM of aligned reads (e.g. 10x Chromium output) and downstream analysis, such as truncation-point pileups or per-strand peak calling, requires reads to be separated by strand and read group before processing.

## When NOT to use

- The input is not a BAM of aligned reads but an already-processed feature table, GFF/GTF annotation, or normalized coverage track
- The downstream tool consumes a single unsplit BAM and performs its own strand or read-group handling internally
- The BAM lacks read-group (RG) tags or strand information, so no meaningful RG-level or strand-level partition can be made

## Inputs

- Input BAM file of aligned sequencing reads with a valid SAM header (e.g. Tb927_01_v5.1.slice.bam from the peaks2utr demo set)
- SAM FLAG field reverse-strand bit per read
- Read group (RG) tag per read

## Outputs

- Per-strand, per-read-group BAM files, each written by pysam with the original header copied so sort order and references are retained

## How to apply

Open the input BAM (e.g. the demo Tb927_01_v5.1.slice.bam) with pysam's AlignmentFile, preserving the header for reuse. Iterate over aligned reads and inspect the SAM FLAG field's reverse-strand bit to route each read into either a forward-strand or reverse-strand pile. Within each strand pile, subdivide reads further by their read group (RG) tag into per-read-group buckets, since 10x Chromium-style runs yield multiple read groups. Write each bucket as its own BAM file with pysam, copying the original header so sort order and reference sequences are retained. Rationale: strand-aware splitting keeps truncation points and coverage peaks computed per strand, and RG-level subdivision keeps single-cell read groups from being pooled and diluting per-cell UTR signal.

## Related tools

- **pysam** (Python wrapper for htslib/SAMtools used to open the input BAM, read the FLAG reverse-strand bit and RG tags, and write each strand/read-group bucket as a new BAM with the original header) — https://github.com/haessar/peaks2utr
- **peaks2utr** (Pipeline whose preprocessing stage performs the strand and read-group splitting before SPAT truncation-point analysis and MACS peak calling) — https://github.com/haessar/peaks2utr

## Examples

```
import pysam; bam = pysam.AlignmentFile('Tb927_01_v5.1.slice.bam', 'rb'); header = bam.header.to_dict()
```

## Evaluation signals

- Reopen every output BAM with pysam and confirm each contained read carries the expected strand flag (forward vs reverse-strand bit)
- Confirm each contained read carries the expected read-group (RG) tag for its bucket
- Confirm the summed read counts across all partitions equal the total read count of the input demo BAM — this exact-sum consistency check is the acceptance criterion, with no quantitative threshold reported
- Check each output BAM retains the original header's sort order and reference sequences

## Limitations

- No quantitative threshold is reported in the source text for validating the split; only the exact-sum read-count consistency check is available as an acceptance criterion
- The article notes no software version numbers are stated for the tools in the workflow, so reproducibility depends on pinning pysam/peaks2utr versions externally

## Evidence

- [full_text] The workflow's preprocessing stage partitions each input BAM file into forward- and reverse-strand files and further subdivides them by read group: [evidence span withheld — anchor status: not_found]
- [intro] peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format: "peaks2utr is a Python command-line tool that annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format"
- [intro] This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs.: "This favours BAM files from the likes of 10x Chromium runs, where signal is inherently concentrated at the distal ends of the 3' or 5' UTRs."
