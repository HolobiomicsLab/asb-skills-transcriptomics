---
name: collapsed-uncollapsed-read-input-handling
description: Use when counting small RNA-seq reads from SAM files that exist in both
  collapsed (read + copy-count) and uncollapsed (per-read) forms, or when benchmarking/counting
  runtime across counting tools on these two input representations.
when_to_use_negative:
- Input is already a feature count table — no SAM-level counting or runtime benchmarking
  is needed
- The task is single-nucleotide variant resolution (miRNA/isomiR discrimination) rather
  than input-representation handling or runtime comparison
- Only one input representation (collapsed OR uncollapsed) is available, so the collapsed-vs-uncollapsed
  comparison cannot be performed
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: tiny-count
  role: Small RNA counting tool run on both collapsed and uncollapsed SAM files; performs
    hierarchical classification and quantification of multiple small RNA classes in
    parallel while timing each run
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: HTSeq-count
  role: Baseline counter run on the same two SAM files with equivalent counting parameters
    to compare runtimes
- name: featureCounts
  role: Fast baseline counter run on the same two SAM files; serves as the runtime
    reference point (2.4 s on collapsed SAM in the paper)
- name: Python
  role: Used to assemble recorded wall-clock times into the runtime_benchmark.csv
    table and verify the runtime ordering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:7c79e00f7427c1ddb60d01c57121e4fb9b1f20ce366566e10cd2c84b18b39c93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - HTSeq-count
  - Python
  - featureCounts
  - tiny-count
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# collapsed-uncollapsed-read-input-handling

## Summary

Covers how small RNA-seq counting tools (tiny-count, HTSeq-count, featureCounts) consume collapsed versus uncollapsed SAM alignment files, and how the collapsed-vs-uncollapsed input choice affects counting runtime. Grounded in the tiny-count paper's runtime benchmark, where collapsed-SAM tiny-count took 45.1 s versus featureCounts' 2.4 s.

## When to use

Use when counting small RNA-seq reads from SAM files that exist in both collapsed (read + copy-count) and uncollapsed (per-read) forms, or when benchmarking/counting runtime across counting tools on these two input representations.

## When NOT to use

- Input is already a feature count table — no SAM-level counting or runtime benchmarking is needed
- The task is single-nucleotide variant resolution (miRNA/isomiR discrimination) rather than input-representation handling or runtime comparison
- Only one input representation (collapsed OR uncollapsed) is available, so the collapsed-vs-uncollapsed comparison cannot be performed

## Inputs

- Collapsed SAM alignment file (small RNA reads collapsed with copy counts)
- Uncollapsed SAM alignment file (one alignment record per read)
- Reference feature annotation (GFF3, GFF2, or GTF; each feature needs an ID/gene_id or Parent tag)
- Reference genome FASTA (chromosome identifiers matching the annotation, case-sensitive) when counting against a genome

## Outputs

- runtime_benchmark.csv with columns for tool, input type (collapsed vs uncollapsed), and wall-clock runtime in seconds
- Per-tool count tables for the small RNA features quantified from each SAM input
- Runtime ordering across tiny-count, HTSeq-count, and featureCounts

## How to apply

Load the reference feature annotation (GFF3/GFF2/GTF with ID/gene_id/Parent attributes) together with both the collapsed and uncollapsed SAM alignment files. Run tiny-count on the uncollapsed SAM first, then on the collapsed SAM, recording wall-clock time per run; tiny-count quantifies small RNA classes (miRNAs/isomiRs, piRNAs, siRNAs, tRNA/rRNA fragments) in parallel via user-defined hierarchical selection rules. Repeat with HTSeq-count using equivalent counting parameters, and with featureCounts, recording wall-clock time for each of the two input types. Assemble tool, input type (collapsed vs uncollapsed), and runtime in seconds into runtime_benchmark.csv and compute the runtime ordering across the three tools. Rationale: collapsing deduplicates identical reads so counter throughput changes; the paper reports tiny-count's collapsed-SAM runtime (45.1 s) approaches featureCounts (2.4 s) while quantifying multiple small RNA classes simultaneously, but absolute times are hardware-dependent.

## Related tools

- **tiny-count** (Small RNA counting tool run on both collapsed and uncollapsed SAM files; performs hierarchical classification and quantification of multiple small RNA classes in parallel while timing each run) — https://github.com/MontgomeryLab/tinyRNA
- **HTSeq-count** (Baseline counter run on the same two SAM files with equivalent counting parameters to compare runtimes)
- **featureCounts** (Fast baseline counter run on the same two SAM files; serves as the runtime reference point (2.4 s on collapsed SAM in the paper))
- **Python** (Used to assemble recorded wall-clock times into the runtime_benchmark.csv table and verify the runtime ordering)

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- runtime_benchmark.csv exists with the required columns: tool, input type (collapsed vs uncollapsed), runtime in seconds
- All six runs (3 tools × 2 input types) completed and produced wall-clock times
- Recorded collapsed-SAM values fall in the reported neighborhood (tiny-count ~45.1 s, featureCounts ~2.4 s), with ordering verified on the local benchmark hardware
- Counting results (feature counts) are generated for both collapsed and uncollapsed inputs by each tool, not just timing output
- Equivalent counting parameters are documented for HTSeq-count so the cross-tool comparison is fair

## Limitations

- Absolute runtimes are hardware-dependent; the reported 45.1 s (tiny-count) vs 2.4 s (featureCounts) ordering must be verified on the local benchmark machine
- tiny-count is slower than featureCounts on collapsed SAM files even though it quantifies multiple small RNA classes in parallel
- Installation issues can arise on Apple Silicon-based systems when installing stable tinyRNA releases
- The README notes GFF3/GTF annotation constraints: Parent tags with multiple values are not yet supported, and attribute values containing commas must represent lists

## Evidence

- [abstract] tiny-count can be run alone or as part of tinyRNA, a workflow that provides a basic all-in-one command line-based solution: "tiny-count can be run alone or as part of tinyRNA, a workflow that provides a basic all-in-one command line-based solution"
- [results] On SAM files containing collapsed reads, tiny-count's performance approaches that of featureCounts while quantifying multiple small RNA classes in parallel, at 45.1 s versus 2.4 s.: [evidence span withheld — anchor status: not_found]
- [readme] Alternatively, you can install tinyRNA's precision counting tool by itself. Unlike the full tinyRNA suite, this option can be installed in existing conda environments and requires fewer dependencies.: [evidence span withheld — anchor status: external_doc]
