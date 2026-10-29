---
name: barcode-discovery-from-long-reads
description: Use when cellular barcodes must be discovered de novo from noisy long-read
  single-cell RNA-seq data (e.g. ONT reads) where only the flanking sequence of the
  barcode region is known and no barcode list is available up front.
when_to_use_negative:
- Barcode list is already known — use flexiplex in known-sequence mode with a preset
  chemistry (e.g. `flexiplex -d 10x3v3 -k barcode_list.txt`) instead of discovery
  mode.
- Input is short-read data with exact-match barcodes and no sequencing error in the
  barcode region — the Levenshtein-distance search is designed for noisy long reads
  with imperfect matches.
- The barcode region has no known flanking sequence on either side — discovery mode
  requires the sequence flanking the region of interest.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: flexiplex
  role: Runs barcode discovery in `-f 0` mode on noisy long reads, reporting matching
    reads and read-barcode assignment based on Levenshtein distance
  repo: https://github.com/DavidsonGroup/flexiplex
- name: flexiplex-filter
  role: Filters discovered barcodes against a whitelist and removes empty droplets
    using the inflection point method
  repo: https://github.com/DavidsonGroup/flexiplex
- name: conda
  role: Installation channel for Flexiplex (bioconda/conda-forge)
- name: BLAZE / scTagger / wf-single-cell
  role: Comparator tools for runtime and sensitivity/specificity benchmarking of discovered
    barcode sets
  repo: epi2me-labs/wf-single-cell
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:ca4803c2fd7fdb9ecf6065fef3ce60ede8ed9d2744d99b8e1b7f0473c922e657
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - BLAZE / scTagger / wf-single-cell
  - Flexiplex
  - conda
  - flexiplex
  - flexiplex-filter
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# barcode-discovery-from-long-reads

## Summary

Discover cellular barcodes de novo from noisy long-read single-cell FASTQ/FASTA data using Flexiplex in discovery mode (Levenshtein-distance search from known flanking sequences), then filter discovered barcodes against a whitelist and remove empty droplets with flexiplex-filter's inflection point method.

## When to use

Use when cellular barcodes must be discovered de novo from noisy long-read single-cell RNA-seq data (e.g. ONT reads) where only the flanking sequence of the barcode region is known and no barcode list is available up front.

## When NOT to use

- Barcode list is already known — use flexiplex in known-sequence mode with a preset chemistry (e.g. `flexiplex -d 10x3v3 -k barcode_list.txt`) instead of discovery mode.
- Input is short-read data with exact-match barcodes and no sequencing error in the barcode region — the Levenshtein-distance search is designed for noisy long reads with imperfect matches.
- The barcode region has no known flanking sequence on either side — discovery mode requires the sequence flanking the region of interest.

## Inputs

- Long-read single-cell FASTQ or FASTA file of raw cellular reads (e.g. ONT scmixology 2 or hiPSC GridION/Q20/PromethION replicates)
- Short-read-derived barcode whitelist for filtering (e.g. 10x 3M-february-2018.txt)
- Known flanking sequences of the barcode region (discovery mode requires only the flanking sequence)

## Outputs

- flexiplex_barcodes_counts.txt — discovered barcodes with read-barcode assignments and counts
- filtered.txt — whitelist-matching barcodes after empty-droplet removal via inflection point method
- new_reads.fastq — reads demultiplexed/assigned to the filtered barcode list (optional second pass)
- Barcode frequency lists and benchmark_table.csv of sensitivity, specificity, and runtime when used for benchmarking

## How to apply

Run flexiplex in discovery mode with `flexiplex -f 0 reads.fastq` (four threads, default parameters), which searches for barcodes based on the Levenshtein distance from known flanking sequences and writes read-barcode assignments and counts to flexiplex_barcodes_counts.txt. Then run `flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt` to filter discovered barcodes against a short-read-derived whitelist (e.g. the 10x 3M-february-2018.txt list) and remove empty droplets using the inflection point method, producing filtered.txt. Optionally, re-run flexiplex with a preset chemistry and the filtered barcode list, e.g. `flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq`, to assign all reads to the discovered cells. Benchmarks on scmixology 2 (~25M ONT reads, 248-cell whitelist) and hiPSC GridION/Q20/PromethION replicates showed Flexiplex completes discovery in roughly 5%–16% of BLAZE's runtime and 25%–42% of scTagger's runtime with comparable sensitivity/specificity.

## Related tools

- **flexiplex** (Runs barcode discovery in `-f 0` mode on noisy long reads, reporting matching reads and read-barcode assignment based on Levenshtein distance) — https://github.com/DavidsonGroup/flexiplex
- **flexiplex-filter** (Filters discovered barcodes against a whitelist and removes empty droplets using the inflection point method) — https://github.com/DavidsonGroup/flexiplex
- **conda** (Installation channel for Flexiplex (bioconda/conda-forge))
- **BLAZE / scTagger / wf-single-cell** (Comparator tools for runtime and sensitivity/specificity benchmarking of discovered barcode sets) — epi2me-labs/wf-single-cell

## Examples

```
flexiplex -f 0 reads.fastq && flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt
```

## Evaluation signals

- flexiplex_barcodes_counts.txt exists and serves as valid input to flexiplex-filter (file_exists checks on new_reads.fastq, flexiplex_barcodes_counts.txt, and filtered.txt)
- filtered.txt contains only whitelist-matching barcodes after empty-droplet removal
- Sensitivity and specificity of the discovered barcode set against the withheld short-read-derived barcode truth are comparable to BLAZE and scTagger
- On scmixology 2, the majority of reads have concordant cell-line assignment with <5% of reads having discordant cell lines
- Wall-clock runtime falls in the expected ratio: roughly 5%–16% of BLAZE's runtime and 25%–42% of scTagger's runtime with four threads and default settings

## Limitations

- Discovery mode requires the sequence flanking the region of interest to be known; if no flanking sequence is known, discovery is not possible.
- Whitelist filtering depends on the availability of a short-read-derived barcode truth (e.g. 3M-february-2018.txt); without it, empty-droplet removal relies on the inflection point method applied to barcode frequencies.
- Comparators consume far more compute: wf-single-cell uses an order of magnitude more compute resources than Flexiplex for the same task.

## Evidence

- [readme] Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method.: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [methods] flexiplex-filter command: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [readme] two modes: "Flexiplex works in two modes: (i) when one or more sequences of interest are known, such as barcodes, and (ii) discovery mode"
- [abstract] fast and versatile tool: "we developed Flexiplex, a versatile and fast sequence searching and demultiplexing tool for omics data"
- [abstract] Levenshtein distance basis: "which is based on the Levenshtein distance and thus allows imperfect matches"
