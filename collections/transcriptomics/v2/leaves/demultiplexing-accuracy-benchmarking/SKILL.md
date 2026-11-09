---
name: demultiplexing-accuracy-benchmarking
description: Use when a task requires reproducing or extending the Fig. 1D-style accuracy
  comparison of demultiplexers on data with true barcode labels — e.g. the Ebrahimi
  et al. 2022 simulated 5-million-read/5000-cell long-read single-cell dataset with
  barcode_list.txt as the whitelist.
when_to_use_negative:
- Barcodes are unknown and only flanking sequences are known — use Flexiplex's discovery
  mode instead of whitelist-based assignment benchmarking
- Input is already a demultiplexed count matrix or read-barcode assignment table —
  scoring is the remaining step, not tool benchmarking
- No ground-truth barcode labels exist for the dataset — correct-assignment counts
  and FDR cannot be computed
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Flexiplex
  role: Demultiplexer under test; run in known-barcode mode per maximum edit distance
    0–3 to produce read-barcode assignments
  repo: https://davidsongroup.github.io/flexiplex/
- name: scTagger
  role: Task-specific long-read single-cell demultiplexer used as a comparator on
    identical FASTQ, whitelist, and edit distance settings
- name: FLAMES
  role: Task-specific single-cell tool used as a comparator with matched inputs and
    edit distance settings
  repo: https://github.com/LuyiTian/FLAMES
- name: conda
  role: Installs Flexiplex from bioconda for the benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:5d16c9541290235c67572969b76194c12ada74b2935a1776fff239a0b7e2cc81
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# demultiplexing-accuracy-benchmarking

## Summary

Benchmarks the accuracy of barcode demultiplexing tools (Flexiplex, scTagger, FLAMES) by running each on identical simulated FASTQ reads with a known barcode whitelist across maximum edit distances 0–3, then scoring assignments against truth labels. It quantifies correct assignment counts and false discovery rate per tool per edit distance.

## When to use

Use when a task requires reproducing or extending the Fig. 1D-style accuracy comparison of demultiplexers on data with true barcode labels — e.g. the Ebrahimi et al. 2022 simulated 5-million-read/5000-cell long-read single-cell dataset with barcode_list.txt as the whitelist.

## When NOT to use

- Barcodes are unknown and only flanking sequences are known — use Flexiplex's discovery mode instead of whitelist-based assignment benchmarking
- Input is already a demultiplexed count matrix or read-barcode assignment table — scoring is the remaining step, not tool benchmarking
- No ground-truth barcode labels exist for the dataset — correct-assignment counts and FDR cannot be computed

## Inputs

- Simulated single-cell long-read dataset as FASTQ (e.g. Ebrahimi et al. 2022, 5 million reads over 5000 cells)
- True cellular barcode whitelist file (e.g. barcode_list.txt)
- Truth labels mapping reads to barcodes for scoring

## Outputs

- fig1d_benchmark_results.tsv: correct assignments and FDR per tool per edit distance (0–3)
- Optional grouped bar chart mirroring Fig. 1D
- Per-run read-barcode assignment files and stats (e.g. new_reads.fastq)

## How to apply

Obtain the simulated FASTQ (5 million reads over 5000 cells) together with the true barcode whitelist (barcode_list.txt) and install Flexiplex via conda (`conda install -c bioconda -c conda-forge flexiplex`). Run each tool in known-barcode mode with the identical FASTQ and whitelist, once per maximum edit distance setting 0, 1, 2, and 3 — for Flexiplex this is `flexiplex -k barcode_list.txt reads.fastq > new_reads.fastq`. Flexiplex's matching is Levenshtein-distance based, so edit distance caps define the tolerance regime being compared. Collect read-barcode assignments and per-run stats, score every tool against the truth labels, and compute correct assignment counts and FDR (false assignments / total assignments) at each edit distance. Tabulate results as a TSV and optionally plot a grouped bar chart mirroring Fig. 1D; the benchmark is a best-case scenario since true barcodes are provided.

## Related tools

- **Flexiplex** (Demultiplexer under test; run in known-barcode mode per maximum edit distance 0–3 to produce read-barcode assignments) — https://davidsongroup.github.io/flexiplex/
- **scTagger** (Task-specific long-read single-cell demultiplexer used as a comparator on identical FASTQ, whitelist, and edit distance settings)
- **FLAMES** (Task-specific single-cell tool used as a comparator with matched inputs and edit distance settings) — https://github.com/LuyiTian/FLAMES
- **conda** (Installs Flexiplex from bioconda for the benchmark)

## Examples

```
conda install -c bioconda -c conda-forge flexiplex && flexiplex -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- Reproduced table shows Flexiplex assigns correct barcodes to more reads than scTagger and FLAMES at every maximum edit distance 0–3
- Flexiplex attains the lowest false discovery rate (false assignments / total assignments) at every edit distance tested
- FDR values fall within [0, 1] and decrease (or stay low) as the maximum edit distance tightens
- Each tool's run is reproducible: identical FASTQ input, identical whitelist, matched edit distance settings 0–3 per tool
- Output TSV contains one row per tool per edit distance with both correct-assignment counts and FDR columns populated

## Limitations

- The benchmark is a best-case scenario — a perfect list of true barcodes is provided, which may not reflect barcode-discovery settings
- Results are on simulated data (Ebrahimi et al. 2022) and may not transfer to real noisy long-read datasets
- Only maximum edit distances 0–3 are covered; higher tolerance regimes are untested here

## Evidence

- [results] Flexiplex reported the correct barcodes for more reads than scTagger and FLAMES across all maximum edit distances tested (0–3) and had the lowest false discovery rate (Fig. 1D).: [evidence span withheld — anchor status: not_found]
- [methods] flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [abstract] Flexiplex is based on the Levenshtein distance and allows imperfect matches: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [abstract] Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools: "Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools"
- [readme] Flexiplex works in two modes: (i) when one or more sequences of interest are known, such as barcodes, and (ii) discovery mode: "Flexiplex works in two modes: (i) when one or more sequences of interest are known, such as barcodes, and (ii) discovery mode"
