---
name: simulated-read-demultiplexing-benchmark
description: Use when benchmarking or reimplementing barcode demultiplexing on simulated
  long-read FASTQ data, or when the analysis goal is to verify that a demultiplexer
  reproduces Flexiplex's unique--match assignment behaviour and reported accuracy.
when_to_use_negative:
- Input reads are already demultiplexed or carry validated barcode assignments — re-searching
  adds no information
- Discovery mode is needed where only flanking sequences are known and barcodes must
  be discovered de novo rather than matched to a whitelist
- The task is whitelist filtering or empty-droplet removal (flexiplex-filter's inflection
  point method) rather than read-to-barcode assignment
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: Flexiplex
  role: Reference demultiplexer whose flank-search, Levenshtein barcode-match and
    chimeric-split loop is reconstructed and benchmarked
  repo: https://davidsongroup.github.io/flexiplex/
- name: Python
  role: Implementation language for FASTQ parsing, the edlib flank search, the dynamic-programming
    Levenshtein aligner, and iterative chimeric splitting
- name: pip
  role: Installs Python dependencies for the reimplementation (e.g. edlib), mirroring
    flexiplex's `make install` path
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:0f4d612c6c66c8e0e986c387ef24f4a5d2bc43c4f7088c3ecf99018c7644277b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Flexiplex
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# simulated-read-demultiplexing-benchmark

## Summary

Reconstructs and benchmarks Flexiplex-style read demultiplexing on simulated long-read data: flank search with edlib, unique-best-match Levenshtein barcode assignment, and chimeric-read splitting, validated against the accuracy reported in Fig. 1D of the Flexiplex paper.

## When to use

Use when benchmarking or reimplementing barcode demultiplexing on simulated long-read FASTQ data, or when the analysis goal is to verify that a demultiplexer reproduces Flexiplex's unique-best-match assignment behaviour and reported accuracy.

## When NOT to use

- Input reads are already demultiplexed or carry validated barcode assignments — re-searching adds no information
- Discovery mode is needed where only flanking sequences are known and barcodes must be discovered de novo rather than matched to a whitelist
- The task is whitelist filtering or empty-droplet removal (flexiplex-filter's inflection point method) rather than read-to-barcode assignment

## Inputs

- Simulated long-read FASTQ file (Ebrahimi et al. 2022 simulated reads, 'reads.fastq')
- Barcode whitelist text file ('barcode_list.txt')
- User-specified flank and barcode edit-distance tolerances (analogous to flexiplex -f and default edit distance of two)

## Outputs

- flexiplex_barcodes_counts.txt with per-read barcode assignments and barcode counts
- Per-read barcode assignment table (read ID to assigned barcode or unassigned)
- Correct-barcode assignment rate comparable to Fig. 1D of the Flexiplex paper

## How to apply

Load the Ebrahimi et al. 2022 simulated long-read FASTQ reads and the barcode whitelist (mirroring flexiplex's 'reads.fastq' / 'barcode_list.txt' input format). Implement the left/right flank search with edlib at a user-specified edit-distance tolerance (analogous to flexiplex's -f parameter) and extract the intervening barcode+UMI sequence between flank hits. Match each extracted barcode against the full barcode list with a dynamic-programming Levenshtein aligner, reporting an assignment only when the minimal edit distance (default tolerance of two) is attained by exactly one barcode — the unique-best-match rule. After a hit is found, mask that region and re-search the remaining read to split chimeric reads carrying multiple barcode constructs; then write per-read barcode assignments and counts to flexiplex_barcodes_counts.txt and compare the correct-assignment rate against Fig. 1D.

## Related tools

- **Flexiplex** (Reference demultiplexer whose flank-search, Levenshtein barcode-match and chimeric-split loop is reconstructed and benchmarked) — https://davidsongroup.github.io/flexiplex/
- **Python** (Implementation language for FASTQ parsing, the edlib flank search, the dynamic-programming Levenshtein aligner, and iterative chimeric splitting)
- **pip** (Installs Python dependencies for the reimplementation (e.g. edlib), mirroring flexiplex's `make install` path)

## Examples

```
flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- Every reported assignment satisfies the unique-best-match criterion: the minimal Levenshtein distance is attained by exactly one barcode in the whitelist
- Correct-barcode assignment rate on the simulated dataset is consistent with the Flexiplex accuracy reported in Fig. 1D
- Output file flexiplex_barcodes_counts.txt contains per-read assignments and barcode counts in the expected format
- Chimeric reads yield multiple barcode constructs, each found by masking the prior hit and re-searching the remaining read sequence
- Flank hits are located within the user-specified edit-distance tolerance before the intervening barcode+UMI sequence is extracted

## Limitations

- The unique-best-match rule means reads whose minimum Levenshtein distance is shared by two or more barcodes are not assigned, which can reduce assignment rate on noisy long reads
- Validation depends on the Ebrahimi et al. 2022 simulated dataset and Fig. 1D reference values; results on other read sets are not covered by this benchmark
- Flexiplex matches are based on the Levenshtein distance and thus allow imperfect matches, so false assignments at loose edit-distance settings remain possible

## Evidence

- [other] Flexiplex reports the best matching barcode within a specified edit distance (default of two), but only if no other barcode has an equally lowest Levenshtein distance.: [evidence span withheld — anchor status: not_found]
- [abstract] Levenshtein distance basis: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [methods] CLI invocation: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [readme] Two modes: "Flexiplex works in two modes: (i) when one or more sequences of interest are known, such as barcodes, and (ii) discovery mode"
