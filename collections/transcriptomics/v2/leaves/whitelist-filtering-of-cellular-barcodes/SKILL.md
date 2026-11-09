---
name: whitelist-filtering-of-cellular-barcodes
description: Use when you have noisy long-read single-cell RNA-seq data with discovered
  cellular barcodes and a short-read-derived whitelist, and need to filter against
  the whitelist and remove empty droplets.
when_to_use_negative:
- When barcodes are already known and the task is read demultiplexing/assignment rather
  than discovery and whitelist filtering (use known-sequence mode with a preset such
  as -d 10x3v3).
- When no short-read-derived whitelist or ground-truth barcode set exists to filter
  against.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: flexiplex-filter
  role: Filters discovered barcodes against a whitelist and removes empty droplets
    using the inflection point method
  repo: https://github.com/DavidsonGroup/flexiplex
- name: flexiplex
  role: Discovers cellular barcodes from noisy long reads in discovery mode ('flexiplex
    -f 0'), producing the barcode counts file consumed by flexiplex-filter
  repo: https://github.com/DavidsonGroup/flexiplex
- name: conda
  role: Installation channel for Flexiplex via bioconda
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:e5030cccdcb69eaa88c97eaaddae054a965338f83c19a160235a6c2d1c063d90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - Flexiplex
  - conda
  - flexiplex
  - flexiplex-filter
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# whitelist-filtering-of-cellular-barcodes

## Summary

Filter discovered or demultiplexed cellular barcodes against a short-read-derived whitelist and remove empty droplets using the inflection point method with flexiplex-filter, producing validated barcode frequency lists for benchmarking barcode discovery.

## When to use

Use when you have noisy long-read single-cell RNA-seq data with discovered cellular barcodes and a short-read-derived whitelist, and need to filter against the whitelist and remove empty droplets.

## When NOT to use

- When barcodes are already known and the task is read demultiplexing/assignment rather than discovery and whitelist filtering (use known-sequence mode with a preset such as -d 10x3v3).
- When no short-read-derived whitelist or ground-truth barcode set exists to filter against.

## Inputs

- flexiplex_barcodes_counts.txt (barcode counts output from flexiplex discovery mode)
- Short-read-derived cellular barcode whitelist (e.g. 3M-february-2018.txt or replicate-specific whitelist used as withheld truth)
- Noisy long-read single-cell RNA-seq FASTQ reads (scmixology 2, GridION Q20/GridION/PromethION hiPSC replicates)

## Outputs

- filtered.txt — whitelist-filtered barcode list
- Barcode frequency lists after inflection-point empty-droplet removal
- benchmark_table.csv comparing sensitivity, specificity, and runtime across tools

## How to apply

Run flexiplex in discovery mode ('flexiplex -f 0 reads.fastq' with four threads and default settings) to discover cellular barcodes from noisy long reads, which produces flexiplex_barcodes_counts.txt. Then run flexiplex-filter with the short-read-derived barcode whitelist ('flexiplex-filter --whitelist <whitelist.txt> --outfile filtered.txt flexiplex_barcodes_counts.txt') to retain barcodes present in the withheld truth and remove empty droplets via the inflection point method. The rationale is that short-read-derived whitelists serve as ground truth, so filtering enables sensitivity and specificity evaluation of the discovered barcode set. Assemble the resulting barcode frequency lists into a benchmark table comparing sensitivity, specificity, and runtime across tools such as BLAZE, scTagger, and wf-single-cell.

## Related tools

- **flexiplex-filter** (Filters discovered barcodes against a whitelist and removes empty droplets using the inflection point method) — https://github.com/DavidsonGroup/flexiplex
- **flexiplex** (Discovers cellular barcodes from noisy long reads in discovery mode ('flexiplex -f 0'), producing the barcode counts file consumed by flexiplex-filter) — https://github.com/DavidsonGroup/flexiplex
- **conda** (Installation channel for Flexiplex via bioconda)

## Examples

```
flexiplex-filter --whitelist whitelist.txt --outfile filtered.txt flexiplex_barcodes_counts.txt
```

## Evaluation signals

- Sensitivity and specificity of the filtered barcode set against the withheld short-read-derived whitelist are similar to BLAZE and scTagger.
- flexiplex-filter produces filtered.txt containing barcodes present in the whitelist with empty droplets removed at the inflection point.
- Flexiplex discovery-mode runtime falls in roughly 5%–16% of BLAZE's runtime and 25%–42% of scTagger's runtime (Fig. 1H).
- benchmark_table.csv assembles sensitivity, specificity, and wall-clock time/memory for each comparator tool on the same replicates.

## Limitations

- Filtering and inflection-point droplet removal depend on the availability and quality of a short-read-derived whitelist used as withheld truth.
- Comparator workflow wf-single-cell uses an order of magnitude more compute resources (minimum 32 CPUs, recommended 64 CPUs/256GB memory) for the same task.

## Evidence

- [methods] Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method.: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [methods] flexiplex-filter command example: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [results] Runtime comparison finding: [evidence span withheld — anchor status: not_found]
- [readme] README usage example for barcode discovery: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
