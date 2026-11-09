---
name: incremental-index-update
description: Use when a previously built PAC index must grow with additional sequence
  datasets — e.g. appending the next batch of SRA FASTQ/FASTA samples via update mode
  to reach ~32,000-sample scale where competitors (SeqOthello, HowDeSBT) fail.
when_to_use_negative:
- No prior PAC index exists for the collection — use default index construction (`./PAC
  -f fof.txt -d index_test`) instead of update mode
- The query task needs exact sequence search in a small collection, where BLAST-like
  methods suffice per the paper
- An exact membership answer is required — PAC is an approximate membership query
  (AMQ) structure with a tunable false-positive rate
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: PAC
  role: Constructs and updates the partitioned aggregated Bloom comb tree index incrementally
    via -l/-d update mode
  repo: https://github.com/Malfoy/PAC
- name: g++ 8 or higher
  role: C++17-ready compiler required to build PAC from source before running incremental
    updates
- name: zlib
  role: Dependency allowing PAC to read gzipped FASTA/FASTQ inputs in the appended
    file of files
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:a88ef821d9333af4005759fcf36b2f9a0554e86fb768599a9b0e2162dfdfef87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - g++ 8 or higher
  - zlib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# incremental-index-update

## Summary

Incrementally extends an existing PAC (Partitioned Aggregated Bloom comb tree) AMQ index by appending a new file-of-files of sequence datasets in update mode, enabling scalable index growth across batches (demonstrated up to 32,768 SRA human RNA-seq samples) without rebuilding from scratch.

## When to use

Use when a previously built PAC index must grow with additional sequence datasets — e.g. appending the next batch of SRA FASTQ/FASTA samples via update mode to reach ~32,000-sample scale where competitors (SeqOthello, HowDeSBT) fail.

## When NOT to use

- No prior PAC index exists for the collection — use default index construction (`./PAC -f fof.txt -d index_test`) instead of update mode
- The query task needs exact sequence search in a small collection, where BLAST-like methods suffice per the paper
- An exact membership answer is required — PAC is an approximate membership query (AMQ) structure with a tunable false-positive rate

## Inputs

- Previously built PAC index directory (e.g. index_test)
- File of files listing new sequence datasets to add (one FASTA/MultiLine FASTA/FASTQ path per line, gzipped or not)
- Parameters: k-mer size (e.g. -k 31), -u single-occurrence k-mer filter, -p partitions, -c threads, -e integer width

## Outputs

- Updated PAC index directory (e.g. index_updated) containing the original plus newly indexed samples
- Per-batch construction metrics: CPU time, temporary disk usage, final index size, peak RAM

## How to apply

Compile PAC from https://github.com/Malfoy/PAC with make (requires a C++17-ready compiler such as g++ 8+ and zlib). For each incremental batch, prepare a file of files (one dataset path per line; FASTA, MultiLine FASTA or FASTQ, gzipped or not) and run update mode with `-l <previous_index> -d <index_updated>`, indexing 31-mers (`-k 31`) and filtering k-mers seen only once (`-u`) since PAC indexes k-mers seen at least two times in a dataset. Choose partitioning (`-p`, splitting the index into 4^p files after raising `ulimit -n`) and thread count (`-c`): more partitions usually speed up construction and reduce memory, but memory can grow linearly with thread count. If the collection exceeds 65,636 indexed files, pass `-e 32` because the default 16-bit aggregated Bloom filter integers cap at that file count. After each batch, record construction CPU time, temporary disk usage, final index size, and peak RAM to validate the run against the reported scale (32,000 human RNA-seq samples in ~5 days, ~22 GB RAM, ~1.1 TB index).

## Related tools

- **PAC** (Constructs and updates the partitioned aggregated Bloom comb tree index incrementally via -l/-d update mode) — https://github.com/Malfoy/PAC
- **g++ 8 or higher** (C++17-ready compiler required to build PAC from source before running incremental updates)
- **zlib** (Dependency allowing PAC to read gzipped FASTA/FASTQ inputs in the appended file of files)

## Examples

```
./PAC -f fof_batch_next.txt -l index_previous -d index_updated -k 31 -u -c 16
```

## Evaluation signals

- Incremental builds complete across all batches up to 32,768 samples where SeqOthello crashes (2^12 batch) and HowDeSBT times out (2^11 batch)
- Final index size and resource profile approach the reported scale: ~1.1 TB index, ~22 GB RAM, ~5 days (~60 CPU days) for ~32,000 human RNA-seq samples
- The -d output index directory contains 4^p partition files after running with -p, and update mode exited without requiring a rebuild of prior batches
- Single-occurrence k-mers are excluded: with -u, PAC only indexes k-mers seen at least two times in each dataset
- If the file count exceeds 65,636, the run required -e 32 (16-bit integers would otherwise cap the indexable file count)

## Limitations

- Memory usage can increase linearly with the number of threads used (-c)
- More partitions (-p) speed up and reduce memory during construction but create more files and require a raised open-file limit (e.g. ulimit -n 5000)
- Default 16-bit integers index up to 65,636 files; larger collections need -e 32
- Larger Bloom filters (larger -b) decrease false-positive rate but increase index size

## Evidence

- [readme] PAC will only index kmers seen at least two times in a given dataset (kmers seen only once are filtered): "PAC will only index kmers seen at least two times in a given dataset (kmers seen only once are filtered)"
- [readme] Bydefault PAC use 16 bits integers and can index up to 65,636 files, if you need to index more files use -e 32: "Bydefault PAC use 16 bits integers and can index up to 65,636 files, if you need to index more files use -e 32"
- [readme] More partitions usually speed up and reduce the memory usage during construction and query operations but create more files.: "More partitions usually speed up and reduce the memory usage during construction and query operations but create more files."
- [abstract] They include 32 000 human RNA-seq samples in 5 days: "They include 32 000 human RNA-seq samples in 5 days"
- [intro] A PAC index can be built from thousands datasets quickly and with low ressource memory and disk footprint: "A PAC index can be built from thousands datasets quickly and with low ressource memory and disk footprint"
