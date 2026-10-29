---
name: genome-collection-index-construction
description: Use when a collection of FASTA/MultiLine FASTA/FASTQ datasets (gzipped
  or not) must be indexed so that future sequence queries can list which datasets
  contain the query, with bounded RAM, disk footprint, and construction time.
when_to_use_negative:
- When the goal is precise alignment or full similarity search (e.g. BLAST-like scoring)
  rather than approximate membership listing of candidate datasets — PAC lists datasets
  that may contain the query.
- When the collection contains more than 65,536 files and you cannot raise the encoding
  (PAC defaults to 16-bit integers, indexing up to 65,636 files; use -e 32 or more).
- When the system cannot open enough files for the chosen partition count — the README
  warns to check `ulimit -n` (e.g. 5000) when using -p.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3277
- http://edamontology.org/topic_0622
tools:
- name: PAC
  role: 'Primary tool: constructs the Partitioned Aggregated Bloom comb tree index
    from a file-of-files of genomic datasets; supports -b (Bloom filter size), -p
    (partitioning), -c (threads), -u (single-kmer filter), -e (encoding width)'
  repo: https://github.com/Malfoy/PAC
- name: g++ 8 or higher
  role: C++17-ready compiler required to build PAC from source with `make`
- name: zlib
  role: Compression library dependency (zlib1g-dev on Debian/Ubuntu) required for
    reading gzipped input datasets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:19712466dcd0bb11b6383176001a6ed3a487c1b5181e422585c3aedbe0f94712
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - g++ 8 or higher
  - zlib
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3277
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# genome-collection-index-construction

## Summary

Build a scalable approximate membership query (AMQ) index over thousands to hundreds of thousands of genomic datasets using PAC (Partitioned Aggregated Bloom Comb trees), enabling fast identification of which datasets contain a query sequence.

## When to use

Use when a collection of FASTA/MultiLine FASTA/FASTQ datasets (gzipped or not) must be indexed so that future sequence queries can list which datasets contain the query, with bounded RAM, disk footprint, and construction time.

## When NOT to use

- When the goal is precise alignment or full similarity search (e.g. BLAST-like scoring) rather than approximate membership listing of candidate datasets — PAC lists datasets that may contain the query.
- When the collection contains more than 65,536 files and you cannot raise the encoding (PAC defaults to 16-bit integers, indexing up to 65,636 files; use -e 32 or more).
- When the system cannot open enough files for the chosen partition count — the README warns to check `ulimit -n` (e.g. 5000) when using -p.

## Inputs

- File-of-files (fof.txt) with one absolute dataset path per line, produced e.g. via `readlink -f` over a folder of assemblies
- FASTA, MultiLine FASTA, or FASTQ dataset files, gzipped or uncompressed
- Build parameters: -b Bloom filter size (bits), -k kmer size (default 21 per README example), -p partitions, -c threads, -e integer encoding width, -u single-kmer filter flag

## Outputs

- PAC index directory containing partitioned aggregated Bloom filter files (e.g. 4^p files when -p is set)
- Resource profile of the construction run: wall-clock hours, accumulated CPU hours, peak RSS memory, and on-disk index size (e.g. via `du -sh index_dir`)

## How to apply

Clone and compile PAC from https://github.com/Malfoy/PAC with a C++17-ready g++ (version 8 or higher) and zlib (zlib1g-dev on Debian/Ubuntu). Assemble a file-of-files where each line names one dataset to index (generate it with `readlink -f` over the dataset folder), then run `./PAC -f fof.txt -d <index_dir>`; each line of the fof is treated as a separate file to index. Choose the Bloom filter size with `-b` (the structure sizes filters to the first power of two above the value; larger filters decrease false positive rate but increase index size), optionally use `-p` partitioning to split the index into 4^p files and reduce memory, and `-c` for threads (memory usage can increase linearly with thread count). Optionally use `-u` to filter kmers seen only once, and set `-e 32` if more than 65,536 files are indexed. For GenBank-scale runs, the reported reference scale is 2^27-bit Bloom filters (~4% FP rate for 5 Mb genomes), a 24 h / 410 CPU-hour construction under 5 GB RAM producing a 3.5 TB index; wrap the run with a resource profiler (e.g. /usr/bin/time -v) to capture wall-clock, CPU hours, and peak RSS. Construction is streaming, with no disk footprint beyond the index itself.

## Related tools

- **PAC** (Primary tool: constructs the Partitioned Aggregated Bloom comb tree index from a file-of-files of genomic datasets; supports -b (Bloom filter size), -p (partitioning), -c (threads), -u (single-kmer filter), -e (encoding width)) — https://github.com/Malfoy/PAC
- **g++ 8 or higher** (C++17-ready compiler required to build PAC from source with `make`)
- **zlib** (Compression library dependency (zlib1g-dev on Debian/Ubuntu) required for reading gzipped input datasets)

## Examples

```
git clone --depth 1 --recursive https://github.com/Malfoy/PAC.git; cd PAC; make; readlink -f my_folder > fof.txt; /usr/bin/time -v ./PAC -f fof.txt -d index_genbank -b 134217728
```

## Evaluation signals

- construction_resources.tsv exists with fields for wall-clock hours, CPU hours, peak memory, and index size
- The index directory contains the partitioned aggregated Bloom filter files (4^p files when -p is used)
- For GenBank-scale reproduction: reported reference values are 24 h wall-clock, 410 CPU hours, <5 GB RAM, 3.5 TB index with 2^27-bit Bloom filters (~4% FP rate for 5 Mb genomes)
- Construction is streaming — verify no disk footprint beyond the index itself during the run
- Querying the finished index (./PAC -l index_test -q query.fasta) returns kmer-sharing counts per indexed dataset as expected

## Limitations

- Larger Bloom filters decrease the false positive rate but increase index size; the -b value is rounded up to the next power of two.
- Memory usage during construction can increase linearly with the number of threads (-c).
- By default PAC uses 16-bit integers and can index up to 65,636 files; more files require -e 32 or higher.
- Partitioning (-p) creates 4^p files and requires sufficient open-file limits (`ulimit -n`, e.g. 5000).

## Evidence

- [full_text] PAC indexed the entire GenBank bacterial genome collection (2^27-bit Bloom filters, ~4% FP rate for 5 Mb genomes) with a 24 h construction lasting 410 CPU hours using less than 5 GB of RAM, producing a 3.5 TB index.: [evidence span withheld — anchor status: not_found]
- [abstract] PAC index construction works in a streaming fashion without any disk footprint besides the index itself.: "PAC index construction works in a streaming fashion without any disk footprint besides the index itself"
- [readme] A modern, C++17 ready compiler such as `g++` version 8 or higher.: "A modern, C++17 ready compiler such as `g++` version 8 or higher"
- [readme] Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824). Larger Bloom filters decrease false positive rate but increase the index size.: "Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824)"
- [readme] PAC will use 16 threads to accelarate the index construction. PAC is able to efficiently use a high amount of threads to divide the construction time but the memory usage can increase linearly with the amount of threads used.: "the memory usage can increase linearly with the amount of threads used"
- [methods] PAC will construct an index in the folder index_test using all the datasets containend in the file of file fof.txt.: "PAC will construct an index in the folder index_test using all the datasets containend in the file of file fof.txt"
