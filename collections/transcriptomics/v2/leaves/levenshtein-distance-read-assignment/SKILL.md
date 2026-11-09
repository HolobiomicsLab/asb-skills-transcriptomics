---
name: levenshtein-distance-read-assignment
description: Use when demultiplexing FASTQ/FASTA reads to a known list of cellular
  barcodes from noisy single-cell long-read data, where sequencing errors mean exact
  barcode matching fails and imperfect (edit-distance-tolerant) matching is required.
when_to_use_negative:
- Only the flanking sequence is known and the barcode itself must be discovered —
  use Flexiplex discovery mode instead of known-barcode assignment
- Input is already a cell x gene count matrix with barcodes assigned — demultiplexing
  is not the appropriate step
- Data are not barcoded single-cell or whitelist-searchable reads (e.g. generic genomic
  alignment tasks) — use a mapper instead
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Flexiplex
  role: 'Primary demultiplexer: Levenshtein-distance-based assignment of reads to
    known cellular barcodes across edit distances 0–3'
  repo: https://davidsongroup.github.io/flexiplex/
- name: scTagger
  role: Comparison tool benchmarked on identical FASTQ, whitelist and edit-distance
    settings
- name: FLAMES
  role: Comparison tool benchmarked on identical FASTQ, whitelist and edit-distance
    settings
  repo: https://github.com/LuyiTian/FLAMES
- name: conda
  role: Installs Flexiplex from bioconda with conda-forge dependencies
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:b8b96cd6dffb396121170bf1c974bdcd0791cc405189bee16999d15914b6e4a1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FLAMES
  - Flexiplex
  - conda
  - scTagger
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# levenshtein-distance-read-assignment

## Summary

Assign reads to cellular barcodes by error-tolerant Levenshtein-distance matching against a known barcode whitelist, as implemented in Flexiplex. Benchmarking on Ebrahimi et al. 2022 simulated 10x long-read data (5M reads, 5000 cells) at maximum edit distances 0–3 shows Flexiplex outperforms scTagger and FLAMES in correct assignments and false discovery rate.

## When to use

Use when demultiplexing FASTQ/FASTA reads to a known list of cellular barcodes from noisy single-cell long-read data, where sequencing errors mean exact barcode matching fails and imperfect (edit-distance-tolerant) matching is required.

## When NOT to use

- Only the flanking sequence is known and the barcode itself must be discovered — use Flexiplex discovery mode instead of known-barcode assignment
- Input is already a cell x gene count matrix with barcodes assigned — demultiplexing is not the appropriate step
- Data are not barcoded single-cell or whitelist-searchable reads (e.g. generic genomic alignment tasks) — use a mapper instead

## Inputs

- FASTQ (or FASTA) single-cell long-read file (e.g. Ebrahimi et al. 2022 simulated 5M reads / 5000 cells)
- Known cellular barcode whitelist file (e.g. barcode_list.txt)
- Chemistry preset / flanking sequence specification (e.g. 10x3v3)

## Outputs

- Demultiplexed reads file (new_reads.fastq) with barcode-corrected reads
- Per-read barcode assignment records and per-run stats
- Benchmark table of correct assignments and FDR per tool per edit distance (e.g. fig1d_benchmark_results.tsv)

## How to apply

Obtain the single-cell FASTQ (e.g. Ebrahimi et al. 2022 simulated 5-million-read/5000-cell dataset) and the true barcode whitelist (barcode_list.txt). Install Flexiplex from bioconda (`conda install -c bioconda -c conda-forge flexiplex`) and run in known-barcode mode with the whitelist supplied via `-k`, sweeping the maximum edit distance setting 0, 1, 2 and 3; use `-d 10x3v3` for 10x Chromium 3' v3 chemistry or equivalent presets for other chemistries. Run comparison tools (scTagger, FLAMES) on the identical FASTQ, whitelist, and edit-distance settings. Score each tool's read-barcode assignments against truth labels, computing correct assignment counts and false discovery rate (false assignments / total assignments) per edit distance, then tabulate (e.g. fig1d_benchmark_results.tsv) or plot as a grouped bar chart mirroring Fig. 1D. Flexiplex's Levenshtein-distance basis means substitutions, insertions and deletions in barcodes are all tolerated, which underpins its accuracy advantage on noisy long reads.

## Related tools

- **Flexiplex** (Primary demultiplexer: Levenshtein-distance-based assignment of reads to known cellular barcodes across edit distances 0–3) — https://davidsongroup.github.io/flexiplex/
- **scTagger** (Comparison tool benchmarked on identical FASTQ, whitelist and edit-distance settings)
- **FLAMES** (Comparison tool benchmarked on identical FASTQ, whitelist and edit-distance settings) — https://github.com/LuyiTian/FLAMES
- **conda** (Installs Flexiplex from bioconda with conda-forge dependencies)

## Examples

```
flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- Flexiplex assigns correct barcodes to more reads than scTagger and FLAMES at every maximum edit distance 0–3
- Flexiplex attains the lowest false discovery rate (false assignments / total assignments) across all edit distances tested
- Reproduced table (fig1d_benchmark_results.tsv) is consistent with the paper's Fig. 1D accuracy advantage under the best-case scenario with true barcodes provided
- Per-run stats from flexiplex report read-barcode assignment counts matching the scored assignment totals

## Limitations

- Benchmark assumes a perfect, known barcode whitelist (best-case scenario); performance under incorrect or incomplete whitelists is not covered
- Levenshtein-distance matching with edit distance > 0 can admit false barcode assignments, so FDR must be monitored as the edit-distance threshold increases
- Chemistry-specific presets (e.g. 10x3v3) must match the input library; wrong presets/flank settings will degrade assignment

## Evidence

- [abstract] Levenshtein basis allowing imperfect matches: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [abstract] Accuracy advantage over leading task-specific tools: "Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools"
- [methods] Known-barcode CLI invocation: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [methods] Install via conda: "You can also install Flexiplex using Anaconda: `conda install -c bioconda -c conda-forge flexiplex`"
- [readme] Assigning reads to cellular barcodes (README): "Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment."
