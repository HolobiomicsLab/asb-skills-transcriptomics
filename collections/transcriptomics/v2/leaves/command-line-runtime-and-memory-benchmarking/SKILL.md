---
name: command-line-runtime-and-memory-benchmarking
description: Use when evaluating whether a tool update (e.g., Seq2Fun 2.0 vs 1.0)
  improves read-processing performance, by reproducing a paper's benchmark on the
  same FASTQ datasets with the same parameters.
when_to_use_negative:
- Input is already a count table or gene list — statistical/functional analysis should
  use the ExpressAnalyst/ExpressAnalystR pipeline, not a read-processing benchmark
- The goal is differential expression or pathway enrichment rather than measuring
  software runtime and memory footprint
- The tool's parameter space or database has changed fundamentally between versions,
  so identical-parameter comparisons are no longer meaningful
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seq2Fun 1.0
  role: baseline command-line tool for mapping RNA-seq reads to KO databases via translated
    search; benchmarked for runtime and peak memory
- name: Seq2Fun 2.0
  role: updated version benchmarked against 1.0 to quantify the reduced memory footprint
    and sustained throughput
- name: ExpressAnalyst
  role: web platform wrapping Seq2Fun-based read quantification and annotation whose
    performance the benchmark characterizes
  repo: https://www.expressanalyst.ca
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:636a20697f854e101a057868b7b146af478ecb853761a90198c2563a3563857a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - Seq2Fun 1.0
  - Seq2Fun 2.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# command-line-runtime-and-memory-benchmarking

## Summary

Benchmark a command-line read-processing tool (e.g., Seq2Fun 1.0 vs 2.0) by running identical FASTQ inputs and parameters under both versions and logging wall-clock runtime and peak memory, to quantify the throughput/memory trade-off of a software update.

## When to use

Use when evaluating whether a tool update (e.g., Seq2Fun 2.0 vs 1.0) improves read-processing performance, by reproducing a paper's benchmark on the same FASTQ datasets with the same parameters.

## When NOT to use

- Input is already a count table or gene list — statistical/functional analysis should use the ExpressAnalyst/ExpressAnalystR pipeline, not a read-processing benchmark
- The goal is differential expression or pathway enrichment rather than measuring software runtime and memory footprint
- The tool's parameter space or database has changed fundamentally between versions, so identical-parameter comparisons are no longer meaningful

## Inputs

- Twelve zebrafish RNA-seq FASTQ files (SRR13332314–SRR13332325, NCBI SRA study SRP299836)
- Seq2Fun 1.0 and Seq2Fun 2.0 binaries
- Zebrafish KO (KEGG ortholog) database

## Outputs

- Per-sample wall-clock runtime and peak memory log for both tool versions
- Benchmark table of throughput (reads processed per minute) and peak memory (GB) per version
- Comparison against reported figures: 1.49 GB (v1.0) vs 0.94 GB (v2.0), ~2 million reads/minute

## How to apply

Download the twelve zebrafish FASTQ samples (SRR13332314–SRR13332325) from NCBI SRA study SRP299836. Run Seq2Fun 1.0 on each sample against the zebrafish KO database using the paper's settings — maximum number of mismatches = 2, minimum matching length = 19, minimum matching BLOSUM62 score = 80 — logging wall-clock runtime and peak memory per run. Repeat identically with Seq2Fun 2.0, then compute throughput in reads processed per minute and compile per-sample time and peak memory into a single benchmark table. Compare measured values against the reported figures (1.49 GB for v1.0 vs 0.94 GB for v2.0; ~2 million reads per minute) as the validation target. Keep thread count fixed (at or below 16) across versions, since beyond 16 threads performance plateaus due to an I/O constraint.

## Related tools

- **Seq2Fun 1.0** (baseline command-line tool for mapping RNA-seq reads to KO databases via translated search; benchmarked for runtime and peak memory)
- **Seq2Fun 2.0** (updated version benchmarked against 1.0 to quantify the reduced memory footprint and sustained throughput)
- **ExpressAnalyst** (web platform wrapping Seq2Fun-based read quantification and annotation whose performance the benchmark characterizes) — https://www.expressanalyst.ca

## Evaluation signals

- Benchmark table contains runtime and peak-memory values for all 12 samples (SRR13332314–SRR13332325) under both versions
- Version 2.0 peak memory falls near 0.94 GB versus 1.49 GB for version 1.0, matching the reported reduction
- Throughput is on the order of ~2 million reads per minute
- Identical Seq2Fun parameters were used across versions: max mismatches = 2, min matching length = 19, min matching BLOSUM62 score = 80
- Scaling check: performance does not improve beyond 16 threads, confirming the run was I/O-bound as expected

## Limitations

- Adding more threads (higher than 16) may not improve performance due to the I/O constraint; using a high-speed drive could relieve it
- Peak memory and throughput figures are specific to the twelve zebrafish test samples and the zebrafish KO database; they may not transfer to other organisms or database sizes
- The version-2.0 database is more than 125 times larger than version 1.0, so memory comparisons reflect the updated algorithm and search strategy, not an identical workload

## Evidence

- [discussion] Seq2Fun 2.0 memory reduction: "We have updated Seq2Fun to version 2.0 to reduce its memory footprint (e.g., from 1.49 GB to 0.94 GB)"
- [discussion] Seq2Fun benchmark parameters: "“maximum number of mismatches” = 2, “minimum matching length” = 19, and “minimum matching BLOSUM62 score” = 80."
- [discussion] I/O thread scaling limitation: "Adding more threads (higher than 16) may not improve performance due to this I/O constraint."
- [discussion] benchmark dataset accessions: "sample accessions SRR13332314 – SRR13332325"
