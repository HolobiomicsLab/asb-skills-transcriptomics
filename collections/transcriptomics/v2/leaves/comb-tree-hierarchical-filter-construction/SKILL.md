---
name: comb-tree-hierarchical-filter-construction
description: Use when building an approximate membership query index over a collection
  of sequence datasets (FASTA/FASTQ, gzip allowed) where per-dataset Bloom filters
  must be hierarchically merged into comb trees and the k-mer space partitioned into
  prefix files.
when_to_use_negative:
- When exact membership answers are required — PAC is an approximate membership query
  structure and reports false positives (larger -b values lower but never eliminate
  the FPR).
- When a query must be answered against a single already-assembled sequence rather
  than deciding which of thousands of indexed datasets contain it — sequence alignment
  tools like BLAST are the wrong target here, and PAC targets dataset listing.
- When the number of indexed files exceeds the aggregation integer capacity (256 files
  with -e 8; 65,536 with default 16 bits) and no suitable -e setting can be used.
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0080
tools:
- name: PAC
  role: Primary tool that constructs per-dataset Bloom filters, performs bf_merge
    up the comb trees, encodes the Aggregated Bloom filter run-length array V, and
    writes the partitioned Aggregated Bloom Comb Tree index.
  repo: https://github.com/Malfoy/PAC
- name: g++
  role: C++17 compiler (version 8 or higher) required to build PAC from source with
    make.
- name: zlib
  role: Compression library dependency (zlib1g-dev on Debian/Ubuntu) enabling PAC
    to read gzip-compressed FASTA/FASTQ inputs during index construction.
- name: BitMagic
  role: Succinct compressed bit-vector library referenced in the article context as
    a building block for compressed bitset operations underlying Bloom filter construction.
  repo: tlk00/BitMagic
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:e52096ccd54aa011b66de2cf540246ace98a96b768e38951042c670b1ee9c9c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - BitMagic
  - PAC
  - g++
  - zlib
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# comb-tree-hierarchical-filter-construction

## Summary

Builds the left and right Bloom comb trees of a PAC index by bf_merge (bitwise OR, Definition 1) of child Bloom filters up the tree, then encodes the nested series of filters (Rt⊆…⊆R1) along the longest branch as an Aggregated Bloom filter (Definition 3) to form the paired Aggregated Bloom Comb Tree (Definition 6).

## When to use

Use when building an approximate membership query index over a collection of sequence datasets (FASTA/FASTQ, gzip allowed) where per-dataset Bloom filters must be hierarchically merged into comb trees and the k-mer space partitioned into prefix files.

## When NOT to use

- When exact membership answers are required — PAC is an approximate membership query structure and reports false positives (larger -b values lower but never eliminate the FPR).
- When a query must be answered against a single already-assembled sequence rather than deciding which of thousands of indexed datasets contain it — sequence alignment tools like BLAST are the wrong target here, and PAC targets dataset listing.
- When the number of indexed files exceeds the aggregation integer capacity (256 files with -e 8; 65,536 with default 16 bits) and no suitable -e setting can be used.

## Inputs

- File-of-files (fof.txt), one dataset path per line (FASTA, MultiLine FASTA, or FASTQ; gzip allowed)
- Parameters: k-mer size (-k 21 in the PAC example), Bloom filter bit count (-b, rounded up to next power of two), partition depth (-p), aggregation integer size (-e), single-occurrence k-mer filter (-u), thread count (-c)

## Outputs

- Paired Aggregated Bloom Comb Tree index folder (e.g., index_test) containing 4^p partition files
- Aggregated Bloom filter run-length arrays V (8- or 16-bit integers) encoding the nested series of Bloom filters per comb
- Bloom comb tree structure files built by bf_merge of child filters

## How to apply

Generate one Bloom filter per dataset, each sized to the next power of two above the requested bit count (e.g., the PAC example request of 1,000,000,000 bits rounds up to 1,073,741,824). Merge child filters up each comb tree via bf_merge, a bitwise OR of the Bloom filters (Definition 1). Encode the nested series of filters found along the internal nodes (longest branch) as an Aggregated Bloom filter: a t×b matrix M of stacked filters plus an integer array V of size b where V[i] stores the run length of 1s in row i, so t filters of size b cost b integers of size S (b×log t bits); use -e 8 for 8-bit aggregation integers (up to 256 files) or the default 16 bits (up to 65,536 files). Partition the leaf k-mer sets into 4^p prefix files (PAC example p=5, i.e., 4^5 = 1024 files), checking ulimit -n allows enough open files. Assemble the paired Aggregated Bloom Comb Tree whose two combs are intersected at query time to restrict search intervals, and optionally apply -u to index only k-mers seen at least twice.

## Related tools

- **PAC** (Primary tool that constructs per-dataset Bloom filters, performs bf_merge up the comb trees, encodes the Aggregated Bloom filter run-length array V, and writes the partitioned Aggregated Bloom Comb Tree index.) — https://github.com/Malfoy/PAC
- **g++** (C++17 compiler (version 8 or higher) required to build PAC from source with make.)
- **zlib** (Compression library dependency (zlib1g-dev on Debian/Ubuntu) enabling PAC to read gzip-compressed FASTA/FASTQ inputs during index construction.)
- **BitMagic** (Succinct compressed bit-vector library referenced in the article context as a building block for compressed bitset operations underlying Bloom filter construction.) — tlk00/BitMagic

## Examples

```
./PAC -f test/fof.txt -d index_test -k 21 -b 1000000000 -p 5 -e 8
```

## Evaluation signals

- Round-trip membership check: re-query the encoded comb trees with the indexed k-mers and confirm every k-mer returns its ground-truth containing dataset(s) — no false negatives are permissible.
- Filter sizing invariant: each Bloom filter's size is exactly the next power of two above the requested -b value (e.g., 1,000,000,000 → 1,073,741,824 bits).
- Aggregated Bloom filter size check: t Bloom filters of size b must be representable with b integers of size S = b×log t bits in the run-length array V.
- Partition file count equals 4^p inside the index folder (e.g., 4^5 = 1024 files for p=5).
- Report the observed false-positive rate from validation queries; it should decrease as -b increases and stay comparable across comparable index sizes.

## Limitations

- Larger Bloom filters decrease the false positive rate but linearly increase index size.
- More partitions (-p) speed up construction and queries and reduce memory usage but create more files; the system must allow enough open file descriptors (e.g., ulimit -n 5000).
- Multi-threaded construction (-c) increases memory usage roughly linearly with thread count.
- Aggregation integer width caps the number of indexable files: 8 bits → 256 files, default 16 bits → 65,536 files; use -e 32 for more.

## Evidence

- [readme] Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824).: "Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824). Larger Bloom filters decrease false positive rate but increase the index size."
- [readme] PAC will split the index into 4^p files inside the index_test folder.: "PAC will split the index into 4^p files inside the index_test folder. Please check that you can open enough file with the ulimit command"
- [readme] PAC will use 8 bits integers to encode its aggragated bloom filters, this way PAC can index up to 256 files.: "PAC will use 8 bits integers to encode its aggragated bloom filters, this way PAC can index up to 256 files."
- [methods] PAC will construct an index in the folder index_test using all the datasets containend in the file of file fof.txt.: "PAC will construct an index in the folder index_test using all the datasets containend in the file of file fof.txt. Each line will be considered as a separated file to index."
- [abstract] PAC index construction works in a streaming fashion without any disk footprint besides the index itself.: "PAC index construction works in a streaming fashion without any disk footprint besides the index itself."
