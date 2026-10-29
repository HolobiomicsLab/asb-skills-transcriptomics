---
name: read-barcode-assignment-sc-rnaseq
description: Use when you have FASTQ or FASTA single-cell reads (e.g. 10x Chromium
  or Visium libraries, noisy long-read scRNA-seq) and need to identify cellular barcodes
  and UMIs and assign reads to them.
when_to_use_negative:
- The reads are not barcoded single-cell data and you need a standard exact grep-like
  search without edit-distance tolerance (Flexiplex is built for error-tolerant barcode/sequence
  matching).
- Barcodes have already been extracted and corrected and the input is a count matrix
  or cell-by-gene table rather than raw reads.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: flexiplex
  role: Demultiplexes reads and assigns them to cellular barcodes, in known-barcode
    mode (with chemistry presets such as 10x3v3) or discovery mode (-f 0), using Levenshtein-distance
    error tolerance
  repo: https://github.com/DavidsonGroup/flexiplex
- name: flexiplex-filter
  role: Filters discovered barcodes against the 10x whitelist (3M-february-2018.txt)
    and removes empty droplets using an inflection point method
  repo: https://github.com/DavidsonGroup/flexiplex
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:5802f03178ced8026b7ed37b2c4143e51f654c508b6ec2dbe56d247361bccc88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - flexiplex
  - flexiplex-filter
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# read-barcode-assignment-sc-rnaseq

## Summary

Assign single-cell reads (short or long) to cellular barcodes using Flexiplex, either in known-barcode mode with chemistry presets (e.g. 10x3v3) or in barcode discovery mode followed by whitelist filtering and empty-droplet removal via flexiplex-filter.

## When to use

Use when you have FASTQ or FASTA single-cell reads (e.g. 10x Chromium or Visium libraries, noisy long-read scRNA-seq) and need to identify cellular barcodes and UMIs and assign reads to them.

## When NOT to use

- The reads are not barcoded single-cell data and you need a standard exact grep-like search without edit-distance tolerance (Flexiplex is built for error-tolerant barcode/sequence matching).
- Barcodes have already been extracted and corrected and the input is a count matrix or cell-by-gene table rather than raw reads.

## Inputs

- FASTQ or FASTA reads (short or single-cell long reads, e.g. ONT scRNA-seq)
- barcode_list.txt (known barcodes, for known-barcode mode)
- 3M-february-2018.txt (10x barcode whitelist, for discovery mode)
- flexiplex_barcodes_counts.txt (flexiplex discovery output, input to flexiplex-filter)

## Outputs

- new_reads.fastq (demultiplexed reads with read-barcode assignment)
- flexiplex_barcodes_counts.txt (discovered barcodes with read counts)
- filtered.txt (whitelist-matching barcodes after empty-droplet removal)

## How to apply

If barcodes are known, run flexiplex with a chemistry preset and a barcode list (e.g. `flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq`), which reports read-barcode assignment and writes demultiplexed reads. If barcodes are unknown (discovery mode), run `flexiplex -f 0 reads.fastq` so barcodes are searched using the flanking sequence and read-barcode assignments with counts are written out; then run `flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt` to restrict discovered barcodes to the 10x whitelist and remove empty droplets using an inflection point method. Matching relies on Levenshtein distance, allowing imperfect (error-tolerant) barcode matches, which is key for noisy ONT long reads. Optionally re-run flexiplex with the filtered barcode list to assign reads to the retained barcodes.

## Related tools

- **flexiplex** (Demultiplexes reads and assigns them to cellular barcodes, in known-barcode mode (with chemistry presets such as 10x3v3) or discovery mode (-f 0), using Levenshtein-distance error tolerance) — https://github.com/DavidsonGroup/flexiplex
- **flexiplex-filter** (Filters discovered barcodes against the 10x whitelist (3M-february-2018.txt) and removes empty droplets using an inflection point method) — https://github.com/DavidsonGroup/flexiplex

## Examples

```
flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- flexiplex_barcodes_counts.txt exists as the input to flexiplex-filter
- filtered.txt contains only whitelist-matching barcodes after empty-droplet removal
- new_reads.fastq exists after assignment in known-barcode mode
- A large fraction of assigned reads show concordant cell-line assignment; in the scmixology 2 benchmark Flexiplex reported the highest number of reads with concordant cell-line assignment and <5% of reads with discordant cell lines
- Assigned barcode counts are consistent with the expected cell number (e.g. the 248-cell whitelist scenario in the scmixology 2 benchmark)

## Limitations

- Matching is based on Levenshtein distance, so mismatch patterns are limited to edit-distance-like differences; highly divergent barcodes may be missed.
- In discovery mode, empty droplets must be removed downstream via the inflection point method; the discovered barcode set alone is not the final cell list.
- Discovery mode requires the flanking sequence of the barcode region to be known (only flanking sequences known, not the barcodes themselves).

## Evidence

- [methods] known-barcode mode command: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [methods] filter command: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [methods] whitelist/inflection filtering capability: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [abstract] error tolerance mechanism: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [intro] input/output scope: "Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment."
- [results] benchmark finding: [evidence span withheld — anchor status: not_found]
- [intro] discovery-mode requirement: "discovery mode—when only the sequence which flanks the region of interest is known."
