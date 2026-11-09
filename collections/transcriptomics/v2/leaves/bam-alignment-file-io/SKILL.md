---
name: bam-alignment-file-io
description: Use when a pipeline must preprocess aligned reads in BAM format by opening
  the file, inspecting per-read FLAG and RG tags, and writing partitioned BAMs with
  a preserved header.
when_to_use_negative:
- Input is not aligned reads in BAM format (peaks2utr expects BAM; GFF/GTF gene models
  are handled by separate logic)
- You need to annotate UTRs directly without splitting — the peaks2utr CLI already
  performs its own internal partitioning
- Reads lack RG tags or the read lacks a reverse-strand FLAG bit, making strand/read-group
  routing undefined
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: pysam
  role: Python wrapper for htslib and SAMtools used to open, iterate, split and write
    the BAM partitions
  repo: https://github.com/haessar/peaks2utr
- name: peaks2utr
  role: 'Downstream consumer: the strand/read-group split BAMs feed its SPAT algorithm
    and MACS peak calling'
  repo: https://github.com/haessar/peaks2utr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad112
    title: 'peaks2utr: a robust Python tool for the annotation of 3′ UTRs'
  source_content_hash: sha256:f7a01a08a3906beee4618b2b3b2ab1ee03454e930ad6ccac25c1330e3a1f21a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haessar/peaks2utr
  tools:
  - peaks2utr
  - pysam
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bam-alignment-file-io

## Summary

Read, split, and write BAM alignment files with pysam (Python wrapper for htslib/SAMtools), as peaks2utr does when partitioning an input BAM by strand and read group before peak calling.

## When to use

Use when a pipeline must preprocess aligned reads in BAM format by opening the file, inspecting per-read FLAG and RG tags, and writing partitioned BAMs with a preserved header.

## When NOT to use

- Input is not aligned reads in BAM format (peaks2utr expects BAM; GFF/GTF gene models are handled by separate logic)
- You need to annotate UTRs directly without splitting — the peaks2utr CLI already performs its own internal partitioning
- Reads lack RG tags or the read lacks a reverse-strand FLAG bit, making strand/read-group routing undefined

## Inputs

- Aligned reads in BAM format (e.g. Tb927_01_v5.1.slice.bam from the peaks2utr demo)
- BAM header reused for output files (sort order and references)

## Outputs

- Forward-strand and reverse-strand BAM files
- Per-read-group BAM files within each strand partition
- Per-read FLAG and RG-tag verification plus read-count totals across partitions

## How to apply

Open the BAM (e.g. the demo Tb927_01_v5.1.slice.bam) with pysam.AlignmentFile, keeping the header for reuse. Iterate over aligned reads and route each read into a forward- or reverse-strand pile based on the SAM FLAG field's reverse-strand bit. Within each strand pile, subdivide reads by their read-group (RG) tag into per-read-group buckets. Write each bucket as its own BAM file with pysam, copying the original header so sort order and references are retained. Rationale: peaks2utr partitions the BAM this way before applying the SPAT algorithm and MACS peak calling, so downstream coverage analysis operates on strand- and read-group-homogeneous read sets.

## Related tools

- **pysam** (Python wrapper for htslib and SAMtools used to open, iterate, split and write the BAM partitions) — https://github.com/haessar/peaks2utr
- **peaks2utr** (Downstream consumer: the strand/read-group split BAMs feed its SPAT algorithm and MACS peak calling) — https://github.com/haessar/peaks2utr

## Examples

```
import pysam; bam = pysam.AlignmentFile('Tb927_01_v5.1.slice.bam', 'rb'); fwd = pysam.AlignmentFile('fwd.bam', 'wb', header=bam.header)
```

## Evaluation signals

- Reopen every output BAM with pysam and confirm each contained read carries the expected strand flag
- Confirm each read carries the expected read-group (RG) tag in its partition
- Confirm the summed read counts across all partitions equal the total read count of the input demo BAM (exact-sum consistency check; no threshold given in source)
- Check that output BAM headers preserve sort order and references from the original file

## Limitations

- No quantitative threshold is reported in the source text; acceptance is the exact-sum read-count consistency between input and partitioned outputs
- The article states no software version numbers for pysam, so reproducibility of the split step depends on unspecified pysam/htslib versions

## Evidence

- [results] preprocessing splits BAM: [evidence span withheld — anchor status: not_found]
- [results] pysam wrapper: [evidence span withheld — anchor status: not_found]
- [intro] peaks2utr BAM input: "annotates 3' untranslated regions (UTR) for a given set of aligned sequencing reads in BAM format"
- [readme] demo BAM input: "This uses a small demo set of input files contained in the repository:"
