---
name: single-cell-cluster-assignment-counting
description: Use when evaluating sequence-search or demultiplexing tools on single-cell
  FASTQ data by measuring how many cells in a known cluster are recovered via their
  barcodes after matching target sequences such as BCAS4-BCAS3, E1A, or a variant
  allele.
when_to_use_negative:
- The goal is raw sequence search speed on plain text or FASTA rather than single-cell
  cluster recovery — grep/ugrep-style line matching suffices and cluster counting
  is irrelevant.
- Input data lacks cell barcodes or a known cluster assignment (e.g. bulk sequencing
  reads), so cells-in-correct-cluster cannot be computed.
- Demultiplexing or barcode discovery from noisy long-read data is the goal, which
  is a different Flexiplex mode than error-tolerant grep-like search.
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Flexiplex
  role: Error-tolerant grep-like sequence search (Levenshtein distance, edit distance
    2) used to match target sequences in FASTQ reads and recover cell barcodes
  repo: https://github.com/DavidsonGroup/flexiplex
- name: grep
  role: Exact-match baseline searcher run on the same FASTQ for runtime and recovery
    comparison
- name: ugrep
  role: Faster exact-match baseline searcher compatible with GNU grep, run on the
    same FASTQ for comparison
  repo: https://github.com/Genivia/ugrep
- name: seqkit
  role: FASTQ-aware exact-match baseline (seqkit grep) run on the same FASTQ for comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:9d69faa5e9f8c94691844b5064dfd97a9026a1dba9fe1aa3c1676d4deb4dca3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - Flexiplex
  - grep
  - seqkit
  - ugrep
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-cluster-assignment-counting

## Summary

Assign cell barcodes from sequence-search-matched reads to their single-cell clusters and count cells in the correct cluster, benchmarking error-tolerant search (Flexiplex) against exact matchers (grep, ugrep, seqkit grep) on Illumina short-read single-cell RNA-seq data.

## When to use

Use when evaluating sequence-search or demultiplexing tools on single-cell FASTQ data by measuring how many cells in a known cluster are recovered via their barcodes after matching target sequences such as BCAS4-BCAS3, E1A, or a variant allele.

## When NOT to use

- The goal is raw sequence search speed on plain text or FASTA rather than single-cell cluster recovery — grep/ugrep-style line matching suffices and cluster counting is irrelevant.
- Input data lacks cell barcodes or a known cluster assignment (e.g. bulk sequencing reads), so cells-in-correct-cluster cannot be computed.
- Demultiplexing or barcode discovery from noisy long-read data is the goal, which is a different Flexiplex mode than error-tolerant grep-like search.

## Inputs

- Illumina short-read single-cell RNA-seq FASTQ dataset (Chen et al. 2021)
- Target nucleotide sequences: BCAS4-BCAS3 fusion, Adenovirus E1A, and rs878887783 variant
- Cell-to-cluster assignment table mapping cell barcodes to expression clusters

## Outputs

- cells_in_correct_clusters.tsv with per-tool, per-target cells-in-correct-cluster counts, reference-allele recovery, and wall-clock runtimes
- Per-tool/target barcode lists extracted from matched reads

## How to apply

Run each search tool (e.g. flexiplex -x <target> -d grep -f 2, plus grep, ugrep, and seqkit grep) on the same Illumina short-read single-cell RNA-seq FASTQ file, using edit distance 2 for the error-tolerant tool so sequencing errors are tolerated while exact matchers cannot be. Extract cell barcodes from the matched reads for each tool/target combination and tally how many distinct cells map to the cluster known to express the target (BCAS4-BCAS3, Adenovirus E1A, or rs878887783). For the BCAS4-BCAS3 fusion, additionally compute reference-allele recovery, which should reach about 97% with Flexiplex. Record wall-clock runtimes alongside counts so accuracy can be balanced against computational efficiency, then write the reproduced counts and runtimes to a TSV such as cells_in_correct_clusters.tsv. Validate by comparing per-tool cells-in-correct-cluster counts against reported values (365/956 for BCAS4-BCAS3, 439/869 for E1A, 515/987 for rs878887783) and confirming Flexiplex runtime (~24 min) sits well below seqkit grep (~190 min).

## Related tools

- **Flexiplex** (Error-tolerant grep-like sequence search (Levenshtein distance, edit distance 2) used to match target sequences in FASTQ reads and recover cell barcodes) — https://github.com/DavidsonGroup/flexiplex
- **grep** (Exact-match baseline searcher run on the same FASTQ for runtime and recovery comparison)
- **ugrep** (Faster exact-match baseline searcher compatible with GNU grep, run on the same FASTQ for comparison) — https://github.com/Genivia/ugrep
- **seqkit** (FASTQ-aware exact-match baseline (seqkit grep) run on the same FASTQ for comparison)

## Examples

```
flexiplex -x BCAS4_BCAS3_target -d grep -f 2 reads.fastq > matched.fastq
```

## Evaluation signals

- Reproduced cells-in-correct-cluster counts match reported values: 365/956 (BCAS4-BCAS3), 439/869 (E1A), 515/987 (rs878887783)
- BCAS4-BCAS3 reference-allele recovery reaches approximately 97%
- Flexiplex runtime (~24 min) is substantially below seqkit grep (~190 min) and ugrep (~40 min), while grep runs in 1-2 min but misses error-containing reads
- Flexiplex is highest or second highest in cells recovered per target, confirming error tolerance adds recovery beyond exact matching
- Output file cells_in_correct_clusters.tsv contains counts, recovery, and runtimes for all four tools across all three targets

## Limitations

- Exact-match tools (grep, ugrep, seqkit grep) cannot tolerate sequencing errors, so they recover fewer cells; only a Levenshtein-distance-based search with edit distance 2 captures error-containing reads
- Counts depend on the chosen edit distance threshold (-f 2); varying it changes both recovery and runtime
- Evaluation requires prior knowledge of which cluster expresses each target, so it applies only to datasets with known cell-line-specific sequences

## Evidence

- [methods] Error-tolerant search mode with edit distance 2: "Flexiplex can perform an error-tolerant grep-like search for a single sequence."
- [abstract] Levenshtein-distance-based imperfect matching: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [abstract] Benchmark on Illumina short-read single-cell data: "identifying cell-line-specific sequences in Illumina short-read single-cell data, and discovering and demultiplexing cellular barcodes from noisy long-read single-cell RNA-seq data"
- [abstract] Accuracy and efficiency balance claim: "Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools"
- [readme] Search invocation from task workflow: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
