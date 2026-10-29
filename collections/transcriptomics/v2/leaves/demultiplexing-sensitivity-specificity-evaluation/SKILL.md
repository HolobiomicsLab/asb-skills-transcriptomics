---
name: demultiplexing-sensitivity-specificity-evaluation
description: Use when benchmarking cellular-barcode discovery/demultiplexing tools
  on noisy long-read single-cell FASTQ data and a short-read-derived barcode whitelist
  is available as withheld ground truth for sensitivity/specificity scoring.
when_to_use_negative:
- Barcodes are already known — use known-sequence demultiplexing mode (e.g. 'flexiplex
  -d 10x3v3 -k barcode_list.txt') instead of discovery mode
- Input is a gene-by-cell count matrix or already-demultiplexed BAM — there is no
  barcode discovery left to benchmark
- Data lack 10x-style cellular-barcode flanking structure, so a flanking-sequence-only
  discovery mode and inflection-point droplet filtering do not apply
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Flexiplex
  role: Discovery-mode barcode extraction from noisy long reads with Levenshtein-distance
    imperfect matching ('flexiplex -f 0', four threads, default parameters)
  repo: https://github.com/DavidsonGroup/flexiplex
- name: flexiplex-filter
  role: Filters discovered barcodes against a whitelist and removes empty droplets
    using the inflection point method to produce barcode frequency lists
  repo: https://github.com/DavidsonGroup/flexiplex
- name: BLAZE
  role: Purpose-built comparator for barcode discovery; runtime and accuracy baseline
  repo: https://github.com/shimlab/BLAZE
- name: scTagger
  role: Purpose-built comparator for barcode discovery; runtime and accuracy baseline
- name: wf-single-cell
  role: Full Nextflow pipeline comparator whose compute requirements (min 32 CPUs)
    are benchmarked as an order of magnitude heavier
  repo: epi2me-labs/wf-single-cell
- name: conda
  role: Installs Flexiplex for benchmarking ('conda install -c bioconda -c conda-forge
    flexiplex')
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:9288743c90288334d5c59c64e72e9a333865e339dea8efdf39b4c4d6b64ceb28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - BLAZE
  - Flexiplex
  - conda
  - flexiplex-filter
  - scTagger
  - wf-single-cell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# demultiplexing-sensitivity-specificity-evaluation

## Summary

Benchmarks barcode-discovery and demultiplexing tools (e.g. Flexiplex, BLAZE, scTagger, wf-single-cell) on noisy long-read single-cell RNA-seq FASTQ data by scoring each tool's discovered barcode set for sensitivity and specificity against short-read-derived whitelists treated as withheld truth, alongside wall-clock time and memory measurements.

## When to use

Use when benchmarking cellular-barcode discovery/demultiplexing tools on noisy long-read single-cell FASTQ data and a short-read-derived barcode whitelist is available as withheld ground truth for sensitivity/specificity scoring.

## When NOT to use

- Barcodes are already known — use known-sequence demultiplexing mode (e.g. 'flexiplex -d 10x3v3 -k barcode_list.txt') instead of discovery mode
- Input is a gene-by-cell count matrix or already-demultiplexed BAM — there is no barcode discovery left to benchmark
- Data lack 10x-style cellular-barcode flanking structure, so a flanking-sequence-only discovery mode and inflection-point droplet filtering do not apply

## Inputs

- Noisy long-read single-cell RNA-seq FASTQ replicates (e.g. scmixology 2, hiPSC GridION Q20/GridION/PromethION)
- Short-read-derived barcode whitelists used as withheld truth (e.g. 3M-february-2018.txt)
- Raw discovered-barcode count files (flexiplex_barcodes_counts.txt) from each tool
- Comparator tool outputs (BLAZE, scTagger, wf-single-cell) run on the same replicates

## Outputs

- Filtered barcode frequency lists (filtered.txt per replicate)
- Per-tool wall-clock time and memory-use measurements
- Sensitivity and specificity of each discovered barcode set vs. the withheld whitelist
- benchmark_table.csv comparing sensitivity, specificity, and runtime across tools

## How to apply

Run each discovery-mode tool on the same long-read single-cell replicates (e.g. scmixology 2 and hiPSC GridION Q20/GridION/PromethION FASTQ) with default settings, using four threads — for Flexiplex this is discovery mode with only the flanking sequence known ('flexiplex -f 0 reads.fastq'). Convert each tool's raw barcode output into barcode frequency lists, then filter with flexiplex-filter against the short-read-derived whitelist and remove empty droplets using the inflection point method. Record wall-clock time and memory use for every tool, including heavyweight pipelines like wf-single-cell. Score sensitivity and specificity of each discovered barcode set against the withheld whitelist truth, then assemble sensitivity, specificity, and runtime into a benchmark_table.csv. Validation hinges on comparable accuracy across tools with large runtime differences: Flexiplex completed discovery in roughly 5%–16% of BLAZE's runtime and 25%–42% of scTagger's, while wf-single-cell used an order of magnitude more compute.

## Related tools

- **Flexiplex** (Discovery-mode barcode extraction from noisy long reads with Levenshtein-distance imperfect matching ('flexiplex -f 0', four threads, default parameters)) — https://github.com/DavidsonGroup/flexiplex
- **flexiplex-filter** (Filters discovered barcodes against a whitelist and removes empty droplets using the inflection point method to produce barcode frequency lists) — https://github.com/DavidsonGroup/flexiplex
- **BLAZE** (Purpose-built comparator for barcode discovery; runtime and accuracy baseline) — https://github.com/shimlab/BLAZE
- **scTagger** (Purpose-built comparator for barcode discovery; runtime and accuracy baseline)
- **wf-single-cell** (Full Nextflow pipeline comparator whose compute requirements (min 32 CPUs) are benchmarked as an order of magnitude heavier) — epi2me-labs/wf-single-cell
- **conda** (Installs Flexiplex for benchmarking ('conda install -c bioconda -c conda-forge flexiplex'))

## Examples

```
flexiplex -f 0 reads.fastq; flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt
```

## Evaluation signals

- Flexiplex's discovered-barcode sensitivity and specificity are similar to BLAZE and scTagger against the withheld short-read-derived whitelist
- Flexiplex discovery-mode runtime falls in roughly 5%–16% of BLAZE's runtime and 25%–42% of scTagger's runtime on the same datasets (Fig. 1H)
- wf-single-cell consumes an order of magnitude more compute resources than Flexiplex discovery mode
- flexiplex-filter produces a filtered barcode list (filtered.txt) with empty droplets removed via the inflection point method
- benchmark_table.csv contains sensitivity, specificity, and runtime rows for all tools on every tested replicate

## Limitations

- Sensitivity/specificity are only as trustworthy as the short-read-derived whitelist used as withheld truth
- Benchmark was run with default settings and four threads in discovery mode only; other parameter regimes are not covered
- wf-single-cell is a full end-to-end pipeline (adapter trimming, mapping, UMI counting), so its runtime is not directly comparable to barcode-discovery-only tools

## Evidence

- [methods] Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method.: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [methods] flexiplex-filter whitelist command: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [intro] two modes including discovery mode: "Flexiplex works in two modes: (i) when one or more sequences of interest are known, such as barcodes, and (ii) discovery mode—when only the sequence which flanks the region of interest is known."
- [abstract] accuracy and efficiency balance claim: "Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools"
- [readme] Flexiplex input/output behavior: "Given a set of reads as either FASTQ or FASTA, it will demultiplex and/or identify a sequence of interest, reporting matching reads and read-barcode assignment."
- [readme] wf-single-cell comparator scope: [evidence span withheld — anchor status: external_doc]
