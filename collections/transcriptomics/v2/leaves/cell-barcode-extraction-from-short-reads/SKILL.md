---
name: cell-barcode-extraction-from-short-reads
description: Use when reads are FASTQ/FASTA and a known sequence of interest (barcode,
  fusion, variant) must be located with tolerated sequencing errors, and reads must
  be assigned to cell barcodes for single-cell quantification or genotyping.
when_to_use_negative:
- The target sequence can be matched exactly and raw speed on uncompressed text is
  the only concern — grep (1–2 min) is faster than Flexiplex (~24 min), though it
  cannot tolerate sequencing errors.
- The input is already demultiplexed/assigned (e.g. a BAM tagged with cell barcodes
  and UMIs from a workflow such as epi2me-labs/wf-single-cell) and no sequence search
  is required.
- Only flanking sequence is known and the barcode must be discovered de novo from
  long reads — this is Flexiplex's discovery mode, a different sub-task than known-sequence
  search in short reads.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Flexiplex
  role: 'Primary tool: error-tolerant, Levenshtein-distance-based sequence search
    and demultiplexer assigning reads to cell barcodes; run with -x <target> -d grep
    -f 2 (edit distance 2) for grep-like search'
  repo: https://github.com/DavidsonGroup/flexiplex
- name: flexiplex-filter
  role: Filters Flexiplex barcode counts against a 10x whitelist (3M-february-2018.txt)
    after demultiplexing
  repo: https://github.com/DavidsonGroup/flexiplex
- name: grep
  role: Exact-match baseline for target sequence search; faster (~1–2 min) but intolerant
    of sequencing errors
- name: ugrep
  role: Baseline searcher with fuzzy matching option (-Z, Levenshtein distance); benchmark
    comparator (~40 min on the short-read data)
  repo: https://github.com/Genivia/ugrep
- name: seqkit
  role: seqkit grep baseline for FASTQ sequence search; slowest comparator (~190 min)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:08e64f3e49bd7014f4dec675d57dc667b11a8421cb4502ed81e7f3fed55f3179
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - Flexiplex
  - flexiplex-filter
  - grep
  - seqkit
  - ugrep
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cell-barcode-extraction-from-short-reads

## Summary

Error-tolerant identification of known target sequences (cell barcodes, fusion junctions such as BCAS4-BCAS3, viral sequences such as Adenovirus E1A, or variants such as rs878887783) in FASTQ/FASTA omics reads using Flexiplex's Levenshtein-distance search, followed by tallying of cell-barcode assignments against the matched reads.

## When to use

Use when reads are FASTQ/FASTA and a known sequence of interest (barcode, fusion, variant) must be located with tolerated sequencing errors, and reads must be assigned to cell barcodes for single-cell quantification or genotyping.

## When NOT to use

- The target sequence can be matched exactly and raw speed on uncompressed text is the only concern — grep (1–2 min) is faster than Flexiplex (~24 min), though it cannot tolerate sequencing errors.
- The input is already demultiplexed/assigned (e.g. a BAM tagged with cell barcodes and UMIs from a workflow such as epi2me-labs/wf-single-cell) and no sequence search is required.
- Only flanking sequence is known and the barcode must be discovered de novo from long reads — this is Flexiplex's discovery mode, a different sub-task than known-sequence search in short reads.

## Inputs

- Illumina short-read single-cell RNA-seq FASTQ dataset (e.g. Chen et al. 2021)
- Known target nucleotide sequences (BCAS4-BCAS3 fusion, Adenovirus E1A, rs878887783) as FASTA or plain strings
- Cell barcode list file (barcode_list.txt) for known-barcode demultiplexing mode
- 10x barcode whitelist (e.g. 3M-february-2018.txt) for flexiplex-filter

## Outputs

- Filtered/demultiplexed FASTQ with read-barcode assignments (new_reads.fastq)
- Flexiplex barcode counts file for whitelist filtering (flexiplex_barcodes_counts.txt)
- Cells-in-correct-cluster tallies per tool and target (cells_in_correct_clusters.tsv) with runtimes

## How to apply

Run Flexiplex in its error-tolerant grep-like mode against each single target sequence with an edit distance of 2 (e.g. flexiplex -x <target> -d grep -f 2 on the Illumina short-read FASTQ), which matches imperfectly via Levenshtein distance where exact-match grep fails. For barcode demultiplexing rather than sequence search, use chemistry presets such as 10x3v3 with a barcode list (flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq), and optionally filter barcodes against a whitelist (e.g. 3M-february-2018.txt) with flexiplex-filter. Benchmark against grep, ugrep, and seqkit grep on the same FASTQ, recording wall-clock runtimes. Extract cell barcodes from matched reads per tool/target, tally cells assigned to the correct expressing cluster, and write results to a table such as cells_in_correct_clusters.tsv. The rationale is that Flexiplex balances accuracy and computational efficiency: it tolerates sequencing errors that exact-matching grep cannot, while running faster than ugrep (~40 min) or seqkit grep (~190 min) on the same short-read single-cell data.

## Related tools

- **Flexiplex** (Primary tool: error-tolerant, Levenshtein-distance-based sequence search and demultiplexer assigning reads to cell barcodes; run with -x <target> -d grep -f 2 (edit distance 2) for grep-like search) — https://github.com/DavidsonGroup/flexiplex
- **flexiplex-filter** (Filters Flexiplex barcode counts against a 10x whitelist (3M-february-2018.txt) after demultiplexing) — https://github.com/DavidsonGroup/flexiplex
- **grep** (Exact-match baseline for target sequence search; faster (~1–2 min) but intolerant of sequencing errors)
- **ugrep** (Baseline searcher with fuzzy matching option (-Z, Levenshtein distance); benchmark comparator (~40 min on the short-read data)) — https://github.com/Genivia/ugrep
- **seqkit** (seqkit grep baseline for FASTQ sequence search; slowest comparator (~190 min))

## Examples

```
flexiplex -x <target> -d grep -f 2 reads.fastq > matched_reads.fastq
```

## Evaluation signals

- Cells-in-correct-cluster counts reproduce reported values: 365/956 (BCAS4-BCAS3), 439/869 (Adenovirus E1A), 515/987 (rs878887783)
- BCAS4-BCAS3 reference-allele recovery reaches 97%
- Flexiplex runtime is ~24 min per target, substantially below seqkit grep (~190 min) and ugrep (~40 min)
- Every matched read in the output FASTQ carries the target sequence within the allowed edit distance of 2
- Per-tool/target cell tallies differ from exact-match grep results because Flexiplex recovers reads containing sequencing errors

## Limitations

- Edit-distance-2 search trades runtime for error tolerance: exact-match grep is 10–100x faster when reads are error-free, so Flexiplex's advantage exists only when sequencing errors must be tolerated.
- The grep-like search mode handles a single sequence per run, so multiple targets (fusion, virus, variant) each require a separate invocation.
- Performance and accuracy claims are benchmarked on Illumina short-read scRNA-seq (Chen et al. 2021); other chemistries/assays may behave differently.

## Evidence

- [abstract] Flexiplex as versatile search/demultiplex tool: "we developed Flexiplex, a versatile and fast sequence searching and demultiplexing tool for omics data"
- [abstract] Levenshtein-distance matching rationale: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [methods] Error-tolerant grep-like search mode: "Flexiplex can perform an error-tolerant grep-like search for a single sequence."
- [methods] Whitelist filtering command: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [readme] Known-barcode demultiplexing invocation from README: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
