---
name: run-length-array-compression
description: Use when indexing many datasets with per-dataset Bloom filters whose
  nested merges along a comb tree must be stored compactly, and per-dataset membership
  must be recoverable from run-length aggregation values.
when_to_use_negative:
- The collection has fewer than two datasets, so no nested series of merged Bloom
  filters exists to aggregate
- The number of indexed files exceeds the integer width chosen for aggregation values
  (e.g. >256 files with -e 8); the wrong width silently truncates membership counts
- Exact (non-approximate) dataset membership listing is required, since Bloom filters
  introduce a false-positive rate
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PAC
  role: Implements bf_merge, the Aggregated Bloom filter run-length encoding, and
    the comb tree construction; -e flag selects aggregation-integer width
  repo: https://github.com/Malfoy/PAC
- name: g++ 8 or higher
  role: C++17 compiler used to build PAC from source
- name: zlib
  role: Dependency for reading gzip-compressed FASTA/FASTQ inputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:8831470bbfa600712531a75938fb2aca50be2bbd494e97c396f5254832ee45dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - g++
  - g++ 8 or higher
  - zlib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# run-length-array-compression

## Summary

Encodes the nested series of Bloom filters (Rt⊆…⊆R1) found along a PAC comb-tree branch as an Aggregated Bloom filter: a t×b stacked matrix M plus an integer array V of size b where V[i] stores the length of the run of 1s in row i, so t Bloom filters of size b collapse to b integers of log t bits each.

## When to use

Use when indexing many datasets with per-dataset Bloom filters whose nested merges along a comb tree must be stored compactly, and per-dataset membership must be recoverable from run-length aggregation values.

## When NOT to use

- The collection has fewer than two datasets, so no nested series of merged Bloom filters exists to aggregate
- The number of indexed files exceeds the integer width chosen for aggregation values (e.g. >256 files with -e 8); the wrong width silently truncates membership counts
- Exact (non-approximate) dataset membership listing is required, since Bloom filters introduce a false-positive rate

## Inputs

- File-of-files (fof.txt) listing one dataset per line, each a FASTA, MultiLine FASTA, or FASTQ file, gzip-compressed or not
- Per-dataset k-mer sets at a chosen k (PAC example: k=21)
- Nested series of Bloom filters produced by bf_merge (bitwise OR) of child filters up the comb tree

## Outputs

- Aggregated Bloom filter: run-length integer array V of size b per nested Bloom-filter series
- Left and right Bloom comb tree structures (Definitions 4-5) partitioned into 4^p prefix files (PAC example p=5, i.e., 4^5 files)
- Paired Aggregated Bloom Comb Tree index folder (Definition 6) with encoded arrays written to disk

## How to apply

Build one Bloom filter per dataset, sized to the first power of two above the requested bit count (e.g. 1,000,000,000 bits rounds to 1,073,741,824). Merge child filters with bf_merge, defined as bitwise OR (Definition 1), to produce the nested series Rt⊆…⊆R1 along the comb tree's longest branch. Encode each series as the run-length array V of the stacked t×b matrix M (Definition 3): for each row i, V[i] records the length of the run of 1s, replacing t Bloom filters of size b with b integers of size log t bits. Choose the aggregation-value width to bound the number of indexable files: -e 8 stores 8-bit integers (up to 256 files), while the 16-bit default supports up to 65,636 files; use -e 32 for larger collections.

## Related tools

- **PAC** (Implements bf_merge, the Aggregated Bloom filter run-length encoding, and the comb tree construction; -e flag selects aggregation-integer width) — https://github.com/Malfoy/PAC
- **g++ 8 or higher** (C++17 compiler used to build PAC from source)
- **zlib** (Dependency for reading gzip-compressed FASTA/FASTQ inputs)

## Examples

```
./PAC -f test/fof.txt -d index_test -e 8
```

## Evaluation signals

- Round-trip membership check: re-query the encoded structures with the originally indexed k-mers and confirm every k-mer returns its ground-truth containing dataset(s) (no false negatives)
- Verify each V[i] equals the length of the run of 1s in row i of the stacked t×b matrix M reconstructed from the leaf filters
- Confirm Bloom filter sizes are the first power of two above the requested bit count (e.g. 1,073,741,824 for 1,000,000,000)
- Check the number of indexed files does not exceed the integer-width limit (256 for -e 8, 65,636 for the 16-bit default)
- Report the observed false-positive rate against the expected rate for the chosen Bloom filter size

## Limitations

- Aggregation-value integer width caps the number of indexable files: 8 bits supports up to 256 files, the 16-bit default up to 65,636 files, requiring -e 32 beyond that
- Larger Bloom filters decrease the false-positive rate but increase index size
- Memory usage during construction can increase linearly with the number of threads used
- More partitions (-p) create more files; the ulimit for open files must be raised (e.g. 'ulimit -n 5000')

## Evidence

- [other] Such a nested series (Rt⊆…⊆R1) is encoded as an Aggregated Bloom filter: a t×b matrix M of the stacked filters plus an integer array V of size b in which V[i] stores the length of the run of 1s in row i, so t Bloom filters of size b are represented using b integers of size S (b×log t bits).: [evidence span withheld — anchor status: not_found]
- [readme] PAC will use 8 bits integers to encode its aggragated bloom filters, this way PAC can index up to 256 files. Bydefault PAC use 16 bits integers and can index up to 65,636 files, if you need to index more files use -e 32: "PAC will use 8 bits integers to encode its aggragated bloom filters, this way PAC can index up to 256 files. Bydefault PAC use 16 bits integers and can index up to 65,636 files, if you need to index"
- [readme] Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824). Larger Bloom filters decrease false positive rate but increase the index size.: "Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824). Larger  Bloom filters decrease false positive rate but increase the index size."
