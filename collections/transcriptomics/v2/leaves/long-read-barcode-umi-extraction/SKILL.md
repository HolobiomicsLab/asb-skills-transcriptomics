---
name: long-read-barcode-umi-extraction
description: Use when demultiplexing noisy long-read single-cell RNA-seq reads or
  10x Chromium/Visium short reads where barcodes/UMIs must be extracted, corrected,
  and assigned against a whitelist (barcode_list.txt).
when_to_use_negative:
- Only the flanking sequence is unknown and the barcode region itself is fully known
  with no sequencing errors — a simple exact grep-like search suffices, since Flexiplex's
  value is error-tolerant Levenshtein matching
- Discovery mode is requested but no flanking sequence is known — Flexiplex requires
  at least the sequence flanking the region of interest
- Data are already demultiplexed with barcode/UMI tags in the read headers (e.g. downstream
  output of wf-single-cell), so re-extraction would duplicate the workflow
tools:
- name: Flexiplex
  role: 'Primary demultiplexer: flank-search, Levenshtein barcode matching, and chimeric
    read splitting for barcode/UMI extraction'
  repo: https://davidsongroup.github.io/flexiplex/
- name: flexiplex-filter
  role: Post-hoc filtering of assigned barcodes against a 10x whitelist and empty-droplet
    removal via inflection point
  repo: https://github.com/DavidsonGroup/flexiplex
- name: wf-single-cell
  role: Alternative Nextflow workflow context for cell barcode & UMI identification
    from 10x single-cell data (adapter trimming, read splitting, barcode/UMI correction)
  repo: https://github.com/epi2me-labs/wf-single-cell
- name: conda
  role: Installation channel for Flexiplex via bioconda
- name: Python 3.9 with pip
  role: Runtime dependency for installing Flexiplex (make install) and for re-implementing
    the flank-search/Levenshtein loop in Python
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:39c277a4ec88c2dc62373afa21951c4f5794ce9c704cfef6ee50b00db3ca69f1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Flexiplex
  - Python
  - Python 3.9 with pip
  - conda
  - flexiplex-filter
  - pip
  - wf-single-cell
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# long-read-barcode-umi-extraction

## Summary

Extracts and assigns cellular barcodes and UMIs from single-cell short-read or noisy long-read FASTQ/FASTA data using Flexiplex, a Levenshtein-distance-based demultiplexer that permits imperfect matches and supports both known-barcode and barcode-discovery modes.

## When to use

Use when demultiplexing noisy long-read single-cell RNA-seq reads or 10x Chromium/Visium short reads where barcodes/UMIs must be extracted, corrected, and assigned against a whitelist (barcode_list.txt).

## When NOT to use

- Only the flanking sequence is unknown and the barcode region itself is fully known with no sequencing errors — a simple exact grep-like search suffices, since Flexiplex's value is error-tolerant Levenshtein matching
- Discovery mode is requested but no flanking sequence is known — Flexiplex requires at least the sequence flanking the region of interest
- Data are already demultiplexed with barcode/UMI tags in the read headers (e.g. downstream output of wf-single-cell), so re-extraction would duplicate the workflow

## Inputs

- Read set in FASTQ or FASTA format (e.g. reads.fastq, simulated long-read single-cell reads)
- Barcode whitelist file (barcode_list.txt or 10x 3M-february-2018.txt)
- Chemistry preset string (e.g. 10x3v3) or user-specified flanking sequences and edit-distance tolerance

## Outputs

- Demultiplexed reads with barcode/UMI tags (new_reads.fastq)
- Per-read barcode assignments and barcode counts (flexiplex_barcodes_counts.txt)
- Whitelist-filtered barcode list from flexiplex-filter (filtered.txt)

## How to apply

Run Flexiplex over FASTQ/FASTA reads using a chemistry preset (e.g. -d 10x3v3) plus a barcode list with -k barcode_list.txt; the tool searches flank sequences at a user-specified edit-distance tolerance (-f, default edit distance 2) and extracts the intervening barcode+UMI sequence. Extracted barcodes are matched to the whitelist by Levenshtein distance, allowing imperfect matches; a match is reported only when the minimal edit distance is attained by a unique best barcode. For discovery from long reads, run with -f 0 to find unknown barcodes between known flanking sequences, and mask/re-search the remaining read to split chimeric reads carrying multiple barcode constructs. Optionally apply flexiplex-filter against a 10x whitelist (e.g. 3M-february-2018.txt) to remove empty droplets using an inflection point method, then compare assignment rates against benchmark figures such as Fig. 1D.

## Related tools

- **Flexiplex** (Primary demultiplexer: flank-search, Levenshtein barcode matching, and chimeric read splitting for barcode/UMI extraction) — https://davidsongroup.github.io/flexiplex/
- **flexiplex-filter** (Post-hoc filtering of assigned barcodes against a 10x whitelist and empty-droplet removal via inflection point) — https://github.com/DavidsonGroup/flexiplex
- **wf-single-cell** (Alternative Nextflow workflow context for cell barcode & UMI identification from 10x single-cell data (adapter trimming, read splitting, barcode/UMI correction)) — https://github.com/epi2me-labs/wf-single-cell
- **conda** (Installation channel for Flexiplex via bioconda)
- **Python 3.9 with pip** (Runtime dependency for installing Flexiplex (make install) and for re-implementing the flank-search/Levenshtein loop in Python)

## Examples

```
flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- Every reported barcode assignment satisfies the unique-best-match rule: exactly one barcode attains the minimal Levenshtein distance (no ties)
- Correct-barcode assignment rate on simulated long-read data is consistent with Flexiplex accuracy reported in Fig. 1D
- flexiplex_barcodes_counts.txt contains one assignment per matching read plus barcode counts consistent with the number of demultiplexed reads in new_reads.fastq
- Chimeric reads yield multiple barcode constructs, confirming the iterative mask-and-research splitting loop fired
- flexiplex-filter output (filtered.txt) reflects the expected empty-droplet inflection point cutoff on the barcode count distribution

## Limitations

- Flank search tolerance is user-controlled via the -f edit-distance parameter; a too-loose tolerance can extract spurious barcode+UMI intervals
- Barcode matching uses Levenshtein (indel-permitting) distance with default edit distance 2 — assignments with equal-minimal-distance ties are ambiguous and must not be reported
- Barcode discovery mode requires the flanking sequences to be known; it cannot operate on completely unknown structure

## Evidence

- [abstract] Flexiplex is based on the Levenshtein distance and allows imperfect matches: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [methods] Demultiplexing command with preset and barcode list: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [methods] Whitelist filtering and empty droplet removal: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [readme] Tool definition and modes: "Flexiplex works in two modes: (i) when one or more sequences of interest are known, such as barcodes, and (ii) discovery mode—when only the sequence which flanks the region of interest is known."
- [readme] wf-single-cell workflow scope: [evidence span withheld — anchor status: not_found]
