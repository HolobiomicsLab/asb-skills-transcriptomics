---
name: kmer-dataset-membership-indexing
description: Use when you need to ask 'which indexed dataset(s) may contain this sequence?'
  over thousands of FASTA/FASTQ files and exact per-k-mer counts matter less than
  scalable, low-footprint membership queries.
when_to_use_negative:
- The goal is exact (false-negative-free) sequence search or alignment scoring — PAC
  is an approximate membership query structure like Cobs or HowDeSBT, not a BLAST-like
  aligner
- Only a handful of small genomes are involved and a simple all-against-all search
  suffices; the comb-tree/partition machinery targets collections of thousands of
  datasets
- Per-k-mer provenance must be exact with zero false positives — larger Bloom filters
  reduce but never eliminate the false positive rate
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: PAC
  role: Constructs, queries, and updates the Partitioned Aggregated Bloom Comb Tree
    index mapping k-mers to datasets
  repo: https://github.com/Malfoy/PAC
- name: g++
  role: C++17 compiler (version 8 or higher) used to build PAC from source
- name: zlib
  role: Compression library dependency enabling PAC to read gzipped FASTA/FASTQ inputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:64325936151c9c81f4c072d3e3d127dd59c58d66b4d40d1d44fb6af814dd3195
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - g++
  - zlib
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# kmer-dataset-membership-indexing

## Summary

Builds an approximate membership query index that maps k-mers (e.g. 21-mers) to the datasets containing them, using PAC's Partitioned Aggregated Bloom Comb Trees so thousands of FASTA/FASTQ files can be queried for sequence membership with low memory and disk footprint.

## When to use

Use when you need to ask 'which indexed dataset(s) may contain this sequence?' over thousands of FASTA/FASTQ files and exact per-k-mer counts matter less than scalable, low-footprint membership queries.

## When NOT to use

- The goal is exact (false-negative-free) sequence search or alignment scoring — PAC is an approximate membership query structure like Cobs or HowDeSBT, not a BLAST-like aligner
- Only a handful of small genomes are involved and a simple all-against-all search suffices; the comb-tree/partition machinery targets collections of thousands of datasets
- Per-k-mer provenance must be exact with zero false positives — larger Bloom filters reduce but never eliminate the false positive rate

## Inputs

- File-of-files (fof.txt) listing dataset paths, one per line (FASTA, MultiLine FASTA, or FASTQ, gzipped or not)
- Query FASTA/FASTQ file for membership search (e.g. GCF_000864885.1_ViralProj15500_genomic.fna.gz)
- Previously built index folder (for -l query or index update modes)

## Outputs

- PAC index folder (e.g. index_test) containing partitioned Aggregated Bloom Comb Tree structures: aggregated Bloom filter run-length arrays and comb tree files split into 4^p partition files
- Query output listing, per query header line, k-mer shares per matching entry (e.g. '>NC_004718.3 ... 29719 50 701')
- Updated index folder when adding a new file-of-file to an existing index

## How to apply

List datasets (FASTA, MultiLine FASTA, or gzipped FASTQ) in a file-of-files, one path per line, then run PAC indexing with the desired k (default k=21), Bloom filter bit budget (e.g. -b 1000000000 rounds up to the first power of two above it, 1,073,741,824), partition count p (splits the index into 4^p files, PAC example p=5), and thread count -c. Optionally pass -u to filter k-mers seen only once and -e 8 to store aggregation values with 8-bit integers (supports up to 256 files; the 16-bit default supports up to 65,536, use -e 32 for more). Internally, PAC builds one Bloom filter per dataset, merges child filters into left/right Bloom comb trees via bitwise OR, and encodes the nested series (Rt⊆…⊆R1) as an Aggregated Bloom filter: a t×b matrix of stacked filters compressed to an integer array V of size b where V[i] is the run length of 1s in row i. Construction is streaming with no disk footprint beyond the index itself; verify by round-trip querying indexed k-mers and confirming every k-mer returns its ground-truth dataset(s) with no false negatives.

## Related tools

- **PAC** (Constructs, queries, and updates the Partitioned Aggregated Bloom Comb Tree index mapping k-mers to datasets) — https://github.com/Malfoy/PAC
- **g++** (C++17 compiler (version 8 or higher) used to build PAC from source)
- **zlib** (Compression library dependency enabling PAC to read gzipped FASTA/FASTQ inputs)

## Examples

```
./PAC -f test/fof.txt -d index_test -k 21 -b 1000000000 -p 5 -c 16 -e 8 ; ./PAC -l index_test -q test/GCF_000864885.1_ViralProj15500_genomic.fna.gz
```

## Evaluation signals

- Round-trip membership check: every indexed k-mer re-queried against the encoded structures returns its ground-truth containing dataset(s) (no false negatives)
- Observed false-positive rate is consistent with the Bloom filter sizing (index sized to the first power of two above the requested bit count, e.g. 1,000,000,000 → 1,073,741,824)
- Index folder contains 4^p partition files (e.g. 4^5 = 1024 for -p 5)
- Query output format matches 'kmer-share counts per matching entry' lines beneath each query header (e.g. '29719 50 701')

## Limitations

- Filter-size trade-off: larger Bloom filters decrease the false positive rate but increase the index size
- Threaded construction can increase memory usage linearly with thread count (-c 16 divides time but raises RAM)
- Aggregation integer width caps the number of indexable files: -e 8 supports up to 256 files, the 16-bit default up to 65,536; more requires -e 32
- Partitioning into 4^p files requires enough open-file descriptors ('ulimit -n 5000' allows 5000 opened files)

## Evidence

- [abstract] PAC, a novel approximate membership query data structure for querying collections of sequence datasets: "Here, we present PAC, a novel approximate membership query data structure for querying collections of sequence datasets."
- [abstract] PAC index construction works in a streaming fashion without any disk footprint besides the index itself: "PAC index construction works in a streaming fashion without any disk footprint besides the index itself."
- [readme] Indexed Bloom filters sized to the first power of two above the requested bit count: "Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824). Larger  Bloom filters decrease false positive rate but increase the index size."
- [readme] PAC will split the index into 4^p files inside the index_test folder: "PAC will split the index into 4^p files inside the index_test folder. Please check that you can open enough file with the ulimit command"
- [readme] PAC will only index kmers seen at least two times in a given dataset: "PAC will only index kmers seen at least two times in a given dataset (kmers seen only once are filtered)"
- [full_text] Aggregated Bloom filter encodes the nested series of Bloom filters along comb tree internal nodes: [evidence span withheld — anchor status: not_found]
