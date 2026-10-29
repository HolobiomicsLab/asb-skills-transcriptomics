---
name: sequence-dataset-query-scoring
description: Use when you need to determine which of thousands of indexed FASTA/FASTQ
  datasets contain a query sequence, and how many k-mers the query shares with each
  dataset.
when_to_use_negative:
- You need exact alignment-level search results or full sequence alignment scores
  — PAC is an approximate membership query (AMQ) structure that lists candidate datasets,
  not a BLAST replacement for alignment
- The target collection has fewer datasets than could be searched directly with BLAST-like
  methods against a small number of genomes, where index construction overhead is
  not justified
- The query calls for positions or locations of matches within datasets — PAC only
  reports which datasets contain the sequence and shared k-mer counts
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: PAC
  role: Constructs the partitioned aggregated Bloom comb index and executes single
    or multithreaded queries returning per-dataset hit bit vectors and shared k-mer
    score vectors
  repo: https://github.com/Malfoy/PAC
- name: g++ 8 or higher
  role: C++17-ready compiler required to build PAC from source via make before querying
- name: zlib
  role: Required dependency (zlib1g-dev on Debian/Ubuntu) enabling PAC to read gzip-compressed
    FASTA/FASTQ query and index files
- name: Cobs
  role: Comparable Bloom filter-based AMQ structure; PAC's design point of reference
    for querying collections of sequence datasets
- name: HowDeSBT
  role: Sequence Bloom tree tool cited as an alternative AMQ approach to dataset listing
    that PAC improves upon in construction time
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:bd32adc3a098d8bd2cce8c3fa3589e8a067ad6ef193effff82673a5c450308e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - Cobs
  - HowDeSBT
  - PAC
  - g++ 8 or higher
  - zlib
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# sequence-dataset-query-scoring

## Summary

Run queries against a PAC (Partitioned Aggregated Bloom Comb tree) index so that each query sequence returns an n-sized bit vector of per-dataset hits plus a score vector counting shared k-mers with each indexed dataset, enabling approximate membership screening of a sequence against thousands of files.

## When to use

Use when you need to determine which of thousands of indexed FASTA/FASTQ datasets contain a query sequence, and how many k-mers the query shares with each dataset.

## When NOT to use

- You need exact alignment-level search results or full sequence alignment scores — PAC is an approximate membership query (AMQ) structure that lists candidate datasets, not a BLAST replacement for alignment
- The target collection has fewer datasets than could be searched directly with BLAST-like methods against a small number of genomes, where index construction overhead is not justified
- The query calls for positions or locations of matches within datasets — PAC only reports which datasets contain the sequence and shared k-mer counts

## Inputs

- Previously built PAC index folder (e.g. index_test) containing 4^p partition files with per-partition aggregated Bloom comb filters
- Query file of FASTA, MultiLine FASTA or FASTQ sequences, gzip-compressed or not, one query per line (e.g. test/GCF_000864885.1_ViralProj15500_genomic.fna.gz)
- Optional: a new file-of-files (fof) for index update via './PAC -f fof_add.txt -l index_test -d index_updated'

## Outputs

- Per query line: an n-sized bit vector of per-dataset hits indicating which indexed datasets likely contain the query
- Per query line: a score vector of shared k-mer counts per dataset, e.g. '29719 50 701' meaning the query shares 29719 k-mers with the first entry, 50 with the second, 701 with the third

## How to apply

Build (or reuse) a PAC index from a file-of-files listing FASTA/FASTQ datasets, choosing k (default 21) and the partition parameter -p so the k-mer space is split into 4^p partition files. Each partition holds an aggregated Bloom comb filter, and lazily constructed inverted Bloom filter indexes map each k-mer hash h to the bit slice of Bloom filters containing it, so querying successive k-mers of a super-k-mer stays within one small structure. Query with './PAC -l index_test -q queries.fasta', optionally with '-c 16' threads; PAC loads the index and processes each line of the query file. The query maintains a score vector of size n incremented at position j each time a query k-mer is found in leaf/dataset j, returning both the hit bit vector and the shared k-mer counts. Optional flags tune sensitivity and scale: '-u' to index only k-mers seen at least twice (filtering singletons), '-b' to size Bloom filters to the first power of two above the requested bit count (larger filters lower false positive rate), and '-e 8' to index up to 256 files with 8-bit encoding (default 16-bit supports up to 65,636 files).

## Related tools

- **PAC** (Constructs the partitioned aggregated Bloom comb index and executes single or multithreaded queries returning per-dataset hit bit vectors and shared k-mer score vectors) — https://github.com/Malfoy/PAC
- **g++ 8 or higher** (C++17-ready compiler required to build PAC from source via make before querying)
- **zlib** (Required dependency (zlib1g-dev on Debian/Ubuntu) enabling PAC to read gzip-compressed FASTA/FASTQ query and index files)
- **Cobs** (Comparable Bloom filter-based AMQ structure; PAC's design point of reference for querying collections of sequence datasets)
- **HowDeSBT** (Sequence Bloom tree tool cited as an alternative AMQ approach to dataset listing that PAC improves upon in construction time)

## Examples

```
./PAC -l index_test -q test/GCF_000864885.1_ViralProj15500_genomic.fna.gz -c 16
```

## Evaluation signals

- Index folder contains exactly 4^p partition files matching the -p parameter (e.g. -p 2 yields 16 files)
- Query output lines report integer score vectors whose length equals the number of indexed datasets, e.g. '29719 50 701' for the SARS coronavirus Tor2 example
- Shared k-mer counts against a self-indexed dataset are high (near the query's total k-mer count) while unrelated datasets score near zero apart from controlled Bloom filter false positives
- False positive rate stays within bounds implied by the chosen Bloom filter size (-b flag sizes filters to the first power of two above the requested bit count)
- Multithreaded query with -c N produces results consistent with the single-threaded run

## Limitations

- PAC is approximate: Bloom filter false positives inflate per-dataset scores, controlled but not eliminated by filter sizing
- Memory usage during construction and query can increase linearly with the number of threads used (-c)
- More partitions (-p) speed up construction and query and reduce memory, but create more files — the ulimit on open file descriptors must be raised (e.g. 'ulimit -n 5000')
- Default 16-bit integer encoding caps indexing at 65,636 files; more requires -e 32, and -e 8 caps at 256 files
- With -u, k-mers seen only once in a dataset are not indexed, so single-copy query k-mers may be missed

## Evidence

- [methods] Per partition, a lazily constructed Inverted Bloom filter index maps each k-mer hash value h to the bit slice listing which Bloom filters contain it: [evidence span withheld — anchor status: not_found]
- [methods] Query returns score vector incremented per dataset hit: [evidence span withheld — anchor status: not_found]
- [readme] Query output example from README showing shared k-mer counts: "Meaning that the query share 29719 kmers with the first entry, 50 with the second and 701 with the third."
- [abstract] PAC is an AMQ structure for listing candidate datasets: "PAC, a novel approximate membership query data structure for querying collections of sequence datasets"
- [readme] Singleton k-mer filter option from README: "PAC will only index kmers seen at least two times in a given dataset (kmers seen only once are filtered)"
- [readme] Partitioning behavior and open file descriptor caveat: "PAC will split the index into 4^p files inside the index_test folder."
- [readme] Bloom filter sizing rationale: "Larger  Bloom filters decrease false positive rate but increase the index size."
- [abstract] Query throughput performance claim: "We also showed that PAC's ability to query 500 000 transcript sequences in less than an hour."
