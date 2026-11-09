---
name: command-line-sequence-search-benchmarking
description: Use when searching short-read single-cell RNA-seq FASTQ for known target
  sequences (fusions, viral transcripts, variants) and you must choose between error-tolerant
  search (Flexiplex) and exact string matchers (grep/ugrep/seqkit) on accuracy and
  runtime.
when_to_use_negative:
- Demultiplexing barcodes or filtering against a whitelist — use Flexiplex's demultiplexer
  presets (e.g. -d 10x3v3) or flexiplex-filter, not grep-like search mode
- Data are noisy long reads needing barcode discovery mode — error-tolerant grep search
  targets a single known sequence and is benchmarked on short reads
- Pattern matching needs only exact substring search with no error tolerance and no
  cell-level accuracy evaluation — plain grep suffices without benchmarking
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: Flexiplex
  role: Error-tolerant grep-like sequence search based on Levenshtein distance, run
    with edit distance 2 per target sequence
  repo: https://github.com/DavidsonGroup/flexiplex
- name: grep
  role: Exact-match baseline for sequence search; benchmarked for runtime and cells-in-correct-cluster
    comparison
- name: ugrep
  role: Faster exact-match grep alternative benchmarked as baseline; supports fuzzy
    search via -Z
  repo: https://github.com/Genivia/ugrep
- name: seqkit
  role: seqkit grep baseline for FASTQ sequence search, benchmarked for runtime (~190
    min) and accuracy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:6fb339c933228fc37905d2baa2d82f3ed99e2d9447585f1db95ce8d274c8dc0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - Flexiplex
  - grep
  - seqkit
  - ugrep
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# command-line-sequence-search-benchmarking

## Summary

Benchmark error-tolerant sequence search of target sequences (e.g. BCAS4-BCAS3 fusion, Adenovirus E1A, SNP rs878887783) in single-cell FASTQ data by comparing Flexiplex in grep-like mode against exact matchers grep, ugrep, and seqkit grep on cells-in-correct-cluster counts and wall-clock runtime.

## When to use

Use when searching short-read single-cell RNA-seq FASTQ for known target sequences (fusions, viral transcripts, variants) and you must choose between error-tolerant search (Flexiplex) and exact string matchers (grep/ugrep/seqkit) on accuracy and runtime.

## When NOT to use

- Demultiplexing barcodes or filtering against a whitelist — use Flexiplex's demultiplexer presets (e.g. -d 10x3v3) or flexiplex-filter, not grep-like search mode
- Data are noisy long reads needing barcode discovery mode — error-tolerant grep search targets a single known sequence and is benchmarked on short reads
- Pattern matching needs only exact substring search with no error tolerance and no cell-level accuracy evaluation — plain grep suffices without benchmarking

## Inputs

- Illumina short-read single-cell RNA-seq FASTQ file (e.g. Chen et al. 2021)
- Target sequences in text form (BCAS4-BCAS3 fusion, Adenovirus E1A transcript, rs878887783 variant sequence)
- Cell-to-cluster assignments for matched reads

## Outputs

- cells_in_correct_clusters.tsv with per-tool, per-target cells-in-correct-cluster counts and reference-allele recovery
- Wall-clock runtimes per tool and per target sequence

## How to apply

Obtain the single-cell FASTQ dataset (e.g. Chen et al. 2021 Illumina short reads) and the target sequences of interest. Run Flexiplex in error-tolerant grep-like search mode for each target with edit distance 2 (flexiplex -x <target> -d grep -f 2), then run grep, ugrep, and seqkit grep on the same FASTQ, recording wall-clock runtimes for each tool/target combination. Extract cell barcodes from matched reads for each combination and tally cells assigned to the correct cluster; for the BCAS4-BCAS3 fusion also compute reference-allele recovery. Compare per-tool counts and runtimes against expected values (365/956 BCAS4-BCAS3, 439/869 E1A, 515/987 rs878887783; ~24 min Flexiplex, 1-2 min grep, ~40 min ugrep, ~190 min seqkit grep). Rationale: Flexiplex uses Levenshtein distance so it tolerates sequencing errors that exact-matching grep cannot, while remaining substantially faster than seqkit grep.

## Related tools

- **Flexiplex** (Error-tolerant grep-like sequence search based on Levenshtein distance, run with edit distance 2 per target sequence) — https://github.com/DavidsonGroup/flexiplex
- **grep** (Exact-match baseline for sequence search; benchmarked for runtime and cells-in-correct-cluster comparison)
- **ugrep** (Faster exact-match grep alternative benchmarked as baseline; supports fuzzy search via -Z) — https://github.com/Genivia/ugrep
- **seqkit** (seqkit grep baseline for FASTQ sequence search, benchmarked for runtime (~190 min) and accuracy)

## Examples

```
flexiplex -x <target_sequence> -d grep -f 2 reads.fastq
```

## Evaluation signals

- Reproduced cells-in-correct-cluster counts match reported values: 365/956 (BCAS4-BCAS3), 439/869 (E1A), 515/987 (rs878887783)
- BCAS4-BCAS3 reference-allele recovery reaches 97%
- Flexiplex runtime ~24 min is substantially below seqkit grep ~190 min, and ugrep ~40 min and grep 1-2 min fall in the expected ranges
- Barcodes are extracted consistently from matched reads for every tool/target combination before cluster tallying
- Flexiplex tolerates sequencing errors (edit distance 2) that exact-matching grep misses, so Flexiplex cell counts should not trail grep substantially

## Limitations

- Exact matchers (grep, ugrep without -Z, seqkit grep) cannot tolerate sequencing errors, so they undercount cells carrying erroneous target reads
- Error-tolerant search costs runtime (~24 min for Flexiplex vs 1-2 min grep on the benchmark FASTQ)
- grep-like search mode in Flexiplex handles a single sequence at a time, unlike demultiplexing a barcode list

## Evidence

- [methods] Error-tolerant search mode: "Flexiplex can perform an error-tolerant grep-like search for a single sequence."
- [abstract] Levenshtein distance basis: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [abstract] Benchmark findings on short-read data: "identifying cell-line-specific sequences in Illumina short-read single-cell data, and discovering and demultiplexing cellular barcodes from noisy long-read single-cell RNA-seq data"
- [abstract] Accuracy/efficiency claim: "Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools"
- [intro] Search-and-report behaviour: "Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment."
