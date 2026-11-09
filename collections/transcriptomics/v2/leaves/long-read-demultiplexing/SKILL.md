---
name: long-read-demultiplexing
description: Use when noisy ONT long-read FASTQ/FASTA single-cell data needs cell-barcode
  demultiplexing or barcode discovery, e.g. 10x Chromium libraries where barcodes
  contain sequencing errors and a whitelist may or may not be known.
when_to_use_negative:
- When reads already carry trusted cell-barcode tags and the task is downstream counting/UMI
  collapse (e.g. building gene x cell matrices, as in epi2me-labs/wf-single-cell),
  not barcode assignment
- When an exact, error-free pattern search is needed and fuzzy Levenshtein matching
  would introduce false positives; Flexiplex's error tolerance is designed for noisy
  barcodes
- When compute memory is severely constrained but wall-clock time is not, and exact-match
  approaches like scTagger's lower-memory profile are acceptable
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: Flexiplex
  role: Fast, multithreaded Levenshtein-distance demultiplexer/search tool used for
    barcode discovery and read-barcode assignment
  repo: https://davidsongroup.github.io/flexiplex/
- name: flexiplex-filter
  role: Filters discovered barcodes against a whitelist (3M-february-2018.txt) and
    removes empty droplets using an inflection point method
  repo: https://github.com/DavidsonGroup/flexiplex
- name: conda
  role: 'Installation channel: conda install -c bioconda -c conda-forge flexiplex'
- name: scTagger
  role: Comparison tool in benchmarks; faster (1.5-5 h) but >100 GB RAM
- name: FLAMES
  role: Comparison tool in benchmarks; failed to finish within 48 h wall-clock cutoff
  repo: https://github.com/LuyiTian/FLAMES
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:f7052d445abbd6d1a27344a2f2878004bdc32d12d6425f8f5dbd691557e3b390
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FLAMES
  - Flexiplex
  - conda
  - flexiplex-filter
  - scTagger
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# long-read-demultiplexing

## Summary

Error-tolerant demultiplexing of cellular barcodes from noisy long-read (ONT) single-cell RNA-seq data using Flexiplex, including barcode discovery, whitelist filtering, and empty-droplet removal, benchmarked against scTagger and FLAMES.

## When to use

Use when noisy ONT long-read FASTQ/FASTA single-cell data needs cell-barcode demultiplexing or barcode discovery, e.g. 10x Chromium libraries where barcodes contain sequencing errors and a whitelist may or may not be known.

## When NOT to use

- When reads already carry trusted cell-barcode tags and the task is downstream counting/UMI collapse (e.g. building gene x cell matrices, as in epi2me-labs/wf-single-cell), not barcode assignment
- When an exact, error-free pattern search is needed and fuzzy Levenshtein matching would introduce false positives; Flexiplex's error tolerance is designed for noisy barcodes
- When compute memory is severely constrained but wall-clock time is not, and exact-match approaches like scTagger's lower-memory profile are acceptable

## Inputs

- Long-read single-cell FASTQ (e.g. ~25M ONT scmixology 2 reads or 61M PromethION hiPSC reads)
- FASTA or FASTQ reads (short or long)
- Known barcode list / whitelist (e.g. 1022 true + 10000 decoy barcodes, or 3M-february-2018.txt)
- Chemistry preset identifier (e.g. 10x3v3, 10x Chromium, Visium)

## Outputs

- new_reads.fastq with reads assigned to cellular barcodes
- flexiplex_barcodes_counts.txt (discovered barcodes with read assignments and counts)
- filtered.txt (whitelist-matching barcodes after empty-droplet removal)
- benchmark_results.csv (correct-read counts, runtime, peak RAM per tool)

## How to apply

Run flexiplex in barcode discovery mode on the long-read FASTQ with `flexiplex -f 0 reads.fastq` to obtain read-barcode assignments and barcode counts (flexiplex_barcodes_counts.txt), exploiting Levenshtein-distance-based imperfect matching for noisy reads. Filter discovered barcodes against a whitelist (e.g. 3M-february-2018.txt for 10x) and remove empty droplets via the inflection point method using flexiplex-filter. Optionally re-run flexiplex in known-barcode mode with a chemistry preset, e.g. `flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq`, to assign reads to the filtered barcodes. Benchmarking context: Flexiplex completed 61M-read PromethION runs in 7-14 h on 16 threads with <1 GB RAM, whereas scTagger needed >100 GB RAM and FLAMES failed to finish within 48 h.

## Related tools

- **Flexiplex** (Fast, multithreaded Levenshtein-distance demultiplexer/search tool used for barcode discovery and read-barcode assignment) — https://davidsongroup.github.io/flexiplex/
- **flexiplex-filter** (Filters discovered barcodes against a whitelist (3M-february-2018.txt) and removes empty droplets using an inflection point method) — https://github.com/DavidsonGroup/flexiplex
- **conda** (Installation channel: conda install -c bioconda -c conda-forge flexiplex)
- **scTagger** (Comparison tool in benchmarks; faster (1.5-5 h) but >100 GB RAM)
- **FLAMES** (Comparison tool in benchmarks; failed to finish within 48 h wall-clock cutoff) — https://github.com/LuyiTian/FLAMES

## Examples

```
flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- flexiplex_barcodes_counts.txt exists and serves as input to flexiplex-filter
- filtered.txt contains only whitelist-matching barcodes after empty-droplet removal (file_exists checks on new_reads.fastq, flexiplex_barcodes_counts.txt, filtered.txt)
- Cell-line concordance: highest number of concordant reads with <5% discordant cell lines on scmixology 2 (248-cell whitelist)
- PromethION benchmark: Flexiplex finishes 61M reads in 7-14 h on 16 threads with <1 GB peak RAM
- Validation on hiPSC truth set: Flexiplex yields the highest number of correctly demultiplexed reads against the 1022 true barcodes

## Limitations

- Accuracy depends on edit-distance settings; discovery mode uses `-f 0` and known-barcode mode examples use preset `10x3v3` with a supplied barcode list, so wrong preset/edit distance degrades assignment
- FLAMES failed to complete within 48 h on the 61M-read PromethION dataset, and scTagger required >100 GB RAM, making head-to-head benchmarking expensive
- The pre-resolved GitHub metadata lists epi2me-labs/wf-single-cell with license=NOASSERTION; wf-single-cell is a separate Nextflow workflow (port of Sockeye), not part of Flexiplex itself

## Evidence

- [abstract] Levenshtein-distance matching allows imperfect barcodes: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [methods] Discovery mode CLI: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [methods] Whitelist filtering and empty-droplet removal: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [methods] Conda installation command: "conda install -c bioconda -c conda-forge flexiplex"
- [readme] README: workflow capabilities: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
