---
name: minimizer-partition-indexing
description: Use when needing to index thousands of FASTA/FASTQ sequence datasets
  into an approximate membership query (AMQ) structure and list which datasets may
  contain a query sequence, e.g. large collections like SRA or GenBank bacterial genomes.
when_to_use_negative:
- When exact (not approximate) membership answers are required — PAC is an AMQ structure
  with false positives controlled by Bloom filter size
- When the query goal is alignment/scoring of sequences rather than listing which
  indexed datasets may contain the query sequence (BLAST-like methods remain the reference
  for that)
- When only a handful of small datasets must be indexed — PAC targets thousands of
  indexed files; simpler structures may suffice
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: PAC
  role: 'Primary tool: builds the partitioned aggregated Bloom comb tree index and
    runs single/multiple queries against it'
  repo: https://github.com/Malfoy/PAC
- name: g++ 8 or higher
  role: C++17-ready compiler required to build PAC from source via make
- name: zlib
  role: Compression library dependency (zlib1g-dev on Debian/Ubuntu) needed to compile
    PAC and read gzipped FASTA/FASTQ inputs
- name: Cobs
  role: Compared Bloom-filter-based AMQ structure that PAC's approach builds upon;
    not executed as part of this skill
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:1fe0f246151b897a4c1d4f081046b63b89e4121df68e46bff71d80c7178e6f3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - Cobs
  - PAC
  - g++ 8 or higher
  - zlib
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# minimizer-partition-indexing

## Summary

Reconstructs PAC's (Partitioned Aggregated Bloom Comb trees) indexing scheme: a k-mer set split by minimizers into 4^p partition files, each holding an aggregated Bloom comb filter with lazily constructed inverted Bloom filter indexes, plus the query engine that resolves single and batched sequence queries against the partitions.

## When to use

Use when needing to index thousands of FASTA/FASTQ sequence datasets into an approximate membership query (AMQ) structure and list which datasets may contain a query sequence, e.g. large collections like SRA or GenBank bacterial genomes.

## When NOT to use

- When exact (not approximate) membership answers are required — PAC is an AMQ structure with false positives controlled by Bloom filter size
- When the query goal is alignment/scoring of sequences rather than listing which indexed datasets may contain the query sequence (BLAST-like methods remain the reference for that)
- When only a handful of small datasets must be indexed — PAC targets thousands of indexed files; simpler structures may suffice

## Inputs

- File-of-files (fof.txt) listing one FASTA/FASTQ dataset per line (gzip optional)
- Query FASTA or FASTQ file whose lines are queried against the index
- Parameters: k-mer size (-k, e.g. 21), partition count (-p), Bloom filter bit size (-b), integer encoding width (-e, default 16 bits)
- Previously built index folder (for query -l or update -l with -d)

## Outputs

- Index folder containing exactly 4^p partition files, each holding an aggregated Bloom comb filter with lazily constructed inverted Bloom filter indexes
- Per-query-line output: an n-sized bit vector of per-dataset hits and a score vector of shared k-mer counts (e.g. '29719 50 701' meaning the query shares 29719 kmers with the first entry, 50 with the second, 701 with the third)
- Updated index folder when adding a new file of file to a previously built index

## How to apply

Build PAC from source with a C++17-capable compiler (g++ 8 or higher) and zlib installed, then index a file-of-files listing one dataset per line (FASTA, multi-line FASTA, or FASTQ, gzipped or not) with a chosen k (e.g. -k 21) and partition count p so the index folder holds 4^p partition files (e.g. -p 2 gives 16 files), each containing an aggregated Bloom comb filter sized to the first power of two above the requested bit count (-b). The index uses 16-bit integers by default to encode up to 65,636 files; use -e 8 for up to 256 files or -e 32 for more. Optionally filter kmers seen only once with -u and parallelize with -c. At query time, per partition a lazily constructed Inverted Bloom filter maps each k-mer hash value h to the bit slice listing which Bloom filters contain it, so querying successive k-mers from a super-k-mer stays within one small structure; the query maintains and returns a score vector of size n incremented at position j each time a k-mer appears in leaf j, yielding per-dataset shared k-mer counts (e.g. '29719 50 701').

## Related tools

- **PAC** (Primary tool: builds the partitioned aggregated Bloom comb tree index and runs single/multiple queries against it) — https://github.com/Malfoy/PAC
- **g++ 8 or higher** (C++17-ready compiler required to build PAC from source via make)
- **zlib** (Compression library dependency (zlib1g-dev on Debian/Ubuntu) needed to compile PAC and read gzipped FASTA/FASTQ inputs)
- **Cobs** (Compared Bloom-filter-based AMQ structure that PAC's approach builds upon; not executed as part of this skill)

## Examples

```
./PAC -f test/fof.txt -d index_test -k 21 -p 2 -c 16 && ./PAC -l index_test -q test/GCF_000864885.1_ViralProj15500_genomic.fna.gz -c 16
```

## Evaluation signals

- The index folder contains exactly 4^p partition files (e.g. 16 files with -p 2)
- Query output reports the expected per-dataset shared k-mer counts (e.g. a line like '29719 50 701') matching the synthetic sequences indexed
- Index construction leaves no disk footprint besides the index itself (streaming construction)
- False positive rate is controlled by the chosen Bloom filter size (first power of two above the requested bit count)
- With default 16-bit encoding the index supports up to 65,636 files; switching to -e 8 or -e 32 should be reflected in the number of indexable files

## Limitations

- Memory usage can increase linearly with the number of threads used (-c)
- More partitions (-p) create more files; verify ulimit -n allows enough open files (e.g. ulimit -n 5000 for many partitions)
- Bloom filter false positives: larger -b values reduce FPR but increase index size
- Default 16-bit integer encoding caps the index at 65,636 files; more requires -e 32

## Evidence

- [methods] Per partition, a lazily constructed Inverted Bloom filter index maps each k-mer hash value h to the bit slice listing which Bloom filters contain it, so that querying successive k-mers from a super-k-mer stays within one small structure: [evidence span withheld — anchor status: not_found]
- [methods] the query maintains and returns a score vector of size n incremented at position j each time a k-mer appears in leaf j: [evidence span withheld — anchor status: not_found]
- [readme] PAC will construct an index in the folder index_test using all the datasets containend in the file of file fof.txt.: "PAC will construct an index in the folder index_test using all the datasets containend in the file of file fof.txt."
- [readme] PAC will use 8 bits integers to encode its aggragated bloom filters, this way PAC can index up to 256 files.: "PAC will use 8 bits integers to encode its aggragated bloom filters, this way PAC can index up to 256 files."
- [readme] Meaning that the query share 29719 kmers with the first entry, 50 with the second and 701 with the third.: "Meaning that the query share 29719 kmers with the first entry, 50 with the second and 701 with the third."
