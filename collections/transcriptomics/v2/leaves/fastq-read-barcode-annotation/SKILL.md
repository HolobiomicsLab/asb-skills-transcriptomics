---
name: fastq-read-barcode-annotation
description: Use when FASTQ reads must be filtered for reads containing a known sequence/variant
  (with mismatches tolerated) and then annotated with cell-barcode and UMI assignments
  in a single command-line pipeline.
when_to_use_negative:
- Only a simple grep-like sequence search is needed with no downstream barcode demultiplexing
  — a single Flexiplex instance suffices.
- Barcodes are unknown and only flanking sequences are known — use Flexiplex discovery
  mode instead of known-sequence chaining.
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Flexiplex
  role: 'Both pipeline stages: first instance performs the error-tolerant variant
    search (grep-like mode, up to 3 mismatches), second instance demultiplexes 10x
    cell barcodes and UMIs; instances are connected via a Unix pipe through standard
    IO.'
  repo: https://github.com/DavidsonGroup/flexiplex
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:6f8ea7a0f758451474526566aef184e6ad431482d1045a248acc88467ed12483
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - Flexiplex
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# fastq-read-barcode-annotation

## Summary

Chain two piped Flexiplex instances to first perform an error-tolerant (Levenshtein-based) grep-like search of FASTQ reads for a variant of interest, then demultiplex cell barcodes/UMIs from the matched reads using a 10x Chromium v3 preset and a known barcode whitelist.

## When to use

Use when FASTQ reads must be filtered for reads containing a known sequence/variant (with mismatches tolerated) and then annotated with cell-barcode and UMI assignments in a single command-line pipeline.

## When NOT to use

- Only a simple grep-like sequence search is needed with no downstream barcode demultiplexing — a single Flexiplex instance suffices.
- Barcodes are unknown and only flanking sequences are known — use Flexiplex discovery mode instead of known-sequence chaining.

## Inputs

- Input reads file (FASTQ or FASTA)
- Variant query sequence (e.g. rs878887783 variant query)
- Known barcode whitelist file (barcode_list.txt)

## Outputs

- new_reads.fastq with cell-barcode and UMI annotations in read headers
- Barcode counts report (flexiplex_barcodes_counts.txt)

## How to apply

Run the first Flexiplex instance in known-sequence mode with the variant query (e.g. 'flexiplex -x <query_sequence> -d grep -f 3 reads.fastq'), which performs an error-tolerant search allowing up to 3 mismatches based on Levenshtein distance. Because Flexiplex reads and writes standard IO, pipe the matched reads directly into a second Flexiplex instance using the 10x Chromium v3 preset ('-d 10x3v3') with a known barcode whitelist ('-k barcode_list.txt') to demultiplex cellular barcodes from each variant-containing read. Redirect the second instance's stdout to new_reads.fastq, in which read headers carry the cell-barcode and UMI annotation fields, and collect the barcode counts report. Validate that new_reads.fastq exists as valid FASTQ and that headers contain the expected barcode/UMI fields.

## Related tools

- **Flexiplex** (Both pipeline stages: first instance performs the error-tolerant variant search (grep-like mode, up to 3 mismatches), second instance demultiplexes 10x cell barcodes and UMIs; instances are connected via a Unix pipe through standard IO.) — https://github.com/DavidsonGroup/flexiplex

## Examples

```
flexiplex -x <query_sequence> -d grep -f 3 reads.fastq | flexiplex -d 10x3v3 -k barcode_list.txt > new_reads.fastq
```

## Evaluation signals

- new_reads.fastq exists and is valid FASTQ (4-line record structure).
- Read headers in new_reads.fastq contain the expected cell-barcode and UMI annotation fields from the demultiplexing stage.
- All reads in new_reads.fastq contain the variant query sequence within the allowed mismatch count (Levenshtein distance ≤ 3).
- Barcode counts report is produced and barcode assignments are consistent with the supplied whitelist (barcode_list.txt).

## Limitations

- Match tolerance is capped by the Levenshtein-distance edit budget (e.g. -f 3 allows up to 3 mismatches); higher error rates in noisy long reads may require tuning this threshold.
- The whitelist preset assumes 10x Chromium chemistry (10x3v3); other chemistries need a different Flexiplex preset.

## Evidence

- [methods] Flexiplex can perform an error-tolerant grep-like search for a single sequence.: "Flexiplex can perform an error-tolerant grep-like search for a single sequence."
- [readme] Standard IO piping for complex applications: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [abstract] Levenshtein-based imperfect matching: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [intro] Demultiplexing scope of the tool: "Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment."
