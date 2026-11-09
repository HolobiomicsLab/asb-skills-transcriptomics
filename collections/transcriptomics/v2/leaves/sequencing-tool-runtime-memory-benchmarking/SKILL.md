---
name: sequencing-tool-runtime-memory-benchmarking
description: Decision support for choosing a barcode demultiplexer when large noisy
  long-read datasets (e.g. 61M-read PromethION FASTQ with 1022 true plus 10000 decoy
  barcodes) must be processed under wall-clock and memory constraints.
when_to_use_negative:
- When only a single tool is being validated on its own output rather than compared
  against alternatives under identical inputs and thread counts.
- When the question is barcode discovery from flanking sequences alone (Flexiplex
  discovery mode) rather than comparing known-barcode demultiplexing cost and accuracy.
- When datasets are small enough that runtime and RAM differences between tools are
  immaterial.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Flexiplex
  role: 'Demultiplexer under test: assigns reads to cellular barcodes in known-barcode
    mode using Levenshtein-distance imperfect matching; benchmarked for accuracy,
    runtime, and memory'
  repo: https://davidsongroup.github.io/flexiplex/
- name: flexiplex-filter
  role: 'Post-demultiplexing filtering step: filters barcodes against a whitelist
    and removes empty droplets using an inflection point method before counting'
  repo: https://github.com/DavidsonGroup/flexiplex/releases
- name: conda
  role: Installation channel for Flexiplex to standardize tool versions across the
    benchmark
- name: scTagger
  role: Competing demultiplexer benchmarked on the identical FASTQ and barcode list
    under the same 16-thread conditions
- name: FLAMES
  role: Competing demultiplexer launched with a 48 h wall-clock cutoff to confirm
    its failure to complete
  repo: https://github.com/LuyiTian/FLAMES
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:7e0f3417e368d0a50704a67571fad7bb5adaf5a128cc9ab610e79a6afcde863a
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Sequencing-tool runtime-memory benchmarking

## Summary

A method for head-to-head benchmarking of demultiplexing/search tools (e.g. Flexiplex vs scTagger vs FLAMES) on the same FASTQ and barcode list, recording wall-clock runtime, peak RAM, and correctness of read-barcode assignments against a truth set.

## When to use

Decision support for choosing a barcode demultiplexer when large noisy long-read datasets (e.g. 61M-read PromethION FASTQ with 1022 true plus 10000 decoy barcodes) must be processed under wall-clock and memory constraints.

## When NOT to use

- When only a single tool is being validated on its own output rather than compared against alternatives under identical inputs and thread counts.
- When the question is barcode discovery from flanking sequences alone (Flexiplex discovery mode) rather than comparing known-barcode demultiplexing cost and accuracy.
- When datasets are small enough that runtime and RAM differences between tools are immaterial.

## Inputs

- FASTQ (or FASTA) read file, e.g. a 61M-read PromethION single-cell long-read FASTQ
- Barcode list / whitelist file (true cellular barcodes plus decoy barcodes), e.g. barcode_list.txt with 1022 true + 10000 decoy barcodes
- Truth set of expected read-barcode assignments (1022 true barcodes)

## Outputs

- Per-tool read-barcode assignment counts scored against the truth set
- Wall-clock runtime and peak RAM per tool per run
- benchmark_results.csv compiling correct-read counts, runtimes, and memory footprints

## How to apply

Obtain the same input FASTQ (e.g. You et al. 2023 hiPSC PromethION, 61M reads) and identical barcode list (1022 true cellular barcodes plus 10000 decoys) for every tool. Install each tool under comparable conditions (e.g. 'conda install -c bioconda -c conda-forge flexiplex'), run Flexiplex in known-barcode mode with 10x long-read settings passing the barcode list with -k, then run flexiplex-filter with the whitelist to remove empty droplets using the inflection point method; run the competing tools on the identical FASTQ and barcode list with the same thread count (16) and a fixed wall-clock cutoff (e.g. 48 h for FLAMES). Score each tool's read-barcode assignments against the 1022-barcode truth set and record wall-clock runtime and peak RAM, then compile correct-read counts, runtimes, and memory footprints into benchmark_results.csv. Accept the benchmark when the trade-offs are consistent, e.g. Flexiplex yields the most correctly demultiplexed reads at 7-14 h on 16 threads with <1 GB RAM while scTagger runs 1.5-5 h but needs >100 GB RAM.

## Related tools

- **Flexiplex** (Demultiplexer under test: assigns reads to cellular barcodes in known-barcode mode using Levenshtein-distance imperfect matching; benchmarked for accuracy, runtime, and memory) — https://davidsongroup.github.io/flexiplex/
- **flexiplex-filter** (Post-demultiplexing filtering step: filters barcodes against a whitelist and removes empty droplets using an inflection point method before counting) — https://github.com/DavidsonGroup/flexiplex/releases
- **conda** (Installation channel for Flexiplex to standardize tool versions across the benchmark)
- **scTagger** (Competing demultiplexer benchmarked on the identical FASTQ and barcode list under the same 16-thread conditions)
- **FLAMES** (Competing demultiplexer launched with a 48 h wall-clock cutoff to confirm its failure to complete) — https://github.com/LuyiTian/FLAMES

## Examples

```
flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- Flexiplex yields the highest number of correctly demultiplexed reads when scored against the 1022-true-barcode truth set.
- Flexiplex completes in 7-14 h using 16 threads with <1 GB peak RAM on the 61M-read PromethION FASTQ.
- scTagger completes in 1.5-5 h on identical inputs but requires >100 GB peak RAM.
- FLAMES completes <30% of the run within the 48 h wall-clock cutoff, confirming the failure-to-complete record.
- benchmark_results.csv contains per-tool correct-read counts, runtimes, and memory footprints for all three tools.

## Limitations

- Memory footprints differ by orders of magnitude between tools (Flexiplex <1 GB vs scTagger >100 GB), so comparisons are invalid unless all tools are run on the same FASTQ, barcode list, and thread count.
- Some competing tools (FLAMES) may fail to finish in reasonable time, requiring an explicit wall-clock cutoff (e.g. 48 h) and a partial-completion criterion (<30%) to record a result.
- Results are specific to the dataset scale and barcode composition (1022 true + 10000 decoys on PromethION long reads); runtime and accuracy trade-offs may not transfer to short-read or smaller datasets.

## Evidence

- [results] Flexiplex completed in 7–14 h using 16 threads with <1 GB RAM, while scTagger, though faster (1.5–5 h), required over 100 GB of RAM; FLAMES failed to finish in reasonable time.: [evidence span withheld — anchor status: not_found]
- [methods] conda install command from methods: "You can also install Flexiplex using Anaconda: `conda install -c bioconda -c conda-forge flexiplex`"
- [methods] whitelist filtering and inflection-point droplet removal: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [readme] README example invocation for assigning reads to barcodes: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [abstract] abstract claim on accuracy and efficiency balance: "Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools"
