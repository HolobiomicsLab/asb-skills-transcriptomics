---
name: runtime-profiling-of-sequence-tools
description: Use when choosing a sequence-search tool for FASTQ/FASTA data where sequencing
  errors may cause exact matchers to miss targets, and you need runtime and sensitivity
  measured head-to-head on identical input.
when_to_use_negative:
- The target sequence is expected to appear error-free in reads, so exact-match grep/ugrep
  suffices and the Levenshtein tolerance of Flexiplex is unnecessary overhead
- The input is already demultiplexed or the goal is barcode discovery/demultiplexing
  rather than searching for a known sequence of interest
- Data are noisy long-read FASTQ where Flexiplex's demultiplexing modes (not grep-like
  search benchmarking) are the relevant use case
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Flexiplex
  role: Error-tolerant Levenshtein-distance sequence searcher being profiled; run
    in grep-like mode with edit distance 2
  repo: https://github.com/DavidsonGroup/flexiplex
- name: grep
  role: Exact-match baseline searcher used for runtime and sensitivity comparison
- name: ugrep
  role: Faster drop-in grep replacement baseline, supports fuzzy search via option
    -Z within a specified Levenshtein distance
  repo: https://github.com/Genivia/ugrep
- name: seqkit
  role: seqkit grep baseline for FASTQ sequence searching in the runtime comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:f2236ae432d591efea4ffdfb301718c1c4faf19a9f05416c1da54822aeabcca1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - Flexiplex
  - grep
  - seqkit
  - ugrep
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# runtime-profiling-of-sequence-tools

## Summary

Benchmarks Flexiplex's error-tolerant grep-like sequence search against exact-match tools (grep, ugrep, seqkit grep) on Illumina short-read single-cell FASTQ data, recording both cell-recovery accuracy and wall-clock runtime per tool/target combination.

## When to use

Use when choosing a sequence-search tool for FASTQ/FASTA data where sequencing errors may cause exact matchers to miss targets, and you need runtime and sensitivity measured head-to-head on identical input.

## When NOT to use

- The target sequence is expected to appear error-free in reads, so exact-match grep/ugrep suffices and the Levenshtein tolerance of Flexiplex is unnecessary overhead
- The input is already demultiplexed or the goal is barcode discovery/demultiplexing rather than searching for a known sequence of interest
- Data are noisy long-read FASTQ where Flexiplex's demultiplexing modes (not grep-like search benchmarking) are the relevant use case

## Inputs

- Illumina short-read single-cell RNA-seq FASTQ file (Chen et al. 2021 dataset)
- Target nucleotide sequences (BCAS4-BCAS3 fusion, Adenovirus E1A, rs878887783 variant)
- Installed copies of flexiplex, grep, ugrep, and seqkit

## Outputs

- Per-tool/target cells-in-correct-cluster counts (e.g. cells_in_correct_clusters.tsv)
- Wall-clock runtimes for each tool and target combination
- Reference-allele recovery percentage (97% for BCAS4-BCAS3 with Flexiplex)

## How to apply

Run Flexiplex in error-tolerant grep-like mode (edit distance 2, e.g. `flexiplex -x <target> -d grep -f 2`) for each target sequence and run grep, ugrep, and seqkit grep on the same FASTQ file, recording wall-clock runtimes for every tool/target combination. Extract cell barcodes from matched reads per tool and tally cells assigned to the correct cluster; for the BCAS4-BCAS3 fusion also compute reference-allele recovery. Rationale: Flexiplex is Levenshtein-distance-based so it tolerates sequencing errors that exact-match grep cannot, but exact matchers run faster (~1-2 min grep vs ~24 min Flexiplex vs ~190 min seqkit grep on the benchmark dataset), so the profiling must capture both axes. Compare recovered counts against expected values (365/956 cells for BCAS4-BCAS3, 439/869 for E1A, 515/987 for rs878887783) and write results to a TSV.

## Related tools

- **Flexiplex** (Error-tolerant Levenshtein-distance sequence searcher being profiled; run in grep-like mode with edit distance 2) — https://github.com/DavidsonGroup/flexiplex
- **grep** (Exact-match baseline searcher used for runtime and sensitivity comparison)
- **ugrep** (Faster drop-in grep replacement baseline, supports fuzzy search via option -Z within a specified Levenshtein distance) — https://github.com/Genivia/ugrep
- **seqkit** (seqkit grep baseline for FASTQ sequence searching in the runtime comparison)

## Examples

```
flexiplex -x <target_sequence> -d grep -f 2 reads.fastq > matched_reads.fastq
```

## Evaluation signals

- Reproduced cells-in-correct-cluster counts match reported values: 365/956 (BCAS4-BCAS3), 439/869 (E1A), 515/987 (rs878887783)
- BCAS4-BCAS3 reference-allele recovery reaches ~97% with Flexiplex
- Flexiplex wall-clock runtime (~24 min) is well below seqkit grep (~190 min) and above grep (1-2 min), with ugrep at ~40 min
- All tools are run on the identical FASTQ input and target set so counts and runtimes are directly comparable
- Only reads passing the per-tool search are tallied, so barcode extraction is restricted to matched reads for each tool/target combination

## Limitations

- Flexiplex's error tolerance comes at a runtime cost relative to exact-match grep (1-2 min), so exact matchers win on speed when sequences are error-free
- Runtimes reported (~24 min Flexiplex, ~40 min ugrep, ~190 min seqkit grep) are dataset-specific and will vary with read count, read length, and thread count
- Recovery is reported as cells in the correct cluster, not raw read matches, so the metric depends on the downstream clustering assignment

## Evidence

- [methods] Flexiplex can perform an error-tolerant grep-like search for a single sequence.: "Flexiplex can perform an error-tolerant grep-like search for a single sequence."
- [abstract] Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools: "Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools"
- [abstract] Flexiplex is based on the Levenshtein distance and allows imperfect matches: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [readme] Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment.: "Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment."
- [readme] ugrep fuzzy search within Levenshtein distance: [evidence span withheld — anchor status: external_doc]
