---
name: counting-tool-runtime-benchmarking
description: Use when comparing the computational performance (wall-clock runtime)
  of feature-counting tools on the same SAM/BAM alignment inputs, especially small
  RNA-seq reads with collapsed duplicates.
when_to_use_negative:
- Input is already a count matrix or feature table — there are no alignment files
  left to time
- The goal is to evaluate counting accuracy or classification precision (e.g. isomiR
  resolution) rather than runtime; benchmarking times say nothing about single-nucleotide
  precision
- The analysis goal is aligner benchmarking (e.g. Bowtie/STAR) rather than post-alignment
  counting-tool comparison
edam_operation: http://edamontology.org/operation_2990
tools:
- name: tiny-count
  role: Benchmarked counting tool; hierarchical classification and quantification
    of small RNA reads, timed on collapsed and uncollapsed SAM files
  repo: github.com/MontgomeryLab/tinyRNA
- name: HTSeq-count
  role: Comparison baseline; run on the same SAM files with equivalent counting parameters
- name: featureCounts
  role: Performance reference tool; the fastest counting runtime against which tiny-count's
    speed is judged
- name: Python
  role: Used to assemble recorded wall-clock times into the runtime benchmark table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:d6a08bb0c295d26b40dbec161c2a6d6e7a6f51823d3276d4fb3af8c1c0e6373f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HTSeq-count
  - Python
  - featureCounts
  - tiny-count
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# counting-tool-runtime-benchmarking

## Summary

Benchmarks wall-clock counting runtime of small RNA-seq quantification tools (tiny-count, HTSeq-count, featureCounts) on identical SAM alignment inputs, comparing collapsed versus uncollapsed read files. Establishes that tiny-count approaches featureCounts performance on collapsed SAMs while quantifying multiple small RNA classes in parallel.

## When to use

Use when comparing the computational performance (wall-clock runtime) of feature-counting tools on the same SAM/BAM alignment inputs, especially small RNA-seq reads with collapsed duplicates.

## When NOT to use

- Input is already a count matrix or feature table — there are no alignment files left to time
- The goal is to evaluate counting accuracy or classification precision (e.g. isomiR resolution) rather than runtime; benchmarking times say nothing about single-nucleotide precision
- The analysis goal is aligner benchmarking (e.g. Bowtie/STAR) rather than post-alignment counting-tool comparison

## Inputs

- SAM alignment files, both collapsed and uncollapsed variants
- Reference feature annotation in GFF3/GFF2/GTF format
- Sequencing data inputs (demultiplexed FASTQ) if regenerating alignments

## Outputs

- runtime_benchmark.csv with columns for tool, input type (collapsed vs uncollapsed), and runtime in seconds
- Runtime ordering across tiny-count, HTSeq-count, and featureCounts

## How to apply

Load the same reference feature annotation (GFF/GFF2/GTF/GFF3) and two versions of each alignment input: the uncollapsed SAM file and a collapsed SAM file where duplicate reads are collapsed. Run each tool with equivalent counting parameters on both inputs: tiny-count (Bioconda install, runnable standalone), HTSeq-count, and featureCounts, recording wall-clock time per counting run. Assemble results into a table with columns for tool, input type (collapsed vs uncollapsed), and runtime in seconds, then compute the runtime ordering across the three tools and write it to runtime_benchmark.csv. Reference expectation from the source article is that on collapsed SAM files tiny-count (~45.1 s) approaches featureCounts (~2.4 s), with the caveat that absolute runtimes are hardware dependent, so verify the ordering rather than the absolute numbers.

## Related tools

- **tiny-count** (Benchmarked counting tool; hierarchical classification and quantification of small RNA reads, timed on collapsed and uncollapsed SAM files) — github.com/MontgomeryLab/tinyRNA
- **HTSeq-count** (Comparison baseline; run on the same SAM files with equivalent counting parameters)
- **featureCounts** (Performance reference tool; the fastest counting runtime against which tiny-count's speed is judged)
- **Python** (Used to assemble recorded wall-clock times into the runtime benchmark table)

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- runtime_benchmark.csv exists and contains exactly the columns tool, input type (collapsed vs uncollapsed), and runtime in seconds for all three tools
- On collapsed SAM input, the reported ordering matches the article: featureCounts fastest (~2.4 s), tiny-count approaching it (~45.1 s), with hardware-dependent absolute times
- Each tool was run on both the collapsed and uncollapsed SAM files with equivalent counting parameters, giving a paired comparison
- tiny-count's run covers quantification of multiple small RNA classes in parallel, so its runtime is compared against single-purpose counting, not a reduced workload

## Limitations

- Absolute runtimes are hardware dependent; only the ordering (e.g. collapsed tiny-count ~45.1 s vs featureCounts ~2.4 s) is transferable between systems
- tiny-count's performance claim was made specifically on collapsed SAM files; uncollapsed inputs may show a different relationship
- Installation issues may arise when installing tiny-count/tinyRNA on Apple Silicon-based systems (README caveat for stable releases)

## Evidence

- [results] On SAM files containing collapsed reads, tiny-count's performance approaches that of featureCounts while quantifying multiple small RNA classes in parallel, at 45.1 s versus 2.4 s.: [evidence span withheld — anchor status: not_found]
- [abstract] tiny-count is a highly flexible counting tool: "tiny-count is a highly flexible counting tool that allows for hierarchical classification and quantification of small RNA reads from high-throughput sequencing data."
- [readme] Alternatively, you can install tinyRNA's precision counting tool by itself: [evidence span withheld — anchor status: external_doc]
- [readme] Reference annotations in GFF3/GFF2/GTF: [evidence span withheld — anchor status: not_found]
