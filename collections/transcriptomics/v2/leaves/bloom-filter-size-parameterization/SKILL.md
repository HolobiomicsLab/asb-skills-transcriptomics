---
name: bloom-filter-size-parameterization
description: Use when building a PAC index over a collection of FASTA/FASTQ datasets
  and you must choose the Bloom filter size (-b) trade-off, e.g. 2^27-bit vs 2^29-bit
  filters, balancing false positive rate against index size and RAM.
when_to_use_negative:
- You are querying or updating an already-built PAC index (-l mode) rather than sizing
  a new construction — this skill only governs construction-time filter sizing
- 'Your application cannot tolerate any false positives: PAC is an approximate membership
  query (AMQ) structure and Bloom filter sizing only tunes, never eliminates, the
  FP rate'
- 'Disk budget is fixed and small: raising -b from 2^27 to 2^29 bits grew the 661k-genome
  index from 1.4 TB to 2.4 TB, so oversized filters may be infeasible'
edam_operation: http://edamontology.org/operation_0227
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: PAC
  role: The AMQ data structure whose -b option sizes each indexed Bloom filter to
    the first power of two above the provided value, controlling the false-positive-rate/index-size
    trade-off
  repo: https://github.com/Malfoy/PAC
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:c65aa6e09a751035456f3c1722d5a4c4abae43ba7dee7fe606005cf872ba1543
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - PAC
  - g++ 8 or higher
  - zlib
  edam_operation: http://edamontology.org/operation_0227
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# bloom-filter-size-parameterization

## Summary

Sets the PAC indexed Bloom filter size via the -b parameter, where each filter is rounded up to the first power of two above the given value. This is the key trade-off knob: larger filters (e.g. 2^29 vs 2^27 bits) lower the false positive rate but inflate RAM usage during construction and the on-disk index size.

## When to use

Use when building a PAC index over a collection of FASTA/FASTQ datasets and you must choose the Bloom filter size (-b) trade-off, e.g. 2^27-bit vs 2^29-bit filters, balancing false positive rate against index size and RAM.

## When NOT to use

- You are querying or updating an already-built PAC index (-l mode) rather than sizing a new construction — this skill only governs construction-time filter sizing
- Your application cannot tolerate any false positives: PAC is an approximate membership query (AMQ) structure and Bloom filter sizing only tunes, never eliminates, the FP rate
- Disk budget is fixed and small: raising -b from 2^27 to 2^29 bits grew the 661k-genome index from 1.4 TB to 2.4 TB, so oversized filters may be infeasible

## Inputs

- File-of-files (one dataset path per line, from 'readlink -f') over FASTA, MultiLine FASTA, or FASTQ files, gzipped or not
- Bloom filter size value passed to -b (e.g. 134217728 for 2^27-bit filters)
- Optional -p partition count and -c thread count

## Outputs

- PAC index folder containing 4^p partition files with the sized indexed Bloom filters (e.g. 2.4 TB at 2^29 bits, 1.4 TB at 2^27 bits)
- Resource metrics for the construction run (wall-clock hours, peak RAM, index size), e.g. pac_index_resource_metrics.tsv or construction_resources.tsv

## How to apply

Compile PAC (g++ 8+, zlib), build a file-of-files manifest listing one dataset per line with 'readlink -f', then pass the desired filter size with -b; PAC rounds the value up to the first power of two, so -b 134217728 yields 2^27-bit filters. For whole-GenBank bacterial collections, 2^27-bit filters give a ~4% false positive rate for 5 Mb genomes and produced a 3.5 TB index in 24 h with <5 GB RAM. For a 661,000 bacterial genome collection, 2^29-bit filters gave a 2.4 TB index built in <24 h with 21 GB RAM, while 2^27-bit filters gave a 1.4 TB index with 6 GB RAM. The rationale: larger Bloom filters decrease false positive rate but increase the index size, so pick 2^27 for compactness and 2^29 when fewer false positives justify the extra terabyte. Optionally add -p partitioning and -c threads, remembering memory can grow linearly with thread count.

## Related tools

- **PAC** (The AMQ data structure whose -b option sizes each indexed Bloom filter to the first power of two above the provided value, controlling the false-positive-rate/index-size trade-off) — https://github.com/Malfoy/PAC

## Examples

```
./PAC -f fof.txt -d index_genbank -b 134217728
```

## Evaluation signals

- Index folder contains the expected 4^p partition files for the chosen -p
- On-disk index size scales with the filter size as expected: 2.4 TB at 2^29-bit filters vs 1.4 TB at 2^27-bit filters for the 661k bacterial genome collection
- Construction RAM tracks the filter size (21 GB at 2^29 vs 6 GB at 2^27) and both builds finish within 24 h
- At 2^27-bit filters the false positive rate is ~4% for 5 Mb genomes, matching the published GenBank behaviour
- Metrics table rows reproduce the published figures (21 GB vs 6 GB RAM; 2.4 TB vs 1.4 TB; 3.5 TB, <5 GB RAM, 410 CPU hours for GenBank)

## Limitations

- Larger Bloom filters decrease false positive rate but increase the index size, so the parameter is always a trade-off, not a free improvement
- Memory usage during construction can increase linearly with the number of threads (-c), compounding the RAM cost of larger filters
- The integer encoding (-e) caps the number of indexable files (16-bit default: 65,636 files), a separate parameterization decision from filter size

## Evidence

- [readme] Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824). Larger Bloom filters decrease false positive rate but increase the index size.: "Indexed Bloom filters will be sized to the first power of two above 1000000000 (1,073,741,824). Larger Bloom filters decrease false positive rate but increase the index size."
- [abstract] the entire GenBank bacterial genome collection in a single day for an index size of 3.5 TB: "the entire GenBank bacterial genome collection in a single day for an index size of 3.5 TB"
