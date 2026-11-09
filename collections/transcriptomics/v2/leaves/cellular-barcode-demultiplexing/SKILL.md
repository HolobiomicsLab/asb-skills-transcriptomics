---
name: cellular-barcode-demultiplexing
description: Use when input is single-cell FASTQ/FASTA reads with embedded cellular
  barcodes/UMIs (10x Chromium, Visium) and the goal is read-to-cell-barcode assignment,
  optionally with barcode discovery or whitelist filtering.
when_to_use_negative:
- Input is already a cell x gene count matrix or BAM with cell-barcode tags — demultiplexing
  has already been performed
- The analysis requires full single-cell workflow steps (adapter trimming, read mapping,
  gene/transcript assignment) beyond barcode/UMI extraction — use a workflow such
  as epi2me-labs/wf-single-cell instead
- Exact-match-only pattern searching without edit-distance tolerance is required and
  Levenshtein-based fuzzy matching is undesirable
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: Flexiplex
  role: 'Primary demultiplexer: assigns reads to cellular barcodes using Levenshtein-distance
    matching, presets (10x3v3), whitelist filtering, and inflection-point empty-droplet
    removal'
  repo: https://github.com/DavidsonGroup/flexiplex
- name: flexiplex-filter
  role: Post-processing companion that filters discovered barcodes against a whitelist
    (e.g. 3M-february-2018.txt) from the barcode counts report
  repo: https://github.com/DavidsonGroup/flexiplex
- name: conda
  role: Installation channel for Flexiplex from bioconda
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:b0c73aaee72e8f747e37c29f0479847211e7798a80422924d96f59373b1cbf35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - FLAMES
  - Flexiplex
  - conda
  - flexiplex-filter
  - scTagger
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# cellular-barcode-demultiplexing

## Summary

Demultiplex single-cell reads (FASTQ/FASTA) by assigning cellular barcodes and UMIs using Flexiplex, a Levenshtein-distance-based, multithreaded demultiplexer that supports known-barcode whitelists, chemistry presets (e.g. 10x Chromium v3), and error-tolerant matching.

## When to use

Use when input is single-cell FASTQ/FASTA reads with embedded cellular barcodes/UMIs (10x Chromium, Visium) and the goal is read-to-cell-barcode assignment, optionally with barcode discovery or whitelist filtering.

## When NOT to use

- Input is already a cell x gene count matrix or BAM with cell-barcode tags — demultiplexing has already been performed
- The analysis requires full single-cell workflow steps (adapter trimming, read mapping, gene/transcript assignment) beyond barcode/UMI extraction — use a workflow such as epi2me-labs/wf-single-cell instead
- Exact-match-only pattern searching without edit-distance tolerance is required and Levenshtein-based fuzzy matching is undesirable

## Inputs

- FASTQ or FASTA reads file containing single-cell reads with embedded cellular barcodes and UMIs
- Known cellular barcode whitelist file (e.g. barcode_list.txt, 3M-february-2018.txt for 10x)
- Flexiplex chemistry preset identifier (e.g. -d 10x3v3) or flanking sequences for discovery mode

## Outputs

- new_reads.fastq — matching reads with cell-barcode and UMI assignments recorded in read headers
- flexiplex_barcodes_counts.txt — per-barcode read counts report
- Read-barcode assignment stats (correct assignments, false discovery rate) for benchmarking

## How to apply

Run Flexiplex in known-barcode mode with a preset matching the library chemistry, e.g. `flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq`, which uses the 10x Chromium v3 preset structure and matches observed barcodes against the whitelist allowing imperfect (Levenshtein distance) matches; set the maximum edit distance (0–3 tested in benchmarking against scTagger and FLAMES). In discovery mode (only flanking sequences known), use `flexiplex -f 0` to search for barcodes, then filter against a whitelist and remove empty droplets using the inflection point method via `flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt`. Because Flexiplex reads and writes standard IO, instances can be piped together for combined search-then-demultiplex workflows (e.g. variant grep followed by 10x3v3 barcode assignment). Install via bioconda (`conda install -c bioconda -c conda-forge flexiplex`), precompiled GitHub releases, or `make` with gcc.

## Related tools

- **Flexiplex** (Primary demultiplexer: assigns reads to cellular barcodes using Levenshtein-distance matching, presets (10x3v3), whitelist filtering, and inflection-point empty-droplet removal) — https://github.com/DavidsonGroup/flexiplex
- **flexiplex-filter** (Post-processing companion that filters discovered barcodes against a whitelist (e.g. 3M-february-2018.txt) from the barcode counts report) — https://github.com/DavidsonGroup/flexiplex
- **conda** (Installation channel for Flexiplex from bioconda)

## Examples

```
flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- Output new_reads.fastq is valid FASTQ and read headers contain cell-barcode and UMI annotation fields
- On benchmark data with true barcodes, correct-assignment counts exceed scTagger and FLAMES and FDR is lowest across max edit distances 0–3 (mirrors Fig. 1D)
- Per-edit-distance runs (0, 1, 2, 3) produce assignments whose false discovery rate (false assignments / total assignments) is tabulated and monotonically consistent
- flexiplex_barcodes_counts.txt is produced and flexiplex-filter output (filtered.txt) contains only whitelisted barcodes after inflection-point empty-droplet removal

## Limitations

- Matching is based on Levenshtein distance, so maximum edit distance must be chosen per run (0–3 benchmarked); higher distances increase false assignments
- Benchmark advantage over scTagger and FLAMES was demonstrated on a simulated best-case scenario (5 million reads, 5000 cells) with true barcodes provided
- Demultiplexing alone does not include adapter identification, mapping, or gene/transcript assignment, which require a full workflow (e.g. epi2me-labs/wf-single-cell)

## Evidence

- [methods] Known-barcode mode invocation: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [methods] Whitelist filtering workflow: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [abstract] Levenshtein-based imperfect matching: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [readme] Readme usage for barcode assignment: "Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment."
- [readme] Discovery-mode filtering capability: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
