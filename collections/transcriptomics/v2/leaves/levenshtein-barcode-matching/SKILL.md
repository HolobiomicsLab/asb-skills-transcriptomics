---
name: levenshtein-barcode-matching
description: Use when demultiplexing long-read (FASTQ/FASTA) data against a known
  barcode whitelist and you need edit-distance-tolerant barcode matching with a unique--match
  assignment rule.
when_to_use_negative:
- Only flanking sequences are known and the barcode itself must be discovered (Flexiplex's
  discovery mode is a different workflow).
- The task is filtering an existing barcode-count table against a whitelist or removing
  empty droplets — use flexiplex-filter's inflection-point method instead.
- Input is aligned BAM/single-cell count matrices rather than raw FASTQ/FASTA reads
  (e.g. downstream steps of epi2me-labs/wf-single-cell).
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Flexiplex
  role: Reference implementation of Levenshtein-based demultiplexing whose flank-search,
    barcode-match, and chimeric-split loop is being reconstructed
  repo: https://github.com/DavidsonGroup/flexiplex
- name: Python
  role: Language for the reimplementation (FASTQ parsing, DP Levenshtein aligner,
    masking/iterative re-search)
- name: edlib
  role: Library used to perform error-tolerant left/right flank sequence search within
    a specified edit-distance tolerance
- name: pip
  role: Used to install the Flexiplex build environment (make install with python3.9/python-pip)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:8d946d90674458d5725c611f3946d42e88b0c37c0ee22f0da373ebf52d7a3a59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - Flexiplex
  - Python
  - edlib
  - pip
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# levenshtein-barcode-matching

## Summary

Reimplements Flexiplex's barcode-assignment logic: extract the barcode+UMI region between error-tolerant flank hits, then match the extracted barcode to a whitelist using Levenshtein distance, reporting an assignment only when the minimal edit distance is attained by exactly one barcode. This reproduces Flexiplex's accuracy (Fig. 1D) on simulated long-read data.

## When to use

Use when demultiplexing long-read (FASTQ/FASTA) data against a known barcode whitelist and you need edit-distance-tolerant barcode matching with a unique-best-match assignment rule.

## When NOT to use

- Only flanking sequences are known and the barcode itself must be discovered (Flexiplex's discovery mode is a different workflow).
- The task is filtering an existing barcode-count table against a whitelist or removing empty droplets — use flexiplex-filter's inflection-point method instead.
- Input is aligned BAM/single-cell count matrices rather than raw FASTQ/FASTA reads (e.g. downstream steps of epi2me-labs/wf-single-cell).

## Inputs

- Long-read FASTQ file of simulated reads (reads.fastq)
- Barcode whitelist text file (barcode_list.txt)
- Left and right flank sequences
- Edit-distance tolerance parameters (e.g. default maximum Levenshtein distance of 2; flank search tolerance analogous to flexiplex -f)

## Outputs

- Per-read barcode assignments and barcode counts written to flexiplex_barcodes_counts.txt
- Split/processed reads in FASTQ form (analogous to new_reads.fastq)

## How to apply

Load simulated long-read FASTQ reads (Ebrahimi et al. 2022) and the barcode whitelist in Python. Use edlib to search each read for the left/right flanking sequences within a user-specified edit-distance tolerance (analogous to flexiplex's -f parameter) and extract the intervening barcode+UMI sequence. Compute Levenshtein distances from the extracted barcode to every whitelist barcode with a dynamic-programming aligner, and report an assignment only if the minimal distance is attained by exactly one barcode (unique-best-match rule, default maximum edit distance of two). After a hit, mask the matched region and re-search the remaining sequence to split chimeric reads carrying multiple barcode constructs, then write per-read assignments and counts to flexiplex_barcodes_counts.txt.

## Related tools

- **Flexiplex** (Reference implementation of Levenshtein-based demultiplexing whose flank-search, barcode-match, and chimeric-split loop is being reconstructed) — https://github.com/DavidsonGroup/flexiplex
- **Python** (Language for the reimplementation (FASTQ parsing, DP Levenshtein aligner, masking/iterative re-search))
- **edlib** (Library used to perform error-tolerant left/right flank sequence search within a specified edit-distance tolerance)
- **pip** (Used to install the Flexiplex build environment (make install with python3.9/python-pip))

## Examples

```
flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- Correct-barcode assignment rate on the simulated dataset is consistent with the Flexiplex accuracy reported in Fig. 1D of the paper.
- Every reported assignment satisfies the unique-best-match criterion: the minimal Levenshtein distance is attained by exactly one barcode in the whitelist.
- No assignment exceeds the maximum edit-distance threshold (default of two).
- Chimeric reads produce multiple barcode constructs after masking and iterative re-search of the remaining sequence.
- Output flexiplex_barcodes_counts.txt contains one record per read with a barcode assignment and per-barcode counts.

## Limitations

- Assignments are ambiguous and must be dropped when two or more whitelisted barcodes tie at the minimal Levenshtein distance.
- Accuracy depends on the flank-search edit-distance tolerance; too loose a tolerance can mis-extract the barcode+UMI interval.
- Flexiplex allows imperfect matches by design, so a reported barcode can differ from the true barcode by up to the allowed edit distance.

## Evidence

- [full_text] Flexiplex reports the best matching barcode within a specified edit distance (default of two), but only if no other barcode has an equally lowest Levenshtein distance.: [evidence span withheld — anchor status: not_found]
- [abstract] Flexiplex is based on the Levenshtein distance and allows imperfect matches: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [methods] flexiplex command-line invocation: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [intro] Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment.: "Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment."
- [readme] Flexiplex is a fast, multithreaded, and user-configurable demultiplexer.: "Flexiplex is a fast, multithreaded, and user-configurable demultiplexer."
