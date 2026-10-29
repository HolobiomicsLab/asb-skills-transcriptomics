---
name: transcript-reference-sampling
description: Use when benchmarking the query scalability of an indexed sequence-collection
  search structure (e.g. PAC) and you need realistic, size-graded query workloads
  drawn from a reference transcript database.
when_to_use_negative:
- The analysis goal is measuring index construction time or memory rather than query
  performance on sampled transcript batches
- The query workload must be a real user query set rather than a synthetically subsampled
  reference batch
- The index under test does not support batch querying of FASTA input (e.g. no multithreaded
  query mode comparable across batch sizes)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PAC
  role: 'Query engine under benchmark: loads the index folder and queries each sampled
    RefSeq transcript batch with a fixed number of threads (e.g. 16), reporting per-sequence
    kmer-sharing counts'
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
  source_content_hash: sha256:7196d97236074c2daa1b37240a79edc268a88f30859037d53bdf0bbe25842807
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - seqkit
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# transcript-reference-sampling

## Summary

A benchmarking recipe that samples query batches of increasing size from a reference transcript set (human RefSeq) using seqkit, and measures a sequence-search index's query CPU time, wall-clock time, and peak RAM across batch sizes (100 to 500,000 transcripts).

## When to use

Use when benchmarking the query scalability of an indexed sequence-collection search structure (e.g. PAC) and you need realistic, size-graded query workloads drawn from a reference transcript database.

## When NOT to use

- The analysis goal is measuring index construction time or memory rather than query performance on sampled transcript batches
- The query workload must be a real user query set rather than a synthetically subsampled reference batch
- The index under test does not support batch querying of FASTA input (e.g. no multithreaded query mode comparable across batch sizes)

## Inputs

- Human RefSeq transcript sequences (FASTA) to sample query batches from
- A previously constructed index folder (e.g. PAC index_test built over the full collection of indexed datasets)
- seqkit for subsampling fixed-size transcript batches
- Timing/memory instrumentation for CPU time, wall-clock time, and peak RAM

## Outputs

- Per-batch-size FASTA query files (100, 10,000, 100,000, 500,000 human RefSeq transcripts)
- Per-sequence query output lines reporting kmer sharing with indexed datasets
- A benchmark timing table, pac_query_benchmark_timings.tsv, with one row per batch size (CPU time, wall-clock time, peak RAM)

## How to apply

Load the previously constructed index folder (e.g. index_test) built over the full collection of indexed datasets. Sample query batches of 100, 10,000, 100,000, and 500,000 transcript sequences from human RefSeq using seqkit, writing each batch as a FASTA file. For each batch, run the query tool with a fixed thread count (PAC supports multithreaded query execution, e.g. '-c 16') so timings are comparable across sizes. Parse the per-sequence query output lines — for PAC these report the number of kmers shared with each indexed dataset, e.g. '29719 50 701' — and record CPU time, wall-clock time, and peak RAM per batch size. Compile the measurements into a single TSV with one row per batch size, then check the recorded timings against the expected performance envelope (~5 min wall-clock for 100 transcripts, ~40 min wall-clock / ~6.5 CPU hours for 500,000 transcripts, up to 2 GB RAM for 10,000 transcripts, with the largest batch completing in under an hour).

## Related tools

- **PAC** (Query engine under benchmark: loads the index folder and queries each sampled RefSeq transcript batch with a fixed number of threads (e.g. 16), reporting per-sequence kmer-sharing counts) — https://github.com/Malfoy/PAC
- **seqkit** (Samples query batches of 100, 10,000, 100,000, and 500,000 transcript sequences from human RefSeq, writing each batch as a FASTA file)

## Examples

```
./PAC -l index_test -q human_refseq_500000.fa -c 16
```

## Evaluation signals

- 100-transcript batch completes in approximately 5 min wall-clock
- 500,000-transcript batch completes in approximately 40 min wall-clock and ~6.5 CPU hours, i.e. less than an hour as reported in the paper
- 10,000-transcript batch peaks at up to 2 GB RAM
- Per-sequence output lines are parseable kmer-sharing counts of the form '29719 50 701' (kmers shared with each indexed dataset)
- pac_query_benchmark_timings.tsv contains exactly one row per tested batch size with CPU time, wall-clock time, and peak RAM recorded

## Limitations

- Multithreaded execution memory can grow with thread count; PAC's README notes memory usage during construction/query can increase linearly with the number of threads, so fixed thread counts must be held constant across batches for fair comparison
- PAC index files cap the number of indexable datasets (16-bit aggregated bloom filters index up to 65,536 files; use -e 32 for more), which bounds how large the indexed collection behind the benchmark can be
- Larger partitioning (-p) or bloom filter sizing (-b) changes query speed and false-positive rate, so benchmark results are conditional on the index construction parameters used

## Evidence

- [methods] PAC will load the index inside the index_test folder and query each line of the GCF_000864885.1_ViralProj15500_genomic.fna.gz fasta file.: "PAC will load the index inside the index_test folder and query each line of the GCF_000864885.1_ViralProj15500_genomic.fna.gz fasta file."
- [abstract] We also showed that PAC's ability to query 500 000 transcript sequences in less than an hour.: "We also showed that PAC's ability to query 500 000 transcript sequences in less than an hour."
- [readme] PAC will use 16 threads to perform the queries.: "PAC will use 16 threads to perform the queries."
- [readme] PAC is able to efficiently use a high amount of threads to divide the construction time but the memory usage can increase linearly with the amount of threads used.: "PAC is able to efficiently use a high amount of threads to divide the construction time but the memory usage can increase linearly with the amount of threads used."
