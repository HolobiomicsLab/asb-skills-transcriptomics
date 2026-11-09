---
name: bloom-filter-aggregated-encoding
description: Use when compressing per-dataset Bloom filters of a sequence collection
  into a run-length-encoded (Aggregated) Bloom filter, or building left/right Bloom
  comb trees for a PAC-style approximate membership index over thousands of FASTA/FASTQ
  datasets.
when_to_use_negative:
- The application requires exact (no-false-positive) membership answers — PAC is an
  AMQ structure, like Cobs and Sequence Bloom Tree tools, that only lists datasets
  that may contain the query
- The target collection exceeds the file-count ceiling of the chosen integer width,
  e.g. more than 256 files with -e 8 or more than 65,636 with the default 16-bit encoding
- Indexing k-mers seen only once matters, since PAC filters singletons by default
  in -u mode and the -u option restricts indexing to k-mers seen at least twice
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: PAC
  role: Reference implementation that constructs the Aggregated Bloom filter encoding
    and the partitioned aggregated Bloom comb tree index from a file-of-files of datasets
  repo: https://github.com/Malfoy/PAC
- name: g++
  role: C++17-ready compiler (version 8 or higher) used to build PAC from source before
    running the encoding pipeline
- name: zlib
  role: Required dependency for PAC to read gzip-compressed FASTA/FASTQ inputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:e2d5ffc8562431ac5bb10f171e20ed3860216e755a88b685539f35518a7525e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - g++
  - zlib
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# bloom-filter-aggregated-encoding

## Summary

Encodes the nested series of Bloom filters (Rt⊆…⊆R1) found along the internal nodes of PAC's comb trees as an Aggregated Bloom filter: a t×b matrix of stacked filters compressed into a run-length integer array V of size b, which PAC composes into partitioned aggregated Bloom comb trees for approximate membership queries over dataset collections.

## When to use

Use when compressing per-dataset Bloom filters of a sequence collection into a run-length-encoded (Aggregated) Bloom filter, or building left/right Bloom comb trees for a PAC-style approximate membership index over thousands of FASTA/FASTQ datasets.

## When NOT to use

- The application requires exact (no-false-positive) membership answers — PAC is an AMQ structure, like Cobs and Sequence Bloom Tree tools, that only lists datasets that may contain the query
- The target collection exceeds the file-count ceiling of the chosen integer width, e.g. more than 256 files with -e 8 or more than 65,636 with the default 16-bit encoding
- Indexing k-mers seen only once matters, since PAC filters singletons by default in -u mode and the -u option restricts indexing to k-mers seen at least twice

## Inputs

- Collection of FASTA, MultiLine FASTA or FASTQ datasets, gzip-compressed or not, listed one per line in a file-of-files (fof.txt)
- k-mer size parameter (PAC example: k=21)
- Requested Bloom filter bit count (PAC example: 1,000,000,000 bits, rounded up to the next power of two)
- Partition depth p controlling the number of 4^p prefix-split files (PAC example: p=5)
- Aggregation-value integer width (-e 8, 16 default, or 32 for very large collections)

## Outputs

- Aggregated Bloom filter encoded as the run-length array V (size b) of the nested series of Bloom filters (Definitions 1, 3)
- Left and right Bloom comb trees built by bf_merge of child filters (Definitions 4-5)
- Paired Aggregated Bloom Comb Tree index folder (Definition 6) with the encoded arrays and comb structure files written to disk
- Observed false-positive rate from validation

## How to apply

Generate one Bloom filter per dataset (index k-mers, e.g. k=21), sizing each filter to the next power of two above the requested bit count — the PAC example uses -b 1000000000, which rounds to 1,073,741,824 bits. Merge child filters up the tree by bitwise OR (bf_merge, Definition 1), then encode the nested series of the comb's longest branch as the Aggregated Bloom filter (Definition 3): stack the t filters into a t×b matrix M and store, for each row i of length b, the run of 1s as an integer V[i], so t Bloom filters of size b are represented by b integers of size S (b×log t bits). Store aggregation values as 8-bit integers with the -e 8 mode (up to 256 indexed files) instead of the default 16-bit integers (up to 65,636 files). Partition leaf k-mer sets into 4^p prefix files (PAC example p=5, i.e. 4^5 files) to build the left and right comb trees (Definitions 4-5), and assemble the paired Aggregated Bloom Comb Tree (Definition 6) whose two combs are intersected at query time to restrict search-space intervals. Validate by a round-trip membership check: re-query the encoded structures with the indexed k-mers and confirm every k-mer returns its ground-truth containing dataset(s) with no false negatives, while reporting the observed false-positive rate.

## Related tools

- **PAC** (Reference implementation that constructs the Aggregated Bloom filter encoding and the partitioned aggregated Bloom comb tree index from a file-of-files of datasets) — https://github.com/Malfoy/PAC
- **g++** (C++17-ready compiler (version 8 or higher) used to build PAC from source before running the encoding pipeline)
- **zlib** (Required dependency for PAC to read gzip-compressed FASTA/FASTQ inputs)

## Examples

```
./PAC -f test/fof.txt -d index_test -k 21 -b 1000000000 -p 5 -e 8
```

## Evaluation signals

- Round-trip membership check: re-querying the encoded Aggregated Bloom Comb Tree with the indexed k-mers returns each k-mer's ground-truth containing dataset(s) (no false negatives)
- Reported false-positive rate is consistent with the chosen filter size (larger Bloom filters decrease false positive rate but increase index size)
- Each per-dataset Bloom filter is sized to the first power of two above the requested bit count (e.g. 1,000,000,000 bits becomes 1,073,741,824)
- The run-length array V has exactly b integers, one per row position of the t×b stacked-filter matrix, matching the -e encoding width (8/16/32 bits)
- The partitioned index folder contains 4^p files (e.g. 4^5 = 1024 files for p=5), each openable within the ulimit -n file-descriptor budget

## Limitations

- The Aggregated Bloom filter is an approximate membership structure: queries can produce false positives, controlled by filter size (-b) at the cost of index size
- Encoding width caps the number of indexable files: -e 8 supports up to 256 files, the default 16 bits up to 65,636 files; larger collections require -e 32
- More partitions (4^p files) speed up construction and queries and reduce memory usage but create more files, and require a sufficient ulimit for open file descriptors
- Construction memory usage can increase linearly with the number of threads used (-c)

## Evidence

- [abstract] PAC, a novel approximate membership query data structure for querying collections of sequence datasets: "Here, we present PAC, a novel approximate membership query data structure for querying collections of sequence datasets."
- [intro] PAC scalable structure listing datasets: "PAC is a scalable structure able to list datasets that may contain a query sequence among thousands of indexed files."
- [readme] 8-bit aggregated bloom filter encoding: "PAC will use 8 bits integers to encode its aggragated bloom filters, this way PAC can index up to 256 files."
- [readme] Bloom filter sizing to next power of two: "Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824)."
