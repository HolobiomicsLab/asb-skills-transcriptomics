---
name: edlib-edit-distance-flank-search
description: Use when reads (FASTQ/FASTA) must be demultiplexed against a barcode
  whitelist by error-tolerant sequence matching, or when reproducing Flexiplex's flank-search/Levenshtein-match/chimeric-split
  logic on simulated long-read data such as the Ebrahimi et al. 2022 reads.
when_to_use_negative:
- The barcode whitelist is unknown and only flanking sequences are available — that
  is Flexiplex's discovery mode, not a known-barcode assignment task
- Input is already a gene-by-cell count matrix; downstream whitelist filtering and
  empty-droplet removal (e.g. flexiplex-filter with 3M-february-2018.txt) applies
  later, not to this search loop
- Reads require genomic mapping or gene/transcript assignment before barcode extraction,
  as in the epi2me-labs/wf-single-cell pipeline
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: Flexiplex
  role: 'Reference implementation being reconstructed: flank search with -f edit-distance
    parameter, Levenshtein-based barcode matching, and chimeric-read splitting'
  repo: https://davidsongroup.github.io/flexiplex/
- name: Python
  role: Language in which the FASTQ parsing, edlib flank search, Levenshtein dynamic-programming
    aligner, and masking loop are implemented
- name: edlib
  role: Python library used for error-tolerant search of the left/right flanking sequences
    within each read
- name: pip
  role: Installation path for the Python dependencies of the reconstructed module
    (make install requires python3.9 and python-pip)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:563edc677ddc6909127ced1723da9838f19b51c93d4ef20da55911fce52fa829
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Flexiplex
  - Python
  - edlib
  - pip
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# edlib-edit-distance-flank-search

## Summary

A re-implementation of Flexiplex's demultiplexing loop: error-tolerant left/right flank searching with edlib to extract the barcode+UMI region, dynamic-programming Levenshtein barcode assignment under a unique-best-match rule, and iterative masking to split chimeric reads carrying multiple barcode constructs.

## When to use

Use when reads (FASTQ/FASTA) must be demultiplexed against a barcode whitelist by error-tolerant sequence matching, or when reproducing Flexiplex's flank-search/Levenshtein-match/chimeric-split logic on simulated long-read data such as the Ebrahimi et al. 2022 reads.

## When NOT to use

- The barcode whitelist is unknown and only flanking sequences are available — that is Flexiplex's discovery mode, not a known-barcode assignment task
- Input is already a gene-by-cell count matrix; downstream whitelist filtering and empty-droplet removal (e.g. flexiplex-filter with 3M-february-2018.txt) applies later, not to this search loop
- Reads require genomic mapping or gene/transcript assignment before barcode extraction, as in the epi2me-labs/wf-single-cell pipeline

## Inputs

- Simulated long-read single-cell FASTQ reads (reads.fastq format, e.g. Ebrahimi et al. 2022 simulations)
- Barcode whitelist text file (barcode_list.txt format)
- Left and right flanking sequences flanking the barcode+UMI region
- User-specified edit-distance tolerance for flank search (analogous to flexiplex -f)

## Outputs

- Per-read barcode assignments and barcode counts written to flexiplex_barcodes_counts.txt
- Filtered FASTQ of demultiplexed reads (analogous to flexiplex stdout new_reads.fastq)
- Correct-barcode assignment rate comparable to Fig. 1D of the Flexiplex paper

## How to apply

Parse the simulated long-read FASTQ and barcode whitelist (mirroring flexiplex's reads.fastq / barcode_list.txt inputs), then search each read for the left and right flanking sequences with edlib at a user-specified edit-distance tolerance (analogous to flexiplex's -f parameter), extracting the intervening barcode+UMI sequence. Match each extracted barcode against the full barcode list with a dynamic-programming Levenshtein aligner, reporting an assignment only when the minimal edit distance is attained by exactly one barcode (default tolerance of two edits, and no other barcode with an equally lowest distance). After a hit, mask the matched region and re-search the remaining read sequence to split chimeric reads carrying multiple barcode constructs. Write per-read assignments and barcode counts to flexiplex_barcodes_counts.txt and validate the correct-barcode assignment rate against Fig. 1D of the Flexiplex paper.

## Related tools

- **Flexiplex** (Reference implementation being reconstructed: flank search with -f edit-distance parameter, Levenshtein-based barcode matching, and chimeric-read splitting) — https://davidsongroup.github.io/flexiplex/
- **Python** (Language in which the FASTQ parsing, edlib flank search, Levenshtein dynamic-programming aligner, and masking loop are implemented)
- **edlib** (Python library used for error-tolerant search of the left/right flanking sequences within each read)
- **pip** (Installation path for the Python dependencies of the reconstructed module (make install requires python3.9 and python-pip))

## Examples

```
flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- Every reported barcode assignment satisfies the unique-best-match criterion: exactly one barcode attains the minimal Levenshtein distance (no ties), within the default edit distance of two
- Correct-barcode assignment rate on the simulated Ebrahimi et al. 2022 reads is consistent with the accuracy reported in Fig. 1D of the Flexiplex paper
- Each reported assignment's extracted barcode+UMI lies strictly between a valid left-flank and right-flank edlib hit within the specified edit-distance tolerance
- flexiplex_barcodes_counts.txt contains both per-read assignments and barcode counts, with chimeric reads split into multiple masked-region hits
- Re-running the module is deterministic: identical FASTQ and whitelist inputs yield identical assignment and count files

## Limitations

- A barcode match is withheld whenever two or more barcodes share the equally lowest Levenshtein distance, so ambiguous reads are dropped rather than arbitrarily assigned
- Flank search sensitivity depends on the user-specified edit-distance tolerance (analogous to flexiplex's -f parameter, e.g. -f 0 for exact flank matching); overly loose settings increase spurious barcode+UMI extractions
- Validation is grounded in a simulated dataset (Ebrahimi et al. 2022) and Fig. 1D accuracy values; behaviour on real noisy long reads may differ

## Evidence

- [abstract] Levenshtein-based unique-best-match rule: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [intro] Demultiplexing scope: "Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment."
- [methods] Reference CLI invocation: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [methods] Whitelist filtering step: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [readme] Two operating modes: "Flexiplex works in two modes: (i) when one or more sequences of interest are known, such as barcodes, and (ii) discovery mode—when only the sequence which flanks the region of interest is known."
