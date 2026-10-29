---
name: batch-query-throughput-benchmarking
description: Use when you need to quantify query scalability of an indexed sequence-collection
  structure (e.g. PAC) by timing batches of many sequences — from tens to hundreds
  of thousands — against a fixed index and compiling runtime and memory measurements.
when_to_use_negative:
- The index does not yet exist — this skill benchmarks querying only; index construction
  must be done first with './PAC -f fof.txt -d index_test'
- The goal is exact alignment or similarity scoring (e.g. BLAST-like local alignment)
  — PAC is an approximate membership query structure that only lists datasets that
  may contain a query sequence
- The collection exceeds ~65,536 files without adjusting encoding — by default PAC
  uses 16-bit integers and indexes up to 65,536 files; use -e 32 for larger collections
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3307
tools:
- name: PAC
  role: Performs the multithreaded batch queries against the pre-built index ('./PAC
    -l <index_folder> -q <batch_fasta> -c 16') whose runtimes are benchmarked
  repo: https://github.com/Malfoy/PAC
- name: seqkit
  role: Samples the query batches (100, 10,000, 100,000, 500,000 transcripts) from
    human RefSeq and writes each batch as a FASTA file
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:c89ed11016629d0fadb369099d0dd5e2f4341795fbe6c13c93a8277ef867baed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - seqkit
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# batch-query-throughput-benchmarking

## Summary

Benchmarks the query throughput of PAC (Partitioned Aggregated bloom Comb trees) by running batches of increasing numbers of human RefSeq transcript queries against a pre-built PAC index, recording wall-clock time, CPU time, and peak RAM per batch size.

## When to use

Use when you need to quantify query scalability of an indexed sequence-collection structure (e.g. PAC) by timing batches of many sequences — from tens to hundreds of thousands — against a fixed index and compiling runtime and memory measurements.

## When NOT to use

- The index does not yet exist — this skill benchmarks querying only; index construction must be done first with './PAC -f fof.txt -d index_test'
- The goal is exact alignment or similarity scoring (e.g. BLAST-like local alignment) — PAC is an approximate membership query structure that only lists datasets that may contain a query sequence
- The collection exceeds ~65,536 files without adjusting encoding — by default PAC uses 16-bit integers and indexes up to 65,536 files; use -e 32 for larger collections

## Inputs

- Previously built PAC index folder (e.g. index_test) covering the full collection of indexed datasets
- Human RefSeq transcript sequences (sampled with seqkit into FASTA batches of 100, 10,000, 100,000, and 500,000 sequences)

## Outputs

- pac_query_benchmark_timings.tsv — one row per batch size with wall-clock time, CPU time, and peak RAM
- Per-sequence query output lines listing kmer counts shared with each indexed dataset

## How to apply

First load a previously constructed PAC index folder (e.g. index_test) built over the full collection of indexed datasets. Using seqkit, sample query batches of 100, 10,000, 100,000, and 500,000 transcript sequences from human RefSeq and write each batch as a FASTA file. Run './PAC -l <index_folder> -q <batch_fasta> -c 16' for each batch, exploiting PAC's multithreaded query execution with 16 threads. Parse the output lines, where each line reports the number of kmers shared between the query sequence and each indexed dataset (e.g. '29719 50 701'). Record CPU time, wall-clock time, and peak RAM per batch size and compile them into a single TSV with one row per batch size. The rationale is that PAC was reported to query 500,000 transcript sequences in less than an hour (~40 min wall-clock, ~6.5 CPU hours), so timing escalating batch sizes against that envelope validates the claimed query throughput.

## Related tools

- **PAC** (Performs the multithreaded batch queries against the pre-built index ('./PAC -l <index_folder> -q <batch_fasta> -c 16') whose runtimes are benchmarked) — https://github.com/Malfoy/PAC
- **seqkit** (Samples the query batches (100, 10,000, 100,000, 500,000 transcripts) from human RefSeq and writes each batch as a FASTA file)

## Examples

```
./PAC -l index_test -q batch_500k_refseq_transcripts.fasta -c 16
```

## Evaluation signals

- The 100-transcript batch completes in approximately 5 minutes wall-clock
- The 500,000-transcript batch completes in approximately 40 minutes wall-clock (~6.5 CPU hours), i.e. less than an hour as reported in the paper
- The 10,000-transcript batch peaks at up to ~2 GB RAM
- Query output lines have the expected 'kmer_count dataset_id' triple format (e.g. '29719 50 701') with one line per query sequence per shared dataset
- pac_query_benchmark_timings.tsv contains exactly one row per batch size (100, 10,000, 100,000, 500,000)

## Limitations

- PAC is an approximate membership query structure: it lists datasets that may contain the query (with false positives controlled by Bloom filter sizing via -b), not exact matches
- Memory usage can increase linearly with the number of threads used during construction (and queries are faster but create more files with more partitions via -p)
- PAC can index at most 65,536 files with the default 16-bit integer encoding; -e 32 is required for larger collections
- ulimit on open files may constrain partitioned indexes ('ulimit -n 5000' allows 5000 opened files)

## Evidence

- [abstract] PAC queried 500 000 transcript sequences in less than an hour: "We also showed that PAC's ability to query 500 000 transcript sequences in less than an hour."
- [readme] Multithread query usage: "PAC will use 16 threads to perform the queries."
- [readme] Query output format: "Meaning that the query share 29719 kmers with the first entry, 50 with the second and 701 with the third."
- [methods] Load index and query a fasta file: "PAC will load the index inside the index_test folder and query each line of the GCF_000864885.1_ViralProj15500_genomic.fna.gz fasta file."
- [intro] PAC purpose as scalable listing structure: "PAC is a scalable structure able to list datasets that may contain a query sequence among thousands of indexed files."
