---
name: chimeric-read-splitting
description: Use when demultiplexing long reads (e.g. Ebrahimi et al. 2022 simulated
  FASTQ) that may contain more than one barcode construct per read, and each detected
  construct must be reported separately.
when_to_use_negative:
- Reads are short single-end reads where at most one barcode construct is expected,
  so iterative re-search adds no value
- Only the flanking sequence is known and barcodes must be discovered de novo rather
  than matched against a whitelist (Flexiplex's discovery mode handles that separately)
- The goal is downstream cell-barcode filtering against a whitelist with empty-droplet
  removal via inflection point — that is flexiplex-filter's role, not chimeric splitting
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Flexiplex
  role: Reference implementation whose flank-search/Levenshtein-match loop and chimeric
    read splitting the module reproduces
  repo: https://davidsongroup.github.io/flexiplex/
- name: Python
  role: Implements FASTQ parsing, edlib-based flank search, masking, and the iterative
    re-search loop
- name: edlib
  role: Error-tolerant search of left/right flanking sequences at a specified edit
    distance
- name: pip
  role: Installs the Python dependencies (python-pip required by make install)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:b14b90b0f9b9f17ed4845451df78aac59d03c536bc481bf80ba093d59755a4e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Flexiplex
  - Python
  - edlib
  - pip
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# chimeric-read-splitting

## Summary

Iteratively re-search a read for additional flank/barcode constructs after a first hit, masking each found region so chimeric reads carrying multiple barcode+UMI constructs are split into multiple assignments, mirroring Flexiplex's handling of chimeric long reads.

## When to use

Use when demultiplexing long reads (e.g. Ebrahimi et al. 2022 simulated FASTQ) that may contain more than one barcode construct per read, and each detected construct must be reported separately.

## When NOT to use

- Reads are short single-end reads where at most one barcode construct is expected, so iterative re-search adds no value
- Only the flanking sequence is known and barcodes must be discovered de novo rather than matched against a whitelist (Flexiplex's discovery mode handles that separately)
- The goal is downstream cell-barcode filtering against a whitelist with empty-droplet removal via inflection point — that is flexiplex-filter's role, not chimeric splitting

## Inputs

- Long-read FASTQ file (e.g. simulated reads in the 'reads.fastq' format)
- Barcode whitelist text file ('barcode_list.txt')
- Left and right flanking sequences with an edit-distance tolerance parameter (analogous to flexiplex -f)

## Outputs

- Per-read barcode assignments for each detected construct on chimeric reads
- Barcode count table written to flexiplex_barcodes_counts.txt

## How to apply

Parse the FASTQ reads and flank sequences (matching Flexiplex's 'reads.fastq' / 'barcode_list.txt' input format), then locate left/right flanking sequences with edlib at a user-specified edit-distance tolerance (analogous to flexiplex's -f parameter) and extract the intervening barcode+UMI sequence. Match the extracted barcode against the whitelist with a dynamic-programming Levenshtein aligner, reporting an assignment only under the unique-best-match rule (the minimal edit distance, default tolerance two, is attained by exactly one barcode). After a hit, mask that flank/barcode/UMI region and re-run the flank search on the remaining read sequence, looping until no further constructs are found, which splits chimeric reads into multiple per-read barcode assignments. Write assignments and barcode counts to flexiplex_barcodes_counts.txt.

## Related tools

- **Flexiplex** (Reference implementation whose flank-search/Levenshtein-match loop and chimeric read splitting the module reproduces) — https://davidsongroup.github.io/flexiplex/
- **Python** (Implements FASTQ parsing, edlib-based flank search, masking, and the iterative re-search loop)
- **edlib** (Error-tolerant search of left/right flanking sequences at a specified edit distance)
- **pip** (Installs the Python dependencies (python-pip required by make install))

## Examples

```
flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- Every reported barcode assignment satisfies the unique-best-match rule: the minimal Levenshtein distance is attained by exactly one whitelist barcode
- Correct-barcode assignment rate on the Ebrahimi et al. 2022 simulated reads is consistent with Flexiplex accuracy reported in Fig. 1D
- Chimeric reads yield multiple assignments, one per masked-and-re-searched construct, with no re-detection of the same region
- flexiplex_barcodes_counts.txt counts sum to the number of per-read assignments produced

## Limitations

- Matching tolerance and flank edit distance trade sensitivity for specificity; a barcode with two equally best matches at the minimal Levenshtein distance is not reported (ambiguous)
- Splitting depends on flank hits being found within the specified edit distance; heavily degraded flanks cause missed constructs on chimeric reads
- Performance-oriented design assumes reasonably long reads; the workflow was benchmarked on simulated noisy long-read single-cell RNA-seq data, not general sequence search

## Evidence

- [abstract] Flexiplex is based on the Levenshtein distance and allows imperfect matches: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [intro] Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment.: "Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment."
- [methods] Workflow step 4: mask found region and re-search to split chimeric reads: [evidence span withheld — anchor status: not_found]
- [methods] Unique-best-match rule from finding: [evidence span withheld — anchor status: not_found]
- [readme] README usage showing flexiplex demultiplexing invocation: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
