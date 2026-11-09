---
name: single-cell-read-barcode-assignment
description: Use when demultiplexing noisy long-read single-cell RNA-seq data (e.g.
  a 61M-read PromethION FASTQ) against a barcode list of true and decoy barcodes,
  and you need read-barcode assignments plus computational cost comparison across
  tools.
when_to_use_negative:
- Only the sequence flanking the barcode region is known (use Flexiplex discovery
  mode with -f 0 instead of known-barcode mode)
- Input is already a cell-by-gene count matrix or BAM with cell barcodes tagged —
  barcode assignment has already been done (e.g. by wf-single-cell)
- Exact-match barcode search suffices and no error tolerance is needed — a simple
  grep-like tool may be preferable
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Flexiplex
  role: 'Primary demultiplexer: assigns long reads to cellular barcodes using Levenshtein-distance
    matching with 10x long-read presets'
  repo: https://github.com/DavidsonGroup/flexiplex
- name: flexiplex-filter
  role: Filters barcode assignments against a whitelist and removes empty droplets
    using the inflection point method
  repo: https://github.com/DavidsonGroup/flexiplex
- name: conda
  role: Installation channel for Flexiplex (bioconda/conda-forge)
- name: scTagger
  role: Comparison tool in the benchmark; faster (1.5–5 h) but requires over 100 GB
    RAM
- name: FLAMES
  role: Comparison tool in the benchmark; failed to finish within the 48 h wall-clock
    cutoff
  repo: https://github.com/LuyiTian/FLAMES
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:915cec39949bcd2b2e0867febe42185bc59b9915cacefc0383ef46fa7d0b17ef
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
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-read-barcode-assignment

## Summary

Assign single-cell long reads (FASTQ/FASTA) to known cellular barcodes using Flexiplex in known-barcode mode with 10x long-read settings, then filter with flexiplex-filter against a whitelist and remove empty droplets via the inflection point method. This skill also covers benchmarking the result against alternatives (scTagger, FLAMES) for accuracy, wall-clock time, and peak RAM.

## When to use

Use when demultiplexing noisy long-read single-cell RNA-seq data (e.g. a 61M-read PromethION FASTQ) against a barcode list of true and decoy barcodes, and you need read-barcode assignments plus computational cost comparison across tools.

## When NOT to use

- Only the sequence flanking the barcode region is known (use Flexiplex discovery mode with -f 0 instead of known-barcode mode)
- Input is already a cell-by-gene count matrix or BAM with cell barcodes tagged — barcode assignment has already been done (e.g. by wf-single-cell)
- Exact-match barcode search suffices and no error tolerance is needed — a simple grep-like tool may be preferable

## Inputs

- FASTQ (or FASTA) file of single-cell long reads, e.g. the You et al. 2023 hiPSC PromethION FASTQ (61M reads)
- Barcode list file (one barcode per line) containing true cellular barcodes and optional decoy barcodes, passed via -k
- Barcode whitelist file for flexiplex-filter (e.g. 3M-february-2018.txt)
- Truth set of correctly assigned reads (1022 true barcodes) for accuracy scoring

## Outputs

- Demultiplexed FASTQ (new_reads.fastq) with read-barcode assignments
- Barcode counts file from flexiplex-filter (flexiplex_barcodes_counts.txt)
- Filtered read set with empty droplets removed via inflection point method
- benchmark_results.csv with per-tool correct-read counts, wall-clock runtimes, and peak RAM

## How to apply

Install Flexiplex via 'conda install -c bioconda -c conda-forge flexiplex' and run it in known-barcode mode with the 10x long-read preset, passing the barcode list (1022 true + 10000 decoy barcodes in the hiPSC benchmark) with -k to assign reads to cellular barcodes. Then run flexiplex-filter with the barcode whitelist to produce the barcode counts file and remove empty droplets using the inflection point method. Run competing tools (scTagger, FLAMES) on the identical FASTQ and barcode list under the same thread conditions (16 threads), with a 48 h wall-clock cutoff for tools that may fail to finish. Score each tool's read-barcode assignments against the truth set of 1022 true barcodes and record wall-clock runtime and peak RAM in benchmark_results.csv. Rationale: Flexiplex uses Levenshtein-distance matching to tolerate barcode sequencing errors, and the paper's benchmark shows it achieves the best accuracy-vs-cost balance — 7–14 h with <1 GB RAM versus scTagger's >100 GB RAM despite 1.5–5 h runtime, while FLAMES fails to complete in reasonable time.

## Related tools

- **Flexiplex** (Primary demultiplexer: assigns long reads to cellular barcodes using Levenshtein-distance matching with 10x long-read presets) — https://github.com/DavidsonGroup/flexiplex
- **flexiplex-filter** (Filters barcode assignments against a whitelist and removes empty droplets using the inflection point method) — https://github.com/DavidsonGroup/flexiplex
- **conda** (Installation channel for Flexiplex (bioconda/conda-forge))
- **scTagger** (Comparison tool in the benchmark; faster (1.5–5 h) but requires over 100 GB RAM)
- **FLAMES** (Comparison tool in the benchmark; failed to finish within the 48 h wall-clock cutoff) — https://github.com/LuyiTian/FLAMES

## Examples

```
flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- Flexiplex yields the highest number of correctly demultiplexed reads against the 1022-true-barcode truth set
- Flexiplex runtime falls in the 7–14 h range on 16 threads with peak RAM <1 GB
- scTagger completes in 1.5–5 h but peak RAM exceeds 100 GB
- FLAMES completes fewer than 30% of reads after a 48 h wall-clock cutoff
- flexiplex-filter output is a valid barcode counts file and the empty-droplet cutoff matches the inflection point of the barcode-count distribution

## Limitations

- Benchmark results are dataset-specific (hiPSC PromethION, ~11,000 barcodes with 1022 true + 10,000 decoys); runtime and memory trade-offs may differ on other datasets
- Flexiplex matches based on Levenshtein distance, so error tolerance and barcode length settings must match the chemistry; wrong preset (e.g. not 10x3v3) will misassign barcodes
- Competing tools may be impractical: scTagger requires >100 GB RAM and FLAMES may not finish within a 48 h cutoff

## Evidence

- [abstract] Levenshtein-distance matching tolerates barcode errors: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [methods] Flexiplex command-line usage in known-barcode mode: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [methods] flexiplex-filter whitelist filtering: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [methods] Flexiplex can remove empty droplets via inflection point: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [readme] Flexiplex installation via conda: "conda install -c bioconda -c conda-forge flexiplex"
- [abstract] Benchmark finding on accuracy vs computational efficiency: "Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools"
