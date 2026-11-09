---
name: super-kmer-extraction
description: Use when building or querying a PAC index and you need k-mers grouped
  by minimizer into super-k-mers, split into 4^p partition files, and resolved via
  per-partition inverted Bloom filters.
when_to_use_negative:
- The collection is small enough that BLAST-like exact search is routine — PAC only
  lists datasets that may contain the query (approximate membership), not alignments
- You need exact containment calls with no false positives — Bloom filter false positives
  are controlled only by sizing -b, not eliminated
- The input should retain k-mers seen only once — the -u option filters singletons,
  so counts relying on unique k-mers are incompatible
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: PAC
  role: Builds the minimizer-partitioned index (4^p files), constructs per-partition
    inverted Bloom filter indexes, and resolves queries over super-k-mers into n-sized
    score vectors
  repo: https://github.com/Malfoy/PAC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:14f39a6dc32a6299a8706b3f4b6100cd802eb7aab1ff623a4c5a02d04aebe8f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - g++ 8 or higher
  - zlib
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# super-kmer-extraction

## Summary

Groups consecutive k-mers sharing a minimizer into super-k-mers and routes them into minimizer-defined partitions of a PAC index, so that querying successive k-mers of a super-k-mer resolves within one small per-partition inverted Bloom filter structure.

## When to use

Use when building or querying a PAC index and you need k-mers grouped by minimizer into super-k-mers, split into 4^p partition files, and resolved via per-partition inverted Bloom filters.

## When NOT to use

- The collection is small enough that BLAST-like exact search is routine — PAC only lists datasets that may contain the query (approximate membership), not alignments
- You need exact containment calls with no false positives — Bloom filter false positives are controlled only by sizing -b, not eliminated
- The input should retain k-mers seen only once — the -u option filters singletons, so counts relying on unique k-mers are incompatible

## Inputs

- FASTA, MultiLine FASTA, or FASTQ dataset files (gzip or plain)
- File-of-files text listing one dataset path per line (e.g. built with 'readlink -f my_folder')
- Query FASTA/FASTQ file whose lines are successively queried k-mer/super-k-mer sources
- Indexing parameters: -k (k-mer size, e.g. 21), -p (partition exponent), -b (Bloom filter bits), -e (integer encoding width), -u (singleton filter)

## Outputs

- Index folder containing exactly 4^p partition files, each an aggregated Bloom Comb holding super-k-mers for that minimizer partition
- Lazily constructed inverted Bloom filter indexes per partition mapping k-mer hash values to bit slices
- Per-query-line output: an n-sized score vector of shared k-mer counts per dataset (e.g. '29719 50 701')

## How to apply

Build the PAC index with a chosen k-mer size (e.g. -k 21) and a partition exponent (-p 2 gives 4^2 = 16 partition files inside the index folder), so each k-mer's minimizer hash determines which partition file its super-k-mer is stored in. Within each partition, a lazily constructed inverted Bloom filter index maps each k-mer hash value h to a bit slice listing which Bloom filters contain it, meaning successive k-mers taken from the same super-k-mer are all resolved inside one small structure instead of walking the whole index. During query, maintain and return a score vector of size n (number of indexed datasets), incrementing position j each time a queried k-mer appears in leaf j, yielding per-dataset shared k-mer counts such as '29719 50 701'. Optionally pass -u to only index k-mers seen at least two times (filtering singletons) and -b to size Bloom filters to the first power of two above the requested bit count, trading index size against false positive rate.

## Related tools

- **PAC** (Builds the minimizer-partitioned index (4^p files), constructs per-partition inverted Bloom filter indexes, and resolves queries over super-k-mers into n-sized score vectors) — https://github.com/Malfoy/PAC

## Examples

```
./PAC -f test/fof.txt -d index_test -k 21 -p 2
```

## Evaluation signals

- After indexing with -p 2, the index folder contains exactly 4^2 = 16 partition files
- Each indexed Bloom filter's size is the first power of two above the requested bit count from -b
- Query output for each line is an n-sized vector of per-dataset shared k-mer counts matching the expected values for the synthetic data (e.g. '29719 50 701')
- In favorable instances, resolving a query's super-k-mer requires only single random access within one partition's structure
- With -e 8 the encoding supports up to 256 indexed files; default 16-bit supports up to 65,636 per the README

## Limitations

- More partitions (-p) speed up construction/query and reduce memory but create more files; check 'ulimit -n' (e.g. 'ulimit -n 5000') allows enough open files
- Threading with -c accelerates construction but memory usage can increase linearly with the number of threads
- Larger Bloom filters decrease false positive rate but increase index size; false positives are inherent to the approximate membership query design
- Default 16-bit integer encoding caps indexing at 65,636 files; use -e 8 for up to 256 files or -e 32 for more

## Evidence

- [readme] PAC will split the index into 4^p files inside the index_test folder.: "PAC will split the index into 4^p files inside the index_test folder."
- [readme] Indexed Bloom filters sized to power of two: "Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824)."
- [methods] Inverted Bloom filter maps k-mer hash to bit slice: [evidence span withheld — anchor status: not_found]
- [readme] Singleton k-mer filtering option: "PAC will only index kmers seen at least two times in a given dataset (kmers seen only once are filtered)"
- [abstract] Query constant-time favorable instances: "A PAC query can need single random access and be performed in constant time in favorable instances."
- [readme] 8-bit encoding caps at 256 files: "PAC will use 8 bits integers to encode its aggragated bloom filters, this way PAC can index up to 256 files."
