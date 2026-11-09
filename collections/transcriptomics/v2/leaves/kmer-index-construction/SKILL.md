---
name: kmer-index-construction
description: Use when a collection of many sequence datasets (hundreds to hundreds
  of thousands of FASTA/MultiLine FASTA/FASTQ, gzipped or not) needs an approximate
  membership query index so later queries can list which datasets share k-mers with
  a query sequence.
when_to_use_negative:
- The goal is exact alignment/local similarity scoring against a small genome collection
  (BLAST-like methods suffice at that scale).
- The task requires exact membership answers rather than an approximate membership
  query structure with Bloom-filter false positives.
- Input is a single small reference genome where a full AMQ index over thousands of
  datasets is unnecessary overhead.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: PAC
  role: Primary AMQ data structure; builds, queries, and updates the partitioned aggregated
    Bloom comb-tree index from a file-of-files.
  repo: https://github.com/Malfoy/PAC
- name: g++ 8 or higher
  role: C++17-ready compiler required to build PAC from source with make.
- name: zlib
  role: Required library enabling PAC to read gzipped FASTA/FASTQ inputs (zlib1g-dev
    on Debian/Ubuntu).
- name: Cobs
  role: Alternative Bloom-filter-based AMQ structure used as a comparison method for
    construction cost and index size benchmarks.
- name: HowDeSBT
  role: Sequence Bloom tree comparison method for construction-time benchmarks on
    the 2585 human RNA-seq dataset (built with kmtricks solid k-mers at k=21).
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:7b48a0abcf436d37bd61f6afb2e7b2ef4b2f7c2d4efe6d6cb646379b7b98e1a1
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# kmer-index-construction

## Summary

Build a PAC (Partitioned Aggregated bloom Comb trees) k-mer index — an approximate membership query (AMQ) structure over thousands of FASTA/FASTQ datasets — that lists which indexed datasets may contain a query sequence, with streaming construction and no disk footprint beyond the index itself.

## When to use

Use when a collection of many sequence datasets (hundreds to hundreds of thousands of FASTA/MultiLine FASTA/FASTQ, gzipped or not) needs an approximate membership query index so later queries can list which datasets share k-mers with a query sequence.

## When NOT to use

- The goal is exact alignment/local similarity scoring against a small genome collection (BLAST-like methods suffice at that scale).
- The task requires exact membership answers rather than an approximate membership query structure with Bloom-filter false positives.
- Input is a single small reference genome where a full AMQ index over thousands of datasets is unnecessary overhead.

## Inputs

- File-of-files (fof.txt) listing one dataset path per line
- Sequence dataset files: FASTA, MultiLine FASTA, or FASTQ, gzipped or uncompressed
- Previously built PAC index folder (for update mode, passed with -l)

## Outputs

- PAC index folder (e.g. index_test) containing 4^p partition files
- Optionally updated PAC index folder when appending a new file-of-files in update mode
- Construction resource metrics (wall-clock time, peak RAM, on-disk index size)

## How to apply

Compile PAC from the cloned repository with make, requiring a C++17-ready compiler (g++ 8 or higher) and zlib (zlib1g-dev on Debian/Ubuntu). Assemble the datasets into a file-of-files where each line is one file to index (e.g. 'readlink -f' over the download folder), then run './PAC -f fof.txt -d index_test' to construct the index in the target folder. Choose parameters deliberately: -k for the k-mer size (e.g. 21 or 31), -b to size the per-dataset Bloom filters to the first power of two above a bit budget (larger filters lower false positive rate but increase index size), -u to index only k-mers seen at least twice in a dataset, -p to split the index into 4^p partition files (raising 'ulimit -n' as needed; more partitions usually speed up construction and reduce memory), -c for thread count (memory can increase linearly with threads), and -e 8/32 to switch aggregated Bloom filter integer encoding when indexing more than 65,536 files. To extend a collection, use update mode ('./PAC -f fof_add.txt -l index_test -d index_updated') to append a new file-of-files to a previously built index. PAC streams its construction, so peak RAM stays low (e.g. ~21 GB or ~6 GB for the 661k bacterial genomes at 2^29- vs 2^27-bit filters) and no temporary disk beyond the index itself is used.

## Related tools

- **PAC** (Primary AMQ data structure; builds, queries, and updates the partitioned aggregated Bloom comb-tree index from a file-of-files.) — https://github.com/Malfoy/PAC
- **g++ 8 or higher** (C++17-ready compiler required to build PAC from source with make.)
- **zlib** (Required library enabling PAC to read gzipped FASTA/FASTQ inputs (zlib1g-dev on Debian/Ubuntu).)
- **Cobs** (Alternative Bloom-filter-based AMQ structure used as a comparison method for construction cost and index size benchmarks.)
- **HowDeSBT** (Sequence Bloom tree comparison method for construction-time benchmarks on the 2585 human RNA-seq dataset (built with kmtricks solid k-mers at k=21).)

## Examples

```
./PAC -f test/fof.txt -d index_test -k 21
```

## Evaluation signals

- The index folder contains the expected 4^p partition files after construction.
- Query output reports per-dataset shared k-mer counts, e.g. '29719 50 701' for a three-entry index.
- Resource metrics fall in the published ranges: e.g. 661k bacterial genomes in <24 h with 21 GB (2^29-bit filters) or 6 GB (2^27-bit filters) RAM and 2.4 TB / 1.4 TB index sizes; 32,768 SRA RNA-seq samples in ~5 days (~60 CPU days) with ~22 GB RAM and ~1.1 TB index.
- No disk footprint other than the index itself, confirming streaming construction.
- For update-mode runs, the new file-of-files is appended to the previous index and the updated folder loads successfully for querying.

## Limitations

- Bloom-filter sizing trades false positive rate against index size: larger filters (-b) decrease false positives but increase the index size.
- Memory usage can increase linearly with the number of threads (-c) during construction.
- Partition count (-p) creates more files: check the open-file limit ('ulimit -n 5000' allows 5000 opened files).
- Default 16-bit aggregated Bloom filter encoding indexes up to 65,536 files; use -e 8 (up to 256 files) or -e 32 for more files.

## Evidence

- [methods] PAC will construct an index in the folder index_test using all the datasets containend in the file of file fof.txt.: "PAC will construct an index in the folder index_test using all the datasets containend in the file of file fof.txt."
- [readme] Indexed Bloom filters will be sized to the first power of two above 1000000000: "Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824). Larger Bloom filters decrease false positive rate but increase the index size."
- [methods] PAC will only index kmers seen at least two times in a given dataset (kmers seen only once are filtered): "PAC will only index kmers seen at least two times in a given dataset (kmers seen only once are filtered)"
- [abstract] PAC index construction works in a streaming fashion without any disk footprint besides the index itself: "PAC index construction works in a streaming fashion without any disk footprint besides the index itself."
- [readme] A modern, C++17 ready compiler such as `g++` version 8 or higher.: "A modern, C++17 ready compiler such as `g++` version 8 or higher."
