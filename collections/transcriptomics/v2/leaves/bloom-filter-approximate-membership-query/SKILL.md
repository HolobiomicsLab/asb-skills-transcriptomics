---
name: bloom-filter-approximate-membership-query
description: Use when benchmarking approximate membership query structures (e.g.,
  PAC, Cobs, HowDeSBT) on large collections of FASTA/FASTQ genomes, or when deciding
  how many Aggregated Bloom Comb trees to enable for sequence search.
when_to_use_negative:
- The task requires exact sequence alignment or full BLAST-like sensitivity rather
  than approximate membership listing of candidate datasets (PAC is an AMQ with Bloom-filter
  false positives).
- The index must be queried with a single sequence against a small collection where
  BLAST-like search is already routine.
- More than 65,536 files must be indexed without adjusting the -e encoding option,
  since the default 16-bit integers cap the index at ~65,636 files.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: PAC
  role: Builds the partitioned aggregated Bloom comb tree index and executes the query
    benchmark across zero/one/two aggregated tree configurations
  repo: https://github.com/Malfoy/PAC
- name: g++ 8 or higher
  role: C++17-ready compiler required to build PAC from source with make
- name: zlib
  role: Dependency (zlib1g-dev on Debian/Ubuntu) enabling PAC to read gzipped FASTA/FASTQ
    inputs
- name: Cobs
  role: Referenced Bloom-filter-based AMQ baseline in the same class of structures
    as PAC
- name: HowDeSBT
  role: Referenced Sequence Bloom tree tool contextualizing PAC among prior AMQ approaches
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:bf2538a31635b6343fb8caf9ab810ac422e0fbf599d2df7c51a984a75a99c40d
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
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# bloom-filter-approximate-membership-query

## Summary

Benchmarks PAC (Partitioned Aggregated Bloom Comb trees), a Bloom-filter-based approximate membership query (AMQ) structure that lists which datasets in a collection may contain a query sequence, comparing index configurations with zero, one, or two Aggregated Bloom Comb trees. Quantifies how aggregated Bloom filters accelerate queries several-fold on dissimilar batches while giving little benefit on highly similar batches.

## When to use

Use when benchmarking approximate membership query structures (e.g., PAC, Cobs, HowDeSBT) on large collections of FASTA/FASTQ genomes, or when deciding how many Aggregated Bloom Comb trees to enable for sequence search.

## When NOT to use

- The task requires exact sequence alignment or full BLAST-like sensitivity rather than approximate membership listing of candidate datasets (PAC is an AMQ with Bloom-filter false positives).
- The index must be queried with a single sequence against a small collection where BLAST-like search is already routine.
- More than 65,536 files must be indexed without adjusting the -e encoding option, since the default 16-bit integers cap the index at ~65,636 files.

## Inputs

- File-of-files manifest (fof.txt) listing one complete S. enterica RefSeq genome path per line
- Query batches in FASTA/FASTQ format (gzip allowed): similar S. enterica batch, dissimilar E. coli batch, and random sequence batch
- Compiled PAC binary (g++ >= 8, zlib1g-dev)

## Outputs

- PAC index folder (e.g. index_test) with Bloom filters split into 4^p partition files
- Per-query output lines reporting kmers shared between each query sequence and each indexed dataset
- query_times_table2.csv: query times / throughput and kmer-share counts per batch and index configuration

## How to apply

Build a file-of-files manifest (one genome path per line, e.g. via 'readlink -f my_folder') listing the complete Salmonella enterica RefSeq genomes. Clone and compile PAC from https://github.com/Malfoy/PAC with make (requires g++ >= 8 and zlib). Construct the index with './PAC -f fof.txt -d index_test', optionally parallelizing with -c and checking 'ulimit -n' against the partition count -p (index is split into 4^p files). Query with './PAC -l index_test -q <batch>' using 16 query threads (-c 16) on three batches (S. enterica, E. coli, random), recording elapsed time and per-query shared kmer counts (output lines report kmers shared with each indexed entry). Repeat across index configurations — plain Bloom matrix (no aggregated tree), one, and two Aggregated Bloom Comb trees — holding all other parameters fixed, then compile throughput and kmer-share counts into query_times_table2.csv.

## Related tools

- **PAC** (Builds the partitioned aggregated Bloom comb tree index and executes the query benchmark across zero/one/two aggregated tree configurations) — https://github.com/Malfoy/PAC
- **g++ 8 or higher** (C++17-ready compiler required to build PAC from source with make)
- **zlib** (Dependency (zlib1g-dev on Debian/Ubuntu) enabling PAC to read gzipped FASTA/FASTQ inputs)
- **Cobs** (Referenced Bloom-filter-based AMQ baseline in the same class of structures as PAC)
- **HowDeSBT** (Referenced Sequence Bloom tree tool contextualizing PAC among prior AMQ approaches)

## Examples

```
./PAC -l index_test -q test/GCF_000864885.1_ViralProj15500_genomic.fna.gz -c 16
```

## Evaluation signals

- The aggregated configurations (one or two Aggregated Bloom Comb trees) must show a several-fold query-throughput gain over the plain Bloom matrix on the dissimilar (E. coli) batch, consistent with the paper's Table 2.
- Little or no query-time improvement should be observed on the highly similar S. enterica batch when adding one aggregated Bloom filter.
- The second aggregated Bloom filter speeds queries further but with diminishing benefit relative to the first.
- Each output query line must report per-dataset shared kmer counts (e.g. '29719 50 701' meaning shared kmers with first, second, third entries).
- Index construction should succeed without extra disk footprint; check 'ulimit -n' covers the chosen partition count (4^p files).

## Limitations

- Larger Bloom filters (via -b) decrease false positive rate but increase index size — an explicit size/accuracy trade-off.
- Query and construction memory usage can increase linearly with the number of threads (-c).
- Default 16-bit integers limit the index to 65,636 files; use -e 32 for larger collections and -e 8 for up to 256 files.
- PAC by default only indexes kmers seen at least two times in a dataset; kmers seen only once are filtered (unless -u semantics changed), affecting sensitivity on low-coverage inputs.
- Aggregated Bloom filters hardly improve query time on highly similar batches (e.g. S. enterica queries against an S. enterica index).

## Evidence

- [intro] PAC as AMQ structure: "PAC is an AMQ (Approximate membership Query) structure based on bloom filters like Cobs, or Sequence Bloom tree tools (HowDeSBT,AllsomeSBT,SSBT,...)"
- [readme] index construction command: "PAC will construct an index in the folder index_test using all the datasets containend in the file of file fof.txt."
- [readme] partition count and ulimit: "PAC will split the index into 4^p files inside the index_test folder. Please check that you can open enough file with the ulimit command"
- [readme] output kmer share format: "Meaning that the query share 29719 kmers with the first entry, 50 with the second and 701 with the third."
- [methods] kmer filtering: "PAC will only index kmers seen at least two times in a given dataset (kmers seen only once are filtered)"
- [abstract] scalability claim: "PAC is a scalable structure able to list datasets that may contain a query sequence among thousands of indexed files"
