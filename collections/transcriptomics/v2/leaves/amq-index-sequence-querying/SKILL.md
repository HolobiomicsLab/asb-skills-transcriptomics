---
name: amq-index-sequence-querying
description: Use when you have batches of query sequences (e.g., human RefSeq transcripts
  in FASTA) and a prebuilt AMQ index over thousands of datasets, and you need to list
  candidate datasets per query while profiling query runtime, CPU usage, and memory
  footprint.
when_to_use_negative:
- When exact alignment or exhaustive similarity search is required — PAC is an approximate
  membership query structure that only lists datasets that may contain the query,
  not BLAST-like alignments
- When no PAC index exists yet — index construction ('./PAC -f fof.txt -d index_test')
  is a separate workflow step and this skill assumes the index is already built
- When the query sequences' informative kmers are seen only once in the indexed datasets
  — PAC's -u construction option filters singletons, so such queries cannot be recalled
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: PAC
  role: 'Query engine: loads the prebuilt index folder and performs multithreaded
    (-c 16) approximate membership queries of batch FASTA files against thousands
    of indexed datasets'
  repo: https://github.com/Malfoy/PAC
- name: seqkit
  role: Samples query batches of 100, 10,000, 100,000, and 500,000 transcript sequences
    from human RefSeq and writes each batch as a FASTA file
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:99f90f050b734bf709c9963074e00035711f7ae4479c7fdaf676f2d2fbaa5abc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - seqkit
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# amq-index-sequence-querying

## Summary

Benchmark-style querying of large batches of nucleotide sequences against a prebuilt PAC (Partitioned Aggregated bloom Comb trees) approximate membership query (AMQ) index to identify which indexed datasets share kmers with each query, while recording wall-clock time, CPU time, and peak RAM per batch size.

## When to use

Use when you have batches of query sequences (e.g., human RefSeq transcripts in FASTA) and a prebuilt AMQ index over thousands of datasets, and you need to list candidate datasets per query while profiling query runtime, CPU usage, and memory footprint.

## When NOT to use

- When exact alignment or exhaustive similarity search is required — PAC is an approximate membership query structure that only lists datasets that may contain the query, not BLAST-like alignments
- When no PAC index exists yet — index construction ('./PAC -f fof.txt -d index_test') is a separate workflow step and this skill assumes the index is already built
- When the query sequences' informative kmers are seen only once in the indexed datasets — PAC's -u construction option filters singletons, so such queries cannot be recalled

## Inputs

- Query batch FASTA file (PAC also accepts multiline FASTA and FASTQ, gzip-compressed or not), e.g., batches of 100–500,000 human RefSeq transcript sequences sampled with seqkit
- Previously constructed PAC index folder (e.g., index_test)
- File-of-files (fof.txt) used to build the index (context for which datasets are indexed)

## Outputs

- Query output lines mapping each query sequence to per-dataset shared kmer counts (e.g., '29719 50 701' per query header)
- Benchmark timings table (e.g., pac_query_benchmark_timings.tsv) with one row per batch size: CPU time, wall-clock time, peak RAM

## How to apply

Load an existing PAC index folder (e.g., index_test) built over the full collection, then run PAC in query mode ('./PAC -l <index_folder> -q <batch_fasta> -c 16') using 16 threads, since PAC supports multithreaded query execution. Prepare query batches (here 100, 10,000, 100,000, and 500,000 human RefSeq transcript sequences sampled with seqkit) as FASTA files; PAC handles FASTA, multiline FASTA, and FASTQ, gzipped or not. Parse the output lines, which report for each query sequence the number of kmers shared with each indexed dataset (e.g., '29719 50 701' meaning 29719, 50, and 701 shared kmers with the first, second, and third entries). Record CPU time, wall-clock time, and peak RAM per batch size and compile them into a table such as pac_query_benchmark_timings.tsv. Note that during index construction, the -u option filters kmers seen only once, so only kmers seen at least twice are indexed — this affects which query kmers can ever match.

## Related tools

- **PAC** (Query engine: loads the prebuilt index folder and performs multithreaded (-c 16) approximate membership queries of batch FASTA files against thousands of indexed datasets) — https://github.com/Malfoy/PAC
- **seqkit** (Samples query batches of 100, 10,000, 100,000, and 500,000 transcript sequences from human RefSeq and writes each batch as a FASTA file)

## Examples

```
./PAC -l index_test -q refseq_transcripts_500k.fasta -c 16
```

## Evaluation signals

- Recorded wall-clock time for the 100-transcript batch is approximately 5 minutes; the 500,000-transcript batch completes in less than an hour (~40 min wall-clock, ~6.5 CPU hours)
- Peak RAM stays within ~2 GB for the 10,000-transcript batch
- Output for every query header is a line of per-dataset shared kmer counts in the documented format (e.g., '29719 50 701')
- Timings across batch sizes (100 → 500,000) form a consistent scaling curve rather than anomalous outliers for any batch
- The index folder is loaded without reconstruction — query mode uses -l, not -f/-d

## Limitations

- PAC is approximate: it lists datasets that may contain a query sequence, with a false-positive rate controlled by Bloom filter sizing (-b); larger filters decrease false positives but increase index size
- Memory usage during PAC execution can increase linearly with the number of threads used (-c)
- More index partitions (-p) create more files — check 'ulimit -n 5000' allows enough open files — though partitions speed up and reduce memory during construction and query
- Default 16-bit integers allow indexing up to 65,536 files; use -e 32 to index more datasets
- With -u, kmers seen only once per dataset are not indexed, so single-occurrence query kmers cannot match

## Evidence

- [abstract] abstract: "We also showed that PAC's ability to query 500 000 transcript sequences in less than an hour."
- [methods] methods: "PAC will load the index inside the index_test folder and query each line of the GCF_000864885.1_ViralProj15500_genomic.fna.gz fasta file."
- [readme] readme-multithread: "PAC will use 16 threads to perform the queries."
- [readme] readme-formats: "PAC can handle FASTA, MultiLine FASTA  and FASTQ files gziped or not."
