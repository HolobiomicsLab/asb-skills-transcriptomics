---
name: resource-usage-profiling-cpu-wallclock-ram
description: Use when measuring runtime and memory footprint of PAC queries against
  an indexed collection, e.g. validating that large transcript batches complete in
  under an hour.
when_to_use_negative:
- The goal is indexing construction throughput or disk footprint rather than query-side
  runtime and memory profiling
- The collection has not yet been indexed with PAC — queries require an existing index
  folder loaded with -l
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PAC
  role: Query engine run in multithreaded mode (-c 16) over the index; its output
    lines report shared kmer counts and its execution is timed for CPU/wall-clock/RAM
  repo: https://github.com/Malfoy/PAC
- name: seqkit
  role: Samples query batches of 100, 10,000, 100,000, and 500,000 transcript sequences
    from human RefSeq, writing each batch as a FASTA file
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:b6f0e4e38657c9c8749b81ebf8cd9c30dc62ff93c1dc17d9469582eea92d9be2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - seqkit
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Resource-usage profiling of PAC queries (CPU time, wall-clock, peak RAM)

## Summary

Benchmarks PAC query runs over batches of human RefSeq transcript sequences of increasing size (100 to 500,000 transcripts), recording CPU time, wall-clock time, and peak RAM per batch to characterize the tool's query scalability envelope.

## When to use

Use when measuring runtime and memory footprint of PAC queries against an indexed collection, e.g. validating that large transcript batches complete in under an hour.

## When NOT to use

- The goal is indexing construction throughput or disk footprint rather than query-side runtime and memory profiling
- The collection has not yet been indexed with PAC — queries require an existing index folder loaded with -l

## Inputs

- Previously built PAC index folder (e.g. index_test) constructed over the indexed dataset collection
- Human RefSeq transcript sequences sampled into query batches of 100, 10,000, 100,000, and 500,000 sequences as FASTA files (produced with seqkit)

## Outputs

- Per-batch measurements of CPU time, wall-clock time, and peak RAM, compiled into pac_query_benchmark_timings.tsv with one row per batch size
- Parsed PAC query output lines reporting the number of kmers shared between each query sequence and each indexed dataset

## How to apply

Load a previously constructed PAC index folder (e.g. index_test built over the full collection) and sample query batches of 100, 10,000, 100,000, and 500,000 transcript sequences from human RefSeq using seqkit, writing each batch as FASTA. Run each batch with './PAC -l <index_folder> -q <batch_fasta> -c 16', exploiting PAC's multithreaded query mode with 16 threads. Parse the query output lines (e.g. '29719 50 701', giving shared kmer counts per indexed dataset) and record CPU time, wall-clock time, and peak RAM for each batch size. Compile measurements into a table such as pac_query_benchmark_timings.tsv with one row per batch size, then check the recorded values against the reported envelope (~5 min wall-clock for 100 transcripts, ~40 min and ~6.5 CPU hours for 500,000 transcripts, up to 2 GB RAM for 10,000 transcripts).

## Related tools

- **PAC** (Query engine run in multithreaded mode (-c 16) over the index; its output lines report shared kmer counts and its execution is timed for CPU/wall-clock/RAM) — https://github.com/Malfoy/PAC
- **seqkit** (Samples query batches of 100, 10,000, 100,000, and 500,000 transcript sequences from human RefSeq, writing each batch as a FASTA file)

## Examples

```
./PAC -l index_test -q refseq_transcripts_500k.fasta -c 16
```

## Evaluation signals

- 100-transcript batch completes in approximately 5 min wall-clock
- 500,000-transcript batch completes in less than an hour, at ~40 min wall-clock and ~6.5 CPU hours
- 10,000-transcript batch peaks at up to 2 GB RAM
- Query output lines follow the expected format of shared kmer counts per indexed dataset (e.g. '29719 50 701')
- pac_query_benchmark_timings.tsv contains one row per batch size (100, 10,000, 100,000, 500,000)

## Limitations

- Multithreaded PAC runs can increase memory usage — thread count (-c) trades construction/query speed against RAM, so timings must be interpreted relative to the thread setting
- Memory usage can increase linearly with the amount of threads used during PAC operations
- Wall-clock and CPU times scale non-linearly with batch size; extrapolating beyond the tested batches (up to 500,000 transcripts) is not supported by the reported measurements

## Evidence

- [abstract] PAC queried 500 000 transcript sequences in less than an hour: "We also showed that PAC's ability to query 500 000 transcript sequences in less than an hour."
- [readme] Multithreaded query invocation: "PAC will use 16 threads to perform the queries."
- [readme] Threads trade speed for memory: "the memory usage can increase linearly with the amount of threads used"
- [readme] Query output format: "Meaning that the query share 29719 kmers with the first entry, 50 with the second and 701 with the third."
