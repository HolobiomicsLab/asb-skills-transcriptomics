---
name: cross-tool-quantification-comparison
description: Use when comparing the runtime or throughput of competing read-counting/quantification
  tools on the same SAM/BAM alignment files and reference annotations, e.g. to decide
  whether a specialized counter's overhead is acceptable.
when_to_use_negative:
- The goal is count accuracy or feature assignment comparison rather than runtime
  benchmarking — this skill measures wall-clock time only, not counting equivalence.
- Inputs are already gene/feature count matrices — no alignment-level counting step
  remains to benchmark.
- Only one counting tool is available; a cross-tool comparison requires at least two
  tools run on identical inputs with equivalent parameters.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: tiny-count
  role: Counting tool under benchmark; quantifies multiple small RNA classes in parallel
    with hierarchical classification
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: HTSeq-count
  role: Baseline counting tool run with equivalent counting parameters for runtime
    comparison
- name: featureCounts
  role: Fastest baseline counting tool in the comparison (2.4 s on collapsed SAM in
    the published benchmark)
- name: Python
  role: Used to assemble the recorded wall-clock times into the runtime_benchmark.csv
    table and compute runtime ordering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:5a44ad5199dd7f823507b35f8f69c5d43bd98b721eae32d162feac74ecb3b9a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - HTSeq-count
  - Python
  - featureCounts
  - tiny-count
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cross-tool-quantification-comparison

## Summary

Benchmarks the wall-clock runtime of alternative read-counting tools (tiny-count, HTSeq-count, featureCounts) on identical alignment inputs (collapsed vs uncollapsed SAM files) to quantify performance trade-offs while quantifying multiple small RNA classes in parallel.

## When to use

Use when comparing the runtime or throughput of competing read-counting/quantification tools on the same SAM/BAM alignment files and reference annotations, e.g. to decide whether a specialized counter's overhead is acceptable.

## When NOT to use

- The goal is count accuracy or feature assignment comparison rather than runtime benchmarking — this skill measures wall-clock time only, not counting equivalence.
- Inputs are already gene/feature count matrices — no alignment-level counting step remains to benchmark.
- Only one counting tool is available; a cross-tool comparison requires at least two tools run on identical inputs with equivalent parameters.

## Inputs

- Collapsed SAM alignment file
- Uncollapsed SAM alignment file
- Reference feature annotation (GFF3/GFF2/GTF with ID/gene_id/Parent attributes)
- tiny-count, HTSeq-count, and featureCounts installations

## Outputs

- runtime_benchmark.csv with columns: tool, input type (collapsed vs uncollapsed), runtime in seconds
- Runtime ordering across the three counting tools

## How to apply

Load the collapsed and uncollapsed SAM alignment files plus the reference feature annotation (GFF3) supplied as inputs, and run each counting tool on both input types with equivalent counting parameters. Run tiny-count on the uncollapsed SAM file and then on the collapsed SAM file, recording wall-clock time per run; repeat with HTSeq-count and featureCounts using settings that match tiny-count's counting behavior. Assemble recorded times into a table with columns for tool, input type (collapsed vs uncollapsed), and runtime in seconds, then compute the runtime ordering across the three tools and write it to runtime_benchmark.csv. Interpret runtimes against the published reference point — collapsed tiny-count at 45.1 s versus featureCounts at 2.4 s — noting that tiny-count's performance 'approaches' featureCounts only relative to its functional advantage of quantifying multiple small RNA classes (e.g. piRNAs, siRNAs, miRNAs/isomiRs) in parallel. Because absolute times are hardware-dependent, verify the ordering rather than the exact second counts on your benchmark machine.

## Related tools

- **tiny-count** (Counting tool under benchmark; quantifies multiple small RNA classes in parallel with hierarchical classification) — https://github.com/MontgomeryLab/tinyRNA
- **HTSeq-count** (Baseline counting tool run with equivalent counting parameters for runtime comparison)
- **featureCounts** (Fastest baseline counting tool in the comparison (2.4 s on collapsed SAM in the published benchmark))
- **Python** (Used to assemble the recorded wall-clock times into the runtime_benchmark.csv table and compute runtime ordering)

## Examples

```
conda install -c bioconda -c conda-forge tiny-count; /usr/bin/time -v tiny-count --features-sheet Features.tsv --samples-sheet Samples.tsv -c config.yaml -o runtime_benchmark_out
```

## Evaluation signals

- runtime_benchmark.csv exists and contains one row per tool x input-type combination (3 tools x 2 input types = 6 rows) with numeric runtime values
- The recorded runtime ordering matches the published pattern: featureCounts fastest, tiny-count intermediate-approaching, on collapsed SAM inputs
- Each tool was run with equivalent counting parameters on the same two SAM files and the same reference annotation, so the comparison is like-for-like
- Wall-clock times were recorded per run (not summed or averaged across tools), and the collapsed-vs-uncollapsed distinction is preserved in the table
- Reported absolute times are qualified as hardware-dependent; ordering, not exact seconds, is the reproducible claim

## Limitations

- Absolute runtimes are hardware-dependent: the published 45.1 s (tiny-count) vs 2.4 s (featureCounts) figures were obtained on the article's benchmark hardware and will differ elsewhere; only the ordering is expected to reproduce
- tiny-count's slower runtime is the trade-off for its single-nucleotide-precision hierarchical classification (resolving piRNAs/siRNAs from the same locus and miRNA/isomiR variants), which the faster generic counters do not perform
- tiny-count (and the full tinyRNA workflow) is implemented in Python, C++, Cython, and R coordinated with CWL, so installation issues can arise, notably on Apple Silicon–based systems with stable releases

## Evidence

- [results] On SAM files containing collapsed reads, tiny-count's performance approaches that of featureCounts while quantifying multiple small RNA classes in parallel, at 45.1 s versus 2.4 s.: [evidence span withheld — anchor status: not_found]
- [abstract] tiny-count can distinguish small RNA variants such as miRNAs and isomiRs with single-nucleotide precision: "It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision."
- [abstract] tinyRNA implemented in Python, C++, Cython, and R coordinated with CWL: "tiny-count and other tinyRNA tools are implemented in Python, C++, Cython, and R, and the workflow is coordinated with CWL."
- [readme] README: stable-release installation issues on Apple Silicon: [evidence span withheld — anchor status: external_doc]
