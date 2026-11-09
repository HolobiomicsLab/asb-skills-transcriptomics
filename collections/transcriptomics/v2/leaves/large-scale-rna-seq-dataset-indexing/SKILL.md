---
name: large-scale-rna-seq-dataset-indexing
description: Use when indexing very large collections (thousands to ~32,000+) of RNA-seq
  or other sequence datasets where Bloom-filter-based AMQ tools hit construction-time
  or memory barriers, and the index must grow incrementally as batches of samples
  arrive.
when_to_use_negative:
- Indexing a small collection of genomes where BLAST-like methods routinely suffice
  — PAC targets thousands-to-tens-of-thousands of datasets
- Constructions where the approximate-membership (with false positives) listing semantics
  of PAC are unacceptable and exact per-read presence is required
- Collections larger than the encoded integer capacity — default 16-bit integers index
  up to ~65,636 files, so -e 32 must be set or smaller batches used
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0622
tools:
- name: PAC
  role: Core AMQ data structure; constructs, updates, and queries the partitioned
    aggregated Bloom comb tree index over RNA-seq sample batches
  repo: https://github.com/Malfoy/PAC
- name: g++ 8 or higher
  role: C++17-ready compiler required to build PAC from source with make
- name: zlib
  role: Compression library dependency (zlib1g-dev on Debian/Ubuntu) needed to compile
    PAC and read gzipped inputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:ce1615fb098da2ab4562bbe6753879ffd43c6602439815c420aac17bd53c0b97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - g++ 8 or higher
  - zlib
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# large-scale-rna-seq-dataset-indexing

## Summary

Build and incrementally extend a PAC (Partitioned Aggregated Bloom Comb trees) approximate membership query index over tens of thousands of SRA RNA-seq samples, a scale where competing AMQ structures such as HowDeSBT and SeqOthello fail to complete construction.

## When to use

Use when indexing very large collections (thousands to ~32,000+) of RNA-seq or other sequence datasets where Bloom-filter-based AMQ tools hit construction-time or memory barriers, and the index must grow incrementally as batches of samples arrive.

## When NOT to use

- Indexing a small collection of genomes where BLAST-like methods routinely suffice — PAC targets thousands-to-tens-of-thousands of datasets
- Constructions where the approximate-membership (with false positives) listing semantics of PAC are unacceptable and exact per-read presence is required
- Collections larger than the encoded integer capacity — default 16-bit integers index up to ~65,636 files, so -e 32 must be set or smaller batches used

## Inputs

- Batches of raw SRA RNA-seq sample files (FASTA, MultiLine FASTA, or FASTQ, gzipped or not)
- File of files (fof) per batch: plain-text list with one sample file path per line
- Previously built PAC index folder for incremental updates

## Outputs

- Partitioned PAC index folder (4^p files, e.g. a 1.1 TB index for ~32,000 samples)
- Per-batch construction logs: CPU time, peak RAM, temporary disk usage, final index size

## How to apply

Compile PAC from https://github.com/Malfoy/PAC with a C++17-ready compiler (g++ 8+) and zlib, then generate a file of files (one indexed file per line, e.g. via 'readlink -f my_folder') listing the FASTA/MultiLine FASTA/FASTQ samples per batch. For each batch, run PAC in update mode (-f <fof_batch> -l <previous_index> -d <index_updated> -k 31 -u), appending the new samples to the previously built index while indexing 31-mers and filtering k-mers seen only once via the -u option. Optionally pass -p to partition the index into 4^p files (raising the open-file limit with 'ulimit -n 5000' if needed) and -c to set thread count, since more partitions usually speed up construction and reduce memory usage, while memory can increase linearly with threads. Encode enough bits per entry (-e) to cover the collection size: the default 16-bit integers index up to ~65,536 files, so use -e 32 for larger collections. After each batch, record construction CPU time, temporary disk usage, final index size, and peak RAM; validate that the incremental build completes across all batches, matching the reported scale (32,000 human RNA-seq samples indexed in ~5 days / ~60 CPU days, 22 GB RAM, 1.1 TB index).

## Related tools

- **PAC** (Core AMQ data structure; constructs, updates, and queries the partitioned aggregated Bloom comb tree index over RNA-seq sample batches) — https://github.com/Malfoy/PAC
- **g++ 8 or higher** (C++17-ready compiler required to build PAC from source with make)
- **zlib** (Compression library dependency (zlib1g-dev on Debian/Ubuntu) needed to compile PAC and read gzipped inputs)

## Examples

```
./PAC -f <fof_batch> -l <previous_index> -d <index_updated> -k 31 -u -c 16 -p 5
```

## Evaluation signals

- Incremental construction completes across all batches up to 32,768 samples, matching the reported 32,000-sample scale
- Resource footprint falls within reported ranges: ~5 days (~60 CPU days) construction, ~22 GB peak RAM, ~1.1 TB final index
- Update-mode runs (PAC -f <fof_batch> -l <previous_index> -d <index_updated>) preserve the prior index content and correctly extend it with the new file of files
- Index folder contains the expected 4^p partition files when -p is used, and no construction crash or timeout occurs (unlike SeqOthello at 2^12 batch or HowDeSBT at 2^11 batch)
- Construction works in a streaming fashion with no disk footprint beyond the index itself

## Limitations

- Memory usage can increase linearly with the number of threads used (-c)
- Partitioning (-p) creates more files, requiring a sufficient open-file limit (e.g. ulimit -n 5000)
- Default 16-bit encoding caps the indexable file count at ~65,636; larger collections need -e 32
- Larger Bloom filters (-b) decrease false positive rate but increase index size
- Competing AMQ structures may not complete at this scale: SeqOthello crashed at the 2^12 batch and HowDeSBT timed out at the 2^11 batch

## Evidence

- [results] scalability experiment on 32 768 SRA RNA-seq samples: [evidence span withheld — anchor status: not_found]
- [abstract] PAC indexed 32 000 human RNA-seq samples in 5 days: "They include 32 000 human RNA-seq samples in 5 days"
- [abstract] PAC index construction works in a streaming fashion without any disk footprint besides the index itself: "PAC index construction works in a streaming fashion without any disk footprint besides the index itself."
- [readme] compiler and zlib requirements: "A modern, C++17 ready compiler such as `g++` version 8 or higher."
- [readme] filter kmers seen only once: "PAC will only index kmers seen at least two times in a given dataset (kmers seen only once are filtered)"
- [readme] more partitions speed up and reduce memory usage: "More partitions usually speed up and reduce the memory usage during construction and query operations but create more files."
- [readme] 16-bit integer encoding capacity: "Bydefault PAC use 16 bits integers and can index up to 65,636 files, if you need to index more files use -e 32"
