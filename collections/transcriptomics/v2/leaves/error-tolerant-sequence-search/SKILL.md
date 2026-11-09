---
name: error-tolerant-sequence-search
description: Use when searching FASTQ/FASTA reads for a known sequence (e.g. a variant,
  transgene, or fusion junction) where base errors are expected and an exact string
  matcher would miss true matches.
when_to_use_negative:
- When reads and targets are error-free such that exact string matching suffices —
  plain grep/ugrep is 1–2 min versus ~24 min for Flexiplex
- When no sequence of interest is known and only flanking sequences are available
  — this is discovery mode, a different Flexiplex use case
- When the goal is general-purpose single-cell processing (mapping, count matrices)
  rather than targeted sequence search — a workflow such as epi2me-labs/wf-single-cell
  is more appropriate
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Flexiplex
  role: Error-tolerant grep-like sequence search based on Levenshtein distance; run
    with -x <target> -d grep -f <edit distance>
  repo: https://github.com/DavidsonGroup/flexiplex
- name: grep
  role: Exact-match baseline for sequence search benchmarking
- name: ugrep
  role: Faster exact/fuzzy grep baseline; supports approximate matching with -Z at
    a specified Levenshtein distance
  repo: https://github.com/Genivia/ugrep
- name: seqkit
  role: seqkit grep baseline for sequence search runtime/accuracy comparison
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:a4bf7f1180ef6a253cf49f296221c60804148434ee318712627d0d2d797af183
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - Flexiplex
  - grep
  - seqkit
  - ugrep
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# error-tolerant-sequence-search

## Summary

Use Flexiplex in grep-like known-sequence mode to find reads containing a target sequence of interest while tolerating sequencing errors via Levenshtein-distance matching, where exact-match tools like grep fail on Illumina short-read single-cell data.

## When to use

Use when searching FASTQ/FASTA reads for a known sequence (e.g. a variant, transgene, or fusion junction) where base errors are expected and an exact string matcher would miss true matches.

## When NOT to use

- When reads and targets are error-free such that exact string matching suffices — plain grep/ugrep is 1–2 min versus ~24 min for Flexiplex
- When no sequence of interest is known and only flanking sequences are available — this is discovery mode, a different Flexiplex use case
- When the goal is general-purpose single-cell processing (mapping, count matrices) rather than targeted sequence search — a workflow such as epi2me-labs/wf-single-cell is more appropriate

## Inputs

- FASTQ (or FASTA) reads file, e.g. Illumina short-read single-cell RNA-seq data
- Known target/query nucleotide sequence(s), e.g. BCAS4-BCAS3 junction, Adenovirus E1A, rs878887783 variant sequence
- Optional barcode whitelist (e.g. 3M-february-2018.txt) when chaining demultiplexing
- Edit-distance tolerance setting (-f) for allowed Levenshtein-distance mismatches

## Outputs

- Matching reads in FASTQ/FASTA with error-tolerant hits to the query sequence
- Read-barcode assignments in read headers (when piped to a demultiplexing instance)
- Per-tool cells-in-correct-cluster counts and runtimes (e.g. cells_in_correct_clusters.tsv)

## How to apply

Run Flexiplex in error-tolerant grep-like search mode with the target sequence supplied via -x and the edit distance set with -f (e.g. -f 2 or -f 3 Levenshtein distance for Illumina short reads). Inputs are the FASTQ file (e.g. the Chen et al. 2021 short-read single-cell RNA-seq data) and query sequences such as BCAS4-BCAS3, Adenovirus E1A, or the rs878887783 variant. Extract cell barcodes from matched reads and tally cells in the correct cluster to quantify recovery; benchmark against exact-matching grep/ugrep and seqkit grep. Because Flexiplex supports standard IO, a search instance can be piped into a second Flexiplex instance (e.g. -d 10x3v3 -k barcode_list.txt) to demultiplex barcodes from variant-containing reads in one command. In the reported benchmark this combination matched the most cells in the correct cluster (365/956, 439/869, 515/987) at ~97% reference-allele recovery for BCAS4-BCAS3 while running in ~24 min versus ~190 min for seqkit grep.

## Related tools

- **Flexiplex** (Error-tolerant grep-like sequence search based on Levenshtein distance; run with -x <target> -d grep -f <edit distance>) — https://github.com/DavidsonGroup/flexiplex
- **grep** (Exact-match baseline for sequence search benchmarking)
- **ugrep** (Faster exact/fuzzy grep baseline; supports approximate matching with -Z at a specified Levenshtein distance) — https://github.com/Genivia/ugrep
- **seqkit** (seqkit grep baseline for sequence search runtime/accuracy comparison)

## Examples

```
flexiplex -x <target_sequence> -d grep -f 2 reads.fastq
```

## Evaluation signals

- Cells-in-correct-cluster counts match reported values: 365/956 (BCAS4-BCAS3), 439/869 (E1A), 515/987 (rs878887783)
- BCAS4-BCAS3 reference-allele recovery reaches ~97%
- Flexiplex runtime is ~24 min, well below seqkit grep (~190 min) and ugrep (~40 min)
- Piped search+demultiplex output is valid FASTQ with cell-barcode/UMI annotation fields in read headers
- Exact-match tools recover fewer true variant-containing cells than the edit-distance-2/3 search

## Limitations

- Error-tolerant search is slower than exact grep (Flexiplex ~24 min vs 1–2 min for grep on the benchmark FASTQ)
- Matching accuracy depends on the chosen edit distance (-f); too large a distance risks spurious matches
- Findings were demonstrated on Illumina short-read single-cell data for specific targets; performance on other data types/targets may differ

## Evidence

- [abstract] Levenshtein-distance based error-tolerant matching: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [methods] error-tolerant grep-like search workflow step: "Flexiplex can perform an error-tolerant grep-like search for a single sequence."
- [methods] chained search then demultiplex invocation: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [other] standard IO enables piping Flexiplex instances: [evidence span withheld — anchor status: not_found]
- [readme] ugrep fuzzy search capability as alternative baseline: [evidence span withheld — anchor status: external_doc]
