---
name: unix-pipe-chaining-of-bioinformatics-tools
description: Use when a read-level analysis requires two or more sequential operations
  (e.g. selecting reads containing a variant, then demultiplexing cell barcodes) and
  the tools involved can read from stdin and write to stdout, so intermediate FASTQ
  files can be avoided.
when_to_use_negative:
- The task needs only a single pass (e.g. one demultiplexing step) — a single Flexiplex
  invocation 'flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq'
  suffices without chaining.
- The tool in the chain cannot read from stdin or write to stdout, so no stream connection
  is possible and intermediate files are required.
- Barcode sequences are unknown and only flanking sequences are known — discovery
  mode rather than known-sequence chaining is needed.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: Flexiplex
  role: Stream-capable demultiplexer and error-tolerant search tool; multiple instances
    are chained via pipes (first as grep-like variant search with '-x ... -d grep
    -f 3', second as 10x3v3 barcode demultiplexer with '-d 10x3v3 -k barcode_list.txt')
  repo: https://davidsongroup.github.io/flexiplex/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:269361bfd6cd7e01a424e9e26c0977c3c9ac4887ca4f962f602f9c8310c325c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Flexiplex
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# unix-pipe-chaining-of-bioinformatics-tools

## Summary

Chains multiple Flexiplex instances (or similar stream-capable bioinformatics tools) on the command line by piping standard output to standard input, e.g. an error-tolerant grep-like variant search followed by 10x Chromium v3 cell-barcode demultiplexing in a single pipeline.

## When to use

Use when a read-level analysis requires two or more sequential operations (e.g. selecting reads containing a variant, then demultiplexing cell barcodes) and the tools involved can read from stdin and write to stdout, so intermediate FASTQ files can be avoided.

## When NOT to use

- The task needs only a single pass (e.g. one demultiplexing step) — a single Flexiplex invocation 'flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq' suffices without chaining.
- The tool in the chain cannot read from stdin or write to stdout, so no stream connection is possible and intermediate files are required.
- Barcode sequences are unknown and only flanking sequences are known — discovery mode rather than known-sequence chaining is needed.

## Inputs

- FASTQ (or FASTA) reads file, e.g. reads.fastq
- Known query sequence (e.g. variant of interest such as rs878887783 query)
- Barcode whitelist file, e.g. barcode_list.txt

## Outputs

- Filtered/demultiplexed FASTQ (e.g. new_reads.fastq) with cell-barcode and UMI assignments in read headers
- Barcode counts report from the demultiplexing stage

## How to apply

Confirm both tools support standard IO; Flexiplex can read and write streams, so its output can be piped directly into another instance. Run the first stage as a grep-like known-sequence search, e.g. 'flexiplex -x <query_sequence> -d grep -f 3 reads.fastq', which performs an error-tolerant search allowing up to 3 mismatches based on Levenshtein distance. Pipe the matched reads into a second Flexiplex instance using the 10x Chromium v3 preset ('-d 10x3v3') with a barcode whitelist ('-k barcode_list.txt') to demultiplex cell barcodes and UMIs. Redirect the final stdout to an output FASTQ; read headers carry the cell-barcode/UMI assignments from the demultiplexing stage. Validate by checking the output is valid FASTQ and that headers contain the expected barcode/UMI annotation fields.

## Related tools

- **Flexiplex** (Stream-capable demultiplexer and error-tolerant search tool; multiple instances are chained via pipes (first as grep-like variant search with '-x ... -d grep -f 3', second as 10x3v3 barcode demultiplexer with '-d 10x3v3 -k barcode_list.txt')) — https://davidsongroup.github.io/flexiplex/

## Examples

```
flexiplex -x <query_sequence> -d grep -f 3 reads.fastq | flexiplex -d 10x3v3 -k barcode_list.txt > new_reads.fastq
```

## Evaluation signals

- new_reads.fastq exists and parses as valid FASTQ (equal number of lines in header/sequence/plus/quality records).
- Read headers contain the expected cell-barcode and UMI annotation fields produced by the demultiplexing stage.
- The number of reads in the output is less than or equal to the input, consistent with the variant-selection stage removing non-matching reads.
- Matched reads satisfy the specified error tolerance (up to 3 mismatches by Levenshtein distance to the query sequence).
- The barcode counts report contains barcodes drawn from the supplied whitelist file.

## Limitations

- Flexiplex matching is based on Levenshtein distance, so the -f mismatch threshold controls the trade-off between sensitivity and specificity; inexact matches within that distance are accepted.
- Chaining requires tools that support standard IO; tools without stdin/stdout support cannot be piped this way.
- Discovery mode (only flanking sequence known) is a different mode from the known-sequence chaining shown here.

## Evidence

- [other] Because Flexiplex can read and write to standard IO, its output can be piped to another Flexiplex instance: [evidence span withheld — anchor status: not_found]
- [methods] Flexiplex can perform an error-tolerant grep-like search for a single sequence.: "Flexiplex can perform an error-tolerant grep-like search for a single sequence."
- [methods] flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [abstract] which is based on the Levenshtein distance and thus allows imperfect matches: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [readme] Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment.: "Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment."
