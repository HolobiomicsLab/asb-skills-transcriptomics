---
name: sequence-database-query-benchmarking
description: Use when measuring how query time/throughput on a large sequence-database
  index changes with index configuration (e.g., zero/one/two Aggregated Bloom Comb
  trees) across query batches of similar, dissimilar, and random sequences.
when_to_use_negative:
- The analysis goal is exact sequence alignment or full similarity search rather than
  approximate membership queries listing datasets that may contain a sequence (BLAST-like
  search is out of PAC's scope).
- The index structures under comparison are not AMQ/Bloom-filter based (e.g., benchmarking
  HowDeSBT/Cobs requires their own CLIs, not PAC's -l/-q interface).
- Only one query batch is available, so no similar-vs-dissimilar contrast can be made
  and Bloom filter acceleration cannot be meaningfully evaluated.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: PAC
  role: 'Primary tool: builds the partitioned aggregated Bloom comb tree index (-f/-d),
    queries it per batch (-l/-q), and reports shared kmer counts and query throughput
    per configuration.'
  repo: https://github.com/Malfoy/PAC
- name: g++ 8 or higher
  role: C++17-ready compiler required to build PAC from source with make.
- name: zlib
  role: Dependency enabling PAC to read gzipped FASTA/FASTQ input files (zlib1g-dev
    on Debian/Ubuntu).
- name: BitMagic
  role: Compressed bit-vector library whose succinct containers/bloom-filter components
    underpin memory-compact structures like PAC's aggregated Bloom filters.
  repo: https://github.com/tlk00/BitMagic
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:35e1ff25034fc0008e9648f69c9d1b50b5533e8deec6a24c7e93d2d67fe32b6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - BitMagic
  - PAC
  - g++ 8 or higher
  - zlib
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# sequence-database-query-benchmarking

## Summary

Benchmarks query throughput of an AMQ index (PAC, built with Aggregated Bloom Comb trees) over collections of genome/transcriptome datasets, comparing index configurations across query batches of varying similarity. Grounded in the PAC paper's ablation showing several-fold speedups on dissimilar batches.

## When to use

Use when measuring how query time/throughput on a large sequence-database index changes with index configuration (e.g., zero/one/two Aggregated Bloom Comb trees) across query batches of similar, dissimilar, and random sequences.

## When NOT to use

- The analysis goal is exact sequence alignment or full similarity search rather than approximate membership queries listing datasets that may contain a sequence (BLAST-like search is out of PAC's scope).
- The index structures under comparison are not AMQ/Bloom-filter based (e.g., benchmarking HowDeSBT/Cobs requires their own CLIs, not PAC's -l/-q interface).
- Only one query batch is available, so no similar-vs-dissimilar contrast can be made and Bloom filter acceleration cannot be meaningfully evaluated.

## Inputs

- File-of-files manifest (fof.txt) listing one genome FASTA/FASTQ path per line (gzip allowed)
- Query batch files (e.g., S. enterica genomes, E. coli genomes, random sequences) in FASTA/FASTQ
- Compiled PAC binary and built index folder (e.g., index_test with 4^p partition files)

## Outputs

- Per-query output lines reporting shared kmer counts per indexed dataset (e.g., '>NC_004718.3 ... / 29719 50 701')
- Query throughput/elapsed-time metrics per batch and per index configuration
- Comparison table such as query_times_table2.csv or pac_index_resource_metrics.tsv (construction time, peak RAM, on-disk index size)

## How to apply

Assemble datasets into a file-of-files manifest (one 'readlink -f' path per line), clone https://github.com/Malfoy/PAC and compile with make (g++ >= 8, C++17, zlib). Build the index with './PAC -f fof.txt -d index_test', optionally with '-b' to set Bloom filter sizes (sizes round up to the next power of two, e.g., 2^29 or 2^27 bits), '-p' for 4^p partitions (check ulimit -n allows enough open files), and '-c' threads for construction. Query each batch (FASTA/multi-line FASTA/FASTQ, gzipped or not) with './PAC -l index_test -q <batch> -c 16', recording elapsed time and throughput. Repeat the identical query for each index configuration (two Aggregated Bloom Comb trees, single tree, plain Bloom matrix), changing only the index configuration. Compile per-batch query times and shared-kmer counts into a table (e.g., query_times_table2.csv) and compare configurations.

## Related tools

- **PAC** (Primary tool: builds the partitioned aggregated Bloom comb tree index (-f/-d), queries it per batch (-l/-q), and reports shared kmer counts and query throughput per configuration.) — https://github.com/Malfoy/PAC
- **g++ 8 or higher** (C++17-ready compiler required to build PAC from source with make.)
- **zlib** (Dependency enabling PAC to read gzipped FASTA/FASTQ input files (zlib1g-dev on Debian/Ubuntu).)
- **BitMagic** (Compressed bit-vector library whose succinct containers/bloom-filter components underpin memory-compact structures like PAC's aggregated Bloom filters.) — https://github.com/tlk00/BitMagic

## Examples

```
./PAC -l index_test -q test/GCF_000864885.1_ViralProj15500_genomic.fna.gz -c 16
```

## Evaluation signals

- Each index folder contains the expected 4^p partition files for the chosen -p value.
- On the dissimilar batch (E. coli), configurations with Aggregated Bloom filters show a several-fold query-throughput gain over the plain Bloom matrix, matching the paper's Table 2 comparison.
- On the highly similar batch (S. enterica), one Aggregated Bloom filter yields hardly any improvement — a sanity contrast against the dissimilar batch.
- A second Aggregated Bloom filter speeds queries further but with diminishing benefit, consistent with reported diminishing returns.
- Per query line, output reports per-dataset shared kmer counts (e.g., '>NC_004718.3 ... / 29719 50 701'), confirming per-dataset membership hits are returned.

## Limitations

- Bloom filter size (-b) trades false positive rate against index size: larger filters decrease false positives but increase index size (sizes round up to the next power of two).
- Memory usage increases roughly linearly with the number of construction threads (-c).
- More partitions (-p) speed up construction and queries and reduce memory, but create more files and require sufficient open file descriptors (ulimit -n, e.g., 'ulimit -n 5000').
- Default 16-bit aggregated filter integers allow indexing only ~65,636 files; use '-e 8' for up to 256 files (smaller) or '-e 32' to index more files.
- PAC is an approximate structure: it lists datasets that may contain a query sequence, with a false positive rate determined by filter size.

## Evidence

- [readme] PAC is a scalable structure able to list datasets that may contain a query sequence among thousands of indexed files.: "PAC is a scalable structure able to list datasets that may contain a query sequence among thousands of indexed files."
- [readme] Multithreaded query mode with recorded throughput: "`./PAC -l index_test -q test/GCF_000864885.1_ViralProj15500_genomic.fna.gz -c 16` PAC will use 16 threads to perform the queries."
- [readme] Bloom filter sizing trade-off: "Larger  Bloom filters decrease false positive rate but increase the index size."
- [readme] Shared kmer output format: "Meaning that the query share 29719 kmers with the first entry, 50 with the second and 701 with the third."
- [abstract] Construction scalability claims: "the entire GenBank bacterial genome collection in a single day for an index size of 3.5 TB."
