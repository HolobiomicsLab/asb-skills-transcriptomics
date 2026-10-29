---
name: aggregated-bloom-comb-tree-construction
description: Use when many sequence datasets (FASTA/FASTQ, gzip-compressed or not)
  must be indexed together so later queries can list which datasets share k-mers with
  a query sequence, especially when index construction time and disk footprint must
  stay low.
when_to_use_negative:
- The task needs exact (not approximate) membership answers — PAC is an AMQ structure
  and its Bloom filters admit false positives controlled by the chosen -b bit count
- The collection is so small that a BLAST-like alignment search is routine and index
  construction overhead is not justified
- More files than the chosen encoding allows must be indexed (e.g. >256 files with
  -e 8); use the default 16-bit or -e 32 instead
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: PAC
  role: Constructs the partitioned aggregated Bloom Comb tree index and runs single/multithread
    queries against it
  repo: https://github.com/Malfoy/PAC
- name: g++ 8 or higher
  role: C++17-ready compiler used to build PAC via make
- name: zlib
  role: Required dependency so PAC can read gzip-compressed FASTA/FASTQ datasets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:7685d1d55e59b327d540693ad8878b88f63c1afd340a121342f845f653b55bf7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
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
yes_rate: 0.5454545454545454
n_claims: 11
---

# aggregated-bloom-comb-tree-construction

## Summary

Constructs a PAC (Partitioned Aggregated Bloom Comb trees) index: an approximate membership query structure that partitions a k-mer set by minimizers into 4^p partition files, each holding aggregated Bloom Comb filters, enabling dataset-membership queries over thousands of FASTA/FASTQ collections with low memory and disk footprint.

## When to use

Use when many sequence datasets (FASTA/FASTQ, gzip-compressed or not) must be indexed together so later queries can list which datasets share k-mers with a query sequence, especially when index construction time and disk footprint must stay low.

## When NOT to use

- The task needs exact (not approximate) membership answers — PAC is an AMQ structure and its Bloom filters admit false positives controlled by the chosen -b bit count
- The collection is so small that a BLAST-like alignment search is routine and index construction overhead is not justified
- More files than the chosen encoding allows must be indexed (e.g. >256 files with -e 8); use the default 16-bit or -e 32 instead

## Inputs

- File-of-files (fof.txt) listing one dataset path per line
- FASTA, MultiLine FASTA, or FASTQ dataset files (gzip-compressed or not)
- Optional query FASTA/FASTQ file (e.g. a .fna.gz genome) for the query step

## Outputs

- Index folder containing exactly 4^p partition files, each an aggregated Bloom Comb filter with a lazily constructed inverted Bloom filter index mapping k-mer hash h to the bit slice of Bloom filters containing it
- Per-query-line output: an n-sized bit vector of per-dataset hits and a score vector of shared k-mer counts (e.g. '29719 50 701')
- Updated index folder when adding a new file-of-files to a previously built index

## How to apply

Clone https://github.com/Malfoy/PAC recursively and build with make, requiring a C++17-capable g++ (version 8 or higher) and zlib (zlib1g-dev on Debian/Ubuntu). Create a file-of-files listing one dataset per line (e.g. via 'readlink -f my_folder'), then run indexing with -f and -d; choose k with -k (e.g. 21), split the k-mer set into 4^p partitions with -p (e.g. -p 2 yields 16 partition files), size Bloom filters with -b (sized to the first power of two above the requested bit count), filter singletons with -u, and select integer encoding with -e (default 16 bits, up to 65,536 files; 8 bits for up to 256). Check ulimit -n so enough partition files can be opened. Indexing is streaming with no disk footprint beyond the index itself; afterwards queries with -l and -q return, per query line, an n-sized score vector of shared k-mer counts, optionally multithreaded with -c. Existing indexes can be extended with a new file-of-files using -f with both -l and -d.

## Related tools

- **PAC** (Constructs the partitioned aggregated Bloom Comb tree index and runs single/multithread queries against it) — https://github.com/Malfoy/PAC
- **g++ 8 or higher** (C++17-ready compiler used to build PAC via make)
- **zlib** (Required dependency so PAC can read gzip-compressed FASTA/FASTQ datasets)

## Examples

```
./PAC -f test/fof.txt -d index_test -k 21 -p 2
```

## Evaluation signals

- The index folder contains exactly 4^p partition files for the chosen -p (e.g. 16 files with -p 2)
- Query output yields an n-sized score vector of shared k-mer counts per query line, matching expected values for known sequences (e.g. '29719 50 701' for the SARS coronavirus Tor2 test file)
- False positive rate stays acceptable given the -b sizing rule (first power of two above the requested bit count)
- Indexed k-mer length matches -k (e.g. 21mers) and -u correctly drops kmers seen only once
- Index update with a new file-of-files produces an index_updated folder loadable for further queries

## Limitations

- Bloom filter false positives are inherent; larger filters (-b) reduce them but increase index size
- Memory usage can increase linearly with the number of construction/query threads (-c)
- More partitions (-p) speed up construction and queries but create more files; ulimit -n may need raising (e.g. 'ulimit -n 5000')
- Integer encoding caps the number of indexable files: default 16 bits up to 65,636 files, -e 8 up to 256 files; use -e 32 for more

## Evidence

- [readme] PAC will split the index into 4^p files inside the index_test folder.: "PAC will split the index into 4^p files inside the index_test folder."
- [readme] Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824).: "Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824). Larger  Bloom filters decrease false positive rate but increase the index size."
- [abstract] PAC index construction works in a streaming fashion without any disk footprint besides the index itself: "PAC index construction works in a streaming fashion without any disk footprint besides the index itself."
- [methods] PAC will only index kmers seen at least two times in a given dataset (kmers seen only once are filtered): "PAC will only index kmers seen at least two times in a given dataset (kmers seen only once are filtered)"
