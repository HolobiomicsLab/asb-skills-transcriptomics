---
name: query-throughput-analysis
description: Use when measuring how query time or throughput on a PAC index changes
  with index configuration (plain Bloom matrix vs. 1 or 2 aggregated Bloom filters)
  across batches that are similar, dissimilar, or random relative to the indexed genomes.
when_to_use_negative:
- The query batch is highly similar to the indexed genomes (e.g. same-species S. enterica
  batch), where the single Aggregated Bloom filter hardly improves query time and
  the benchmark adds little signal
- The goal is exact sequence search or alignment (BLAST-like) rather than approximate
  membership queries listing datasets that may contain a sequence
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: PAC
  role: Index construction (-f/-d, optional -c threads, -p partitions) and multithreaded
    querying (-l/-q/-c 16) under each Aggregated Bloom Comb tree configuration
  repo: https://github.com/Malfoy/PAC
- name: g++ 8 or higher
  role: C++17-ready compiler required to build PAC via make
- name: zlib
  role: Compression library required by PAC (zlib1g-dev on Debian/Ubuntu) for reading
    gzipped FASTA/FASTQ
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:68bf4aebb1928fb5ed91001d796d3657cd202305d58f130048b1d300622a50a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - g++ 8 or higher
  - zlib
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# query-throughput-analysis

## Summary

Benchmarks PAC query throughput (elapsed time and per-query shared kmer counts) on a partitioned aggregated Bloom comb tree index across sequence batches of varying similarity, comparing index configurations with zero, one, or two Aggregated Bloom Comb trees.

## When to use

Use when measuring how query time or throughput on a PAC index changes with index configuration (plain Bloom matrix vs. 1 or 2 aggregated Bloom filters) across batches that are similar, dissimilar, or random relative to the indexed genomes.

## When NOT to use

- The query batch is highly similar to the indexed genomes (e.g. same-species S. enterica batch), where the single Aggregated Bloom filter hardly improves query time and the benchmark adds little signal
- The goal is exact sequence search or alignment (BLAST-like) rather than approximate membership queries listing datasets that may contain a sequence

## Inputs

- File-of-files manifest (one genome FASTA/FASTQ path per line, gzipped or not) of complete Salmonella enterica RefSeq genomes
- Three query batches: similar (S. enterica), dissimilar (E. coli), and random sequences

## Outputs

- Elapsed query times / throughput per batch and per index configuration
- Per-query-line shared kmer counts against indexed datasets
- Compiled comparison table (e.g. query_times_table2.csv)

## How to apply

Build a PAC index with './PAC -f fof.txt -d index_test' (g++ >= 8, zlib), optionally raising 'ulimit -n' and using -p for partitions and -c for parallel construction. Prepare query batches with distinct expected kmer-sharing profiles (e.g. S. enterica genomes = similar, E. coli = dissimilar, random sequences = background) and query each with './PAC -l index_test -q <batch> -c 16', recording elapsed time and throughput. Repeat the identical query workload for each index configuration (two Aggregated Bloom Comb trees, single, none/plain Bloom matrix), holding all other parameters fixed, then compile query times and kmer-share counts per batch/configuration into a table (e.g. query_times_table2.csv). Rationale: aggregated Bloom filters let PAC skip partitions unlikely to contain the query, so gains concentrate on dissimilar batches.

## Related tools

- **PAC** (Index construction (-f/-d, optional -c threads, -p partitions) and multithreaded querying (-l/-q/-c 16) under each Aggregated Bloom Comb tree configuration) — https://github.com/Malfoy/PAC
- **g++ 8 or higher** (C++17-ready compiler required to build PAC via make)
- **zlib** (Compression library required by PAC (zlib1g-dev on Debian/Ubuntu) for reading gzipped FASTA/FASTQ)

## Examples

```
./PAC -l index_test -q ecoli_batch.fa -c 16
```

## Evaluation signals

- The aggregated configurations show a several-fold query-throughput gain over the plain Bloom matrix on the dissimilar (E. coli) batch, consistent with the paper's Table 2 comparison
- Each output query line reports per-dataset shared kmer counts (e.g. '29719 50 701' meaning kmer shares with the first, second, third entry)
- Query throughput is computed with identical -c 16 multithreaded query settings across all index configurations, differing only in the number of aggregated Bloom trees
- Index construction completes in streaming fashion with no disk footprint besides the index itself

## Limitations

- A single Aggregated Bloom filter greatly accelerates dissimilar batches but hardly improves query time on the highly similar batch; a second filter helps further but with diminishing benefit
- Memory usage during construction can increase linearly with the number of threads used
- Indexing splits into 4^p files, requiring sufficient open file descriptors ('ulimit -n 5000'); more partitions speed operations but create more files
- By default PAC uses 16-bit integers supporting up to 65,536 indexed files; use -e 8 for up to 256 files or -e 32 for more
- PAC by default only indexes kmers seen at least two times (-u filters singletons), so query kmer-share counts reflect that filter

## Evidence

- [results] Using one Aggregated Bloom filter greatly accelerates queries on the two dissimilar batches, increasing throughput several fold: [evidence span withheld — anchor status: not_found]
- [readme] PAC will construct an index in the folder index_test using all the datasets: "PAC will construct an index in the folder index_test using all the datasets containend in the file of file fof.txt."
- [readme] PAC will use 16 threads to perform the queries: "`./PAC -l index_test -q test/GCF_000864885.1_ViralProj15500_genomic.fna.gz -c 16`
  
PAC will use 16 threads to perform the queries."
- [readme] the query share 29719 kmers with the first entry, 50 with the second and 701 with the third: "Meaning that the query share 29719 kmers with the first entry, 50 with the second and 701 with the third."
- [abstract] A PAC query can need single random access and be performed in constant time in favorable instances: "A PAC query can need single random access and be performed in constant time in favorable instances."
