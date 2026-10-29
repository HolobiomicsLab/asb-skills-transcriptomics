---
name: false-discovery-rate-calculation
description: Use when evaluating a demultiplexer's read-barcode assignments against
  known truth barcodes and you need an error-rate metric (FDR) plus correct-assignment
  counts, e.g. comparing Flexiplex, scTagger, and FLAMES at edit distances 0–3.
when_to_use_negative:
- When no truth labels or trusted barcode whitelist exist for the reads — FDR cannot
  be computed without a ground-truth reference
- When the goal is barcode discovery from flanking sequences rather than scoring known-barcode
  assignments (Flexiplex's discovery mode is a different task)
- When input is already a per-tool accuracy/FDR summary table rather than raw read-barcode
  assignments
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Flexiplex
  role: Demultiplexer under test; run in known-barcode mode once per maximum edit
    distance 0-3 to produce read-barcode assignments to score
  repo: https://davidsongroup.github.io/flexiplex/
- name: scTagger
  role: Comparison tool; run with identical FASTQ input, same true barcode whitelist,
    and matching edit distances 0-3
- name: FLAMES
  role: Comparison tool; run with identical FASTQ input, same true barcode whitelist,
    and matching edit distances 0-3
  repo: https://github.com/LuyiTian/FLAMES
- name: conda
  role: Installation channel for Flexiplex (bioconda)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:424a19c4e80afbe4283cc3c8e915f347eb4954a7cb8f2e042cca26cefce65aca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FLAMES
  - Flexiplex
  - conda
  - scTagger
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# false-discovery-rate-calculation

## Summary

Computes the false discovery rate of barcode demultiplexing assignments (false assignments / total assignments) by scoring each tool's read-barcode calls against truth labels, used here to benchmark Flexiplex against scTagger and FLAMES across maximum edit distances 0–3 on simulated long-read single-cell data.

## When to use

Use when evaluating a demultiplexer's read-barcode assignments against known truth barcodes and you need an error-rate metric (FDR) plus correct-assignment counts, e.g. comparing Flexiplex, scTagger, and FLAMES at edit distances 0–3.

## When NOT to use

- When no truth labels or trusted barcode whitelist exist for the reads — FDR cannot be computed without a ground-truth reference
- When the goal is barcode discovery from flanking sequences rather than scoring known-barcode assignments (Flexiplex's discovery mode is a different task)
- When input is already a per-tool accuracy/FDR summary table rather than raw read-barcode assignments

## Inputs

- Demultiplexed read-barcode assignment files (per tool, per edit distance) from Flexiplex, scTagger, and FLAMES runs
- Truth labels for the simulated dataset (Ebrahimi et al. 2022, 5 million reads over 5000 cells)
- True cellular barcode whitelist file (e.g. barcode_list.txt)

## Outputs

- Table of correct barcode assignment counts and false discovery rate (false assignments / total assignments) per tool per maximum edit distance (e.g. fig1d_benchmark_results.tsv)
- Optional grouped bar chart mirroring Fig. 1D

## How to apply

Run each demultiplexer (Flexiplex via `flexiplex -k barcode_list.txt reads.fastq > new_reads.fastq`, plus scTagger and FLAMES) on the identical FASTQ input — here the Ebrahimi et al. 2022 simulated 5-million-read/5000-cell dataset — with the same true barcode whitelist and matching maximum edit distance settings 0, 1, 2, and 3. For each tool and each edit distance, score every read-barcode assignment against the truth labels, counting correct assignments and computing FDR as false assignments divided by total assignments. Tabulate correct assignment counts and FDR per tool per edit distance (e.g. fig1d_benchmark_results.tsv) and optionally plot as a grouped bar chart mirroring Fig. 1D. The validation expectation, grounded in the source finding, is that Flexiplex assigns correct barcodes to more reads than scTagger and FLAMES and attains the lowest FDR at every maximum edit distance 0–3.

## Related tools

- **Flexiplex** (Demultiplexer under test; run in known-barcode mode once per maximum edit distance 0-3 to produce read-barcode assignments to score) — https://davidsongroup.github.io/flexiplex/
- **scTagger** (Comparison tool; run with identical FASTQ input, same true barcode whitelist, and matching edit distances 0-3)
- **FLAMES** (Comparison tool; run with identical FASTQ input, same true barcode whitelist, and matching edit distances 0-3) — https://github.com/LuyiTian/FLAMES
- **conda** (Installation channel for Flexiplex (bioconda))

## Examples

```
flexiplex -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- Reproduced table shows Flexiplex assigns correct barcodes to more reads than scTagger and FLAMES at every maximum edit distance 0-3, consistent with the paper's Fig. 1D
- Reproduced table shows Flexiplex attains the lowest false discovery rate (false assignments / total assignments) at every edit distance 0-3
- Per-tool per-edit-distance rows are complete: 3 tools x 4 edit distances = 12 scored conditions, each with a correct-assignment count and an FDR in [0,1]
- Each read-barcode assignment is scored against exactly one truth label from the barcode whitelist, so correct + false assignments = total assignments per condition

## Limitations

- The benchmark is a best-case scenario: true barcodes are provided as the whitelist, so FDR results may overstate accuracy on real data where the whitelist is incomplete or erroneous
- Results are specific to the simulated 5-million-read/5000-cell Ebrahimi et al. 2022 dataset; FDR rankings may differ on real long-read single-cell RNA-seq data with different error profiles

## Evidence

- [results] Best-case FDR finding at all edit distances: [evidence span withheld — anchor status: not_found]
- [methods] Known-barcode invocation used to generate assignments: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [abstract] Flexiplex matches barcodes with imperfect matches, motivating edit-distance sweep: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [abstract] Accuracy-and-efficiency claim that the FDR metric substantiates: "Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools"
- [readme] README confirms demultiplexing output of matching reads and read-barcode assignment that gets scored: "Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment."
