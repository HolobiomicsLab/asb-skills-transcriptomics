---
name: simulated-read-truth-evaluation
description: Use when comparing barcode demultiplexing tools on simulated single-cell
  long-read data where ground-truth read-barcode labels and a true barcode whitelist
  are available, and accuracy/FDR must be scored per edit-distance setting.
when_to_use_negative:
- The data are real (non-simulated) reads without ground-truth barcode labels — accuracy
  and FDR cannot be scored against truth, so a whitelist-filter or downstream-count-matrix
  evaluation is needed instead
- Barcodes are unknown and must be discovered from noisy long reads — this is Flexiplex's
  discovery mode (-f), not a known-whitelist benchmark
- The comparison targets a different read modality (e.g. 10x short-read droplet data
  with a fixed chemistry preset such as -d 10x3v3) rather than simulated long-read
  FASTQ
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: Flexiplex
  role: Demultiplexer under test; run in known-barcode mode (-k whitelist) at maximum
    edit distances 0-3, reporting read-barcode assignments to score against truth
  repo: https://davidsongroup.github.io/flexiplex/
- name: scTagger
  role: Task-specific long-read single-cell demultiplexer used as a comparator on
    the identical FASTQ, whitelist, and edit-distance settings
- name: FLAMES
  role: Task-specific single-cell demultiplexer used as a comparator on the same simulated
    inputs
  repo: https://github.com/LuyiTian/FLAMES
- name: conda
  role: Installation channel for Flexiplex and dependency management (bioconda/conda-forge)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:b5f183364f730840b0f3cbe5c026ecb5b9014602ed278970a268977bc3f7f601
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
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# simulated-read-truth-evaluation

## Summary

Benchmark demultiplexing accuracy by running each tool on simulated FASTQ reads with a known true barcode whitelist (e.g. the Ebrahimi et al. 2022 5-million-read/5000-cell long-read dataset), scoring assignments against truth labels across maximum edit distances 0-3, and reporting correct-assignment counts and false discovery rate per tool.

## When to use

Use when comparing barcode demultiplexing tools on simulated single-cell long-read data where ground-truth read-barcode labels and a true barcode whitelist are available, and accuracy/FDR must be scored per edit-distance setting.

## When NOT to use

- The data are real (non-simulated) reads without ground-truth barcode labels — accuracy and FDR cannot be scored against truth, so a whitelist-filter or downstream-count-matrix evaluation is needed instead
- Barcodes are unknown and must be discovered from noisy long reads — this is Flexiplex's discovery mode (-f), not a known-whitelist benchmark
- The comparison targets a different read modality (e.g. 10x short-read droplet data with a fixed chemistry preset such as -d 10x3v3) rather than simulated long-read FASTQ

## Inputs

- Simulated single-cell long-read FASTQ with ground-truth barcode labels (e.g. Ebrahimi et al. 2022, 5 million reads / 5000 cells)
- True cellular barcode whitelist file (e.g. barcode_list.txt)

## Outputs

- Per-tool, per-edit-distance table of correct barcode assignment counts and false discovery rate (fig1d_benchmark_results.tsv)
- Optional grouped bar chart mirroring the paper's Fig. 1D
- Per-run tool statistics (read-barcode assignments and stats per edit distance 0-3)

## How to apply

Obtain the simulated FASTQ (e.g. 5 million reads over 5000 cells from Ebrahimi et al. 2022) together with the true cellular barcode whitelist file such as barcode_list.txt, giving a best-case scenario where the true barcodes are known. Install each tool (e.g. `conda install -c bioconda -c conda-forge flexiplex` for Flexiplex) and run every tool in known-barcode mode on the identical FASTQ and whitelist, once per maximum edit distance setting 0, 1, 2, and 3 — for Flexiplex this is `flexiplex -k barcode_list.txt reads.fastq > new_reads.fastq`, with scTagger and FLAMES given matching edit-distance settings. Flexiplex's Levenshtein-distance matching allows imperfect barcode matches, so edit distance 0 corresponds to exact matching while 1-3 admit increasing substitution/indel error. Score every tool's read-barcode assignments against the truth labels, computing correct assignment counts and the false discovery rate (false assignments / total assignments) at each edit distance. Tabulate correct assignments and FDR per tool per edit distance (e.g. fig1d_benchmark_results.tsv) and optionally plot as a grouped bar chart mirroring the paper's Fig. 1D.

## Related tools

- **Flexiplex** (Demultiplexer under test; run in known-barcode mode (-k whitelist) at maximum edit distances 0-3, reporting read-barcode assignments to score against truth) — https://davidsongroup.github.io/flexiplex/
- **scTagger** (Task-specific long-read single-cell demultiplexer used as a comparator on the identical FASTQ, whitelist, and edit-distance settings)
- **FLAMES** (Task-specific single-cell demultiplexer used as a comparator on the same simulated inputs) — https://github.com/LuyiTian/FLAMES
- **conda** (Installation channel for Flexiplex and dependency management (bioconda/conda-forge))

## Examples

```
flexiplex -k barcode_list.txt reads.fastq > new_reads.fastq
```

## Evaluation signals

- The reproduced table shows Flexiplex assigns correct barcodes to more reads than scTagger and FLAMES at every maximum edit distance 0-3, consistent with the paper's Fig. 1D
- Flexiplex attains the lowest false discovery rate (false assignments / total assignments) at all tested edit distances
- Every tool run uses the identical FASTQ input, the same true barcode whitelist, and matching maximum edit-distance settings 0-3, so results are directly comparable
- Correct-assignment counts and FDR values are reported for each tool at each of the four edit-distance settings (no missing cells in the results table)
- At edit distance 0 (exact matching) all tools should assign barcodes to near-complete agreement with truth; large discrepancies indicate a misconfigured whitelist or parsing error

## Limitations

- The benchmark is a best-case scenario: true barcodes are provided, which does not measure barcode-discovery performance from unknown barcodes
- Results are specific to the simulated Ebrahimi et al. 2022 dataset (5M reads / 5000 cells); accuracy rankings may differ on real noisy long reads
- Flexiplex matches barcodes via Levenshtein distance allowing imperfect matches, so FDR behavior depends on the chosen maximum edit distance cutoff

## Evidence

- [results] Accuracy advantage over task-specific tools at all edit distances: [evidence span withheld — anchor status: not_found]
- [abstract] Levenshtein-based imperfect matching underpins edit-distance settings: "which is based on the Levenshtein distance and thus allows imperfect matches"
- [abstract] Balance claim to verify in the reproduced benchmark: "Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools"
- [methods] Known-barcode invocation used for each edit-distance run: "flexiplex -d 10x3v3 -k barcode_list.txt reads.fastq > new_reads.fastq"
- [methods] Installation route for reproducing the benchmark: "You can also install Flexiplex using Anaconda: `conda install -c bioconda -c conda-forge flexiplex`"
