---
name: kmer-sharing-count-interpretation
description: Use when running PAC queries against an index of thousands of datasets
  and you need to turn the raw per-line kmer-sharing counts (e.g. '29719 50 701')
  into a ranked list of datasets that may contain the query sequence.
when_to_use_negative:
- You need exact containment calls rather than an approximate membership query — PAC
  lists datasets that may contain a query and can return false positives.
- You are indexing a single dataset or a handful of files where a direct containment
  test is cheaper than building an index.
- Queries are raw reads with many single-occurrence kmers and you did not build the
  index with the -u option, which filters kmers seen only once — shared-count signal
  will be diluted by error kmers.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: PAC
  role: Performs the index query and emits per-dataset shared-kmer count lines that
    this skill interprets; supports multithreaded queries with '-c 16'.
  repo: https://github.com/Malfoy/PAC
- name: seqkit
  role: Used to sample query batches (100, 10,000, 100,000, 500,000 human RefSeq transcripts)
    and write them as FASTA files before querying.
- name: Cobs
  role: Related AMQ structure based on Bloom filters; used as a comparator for PAC's
    query approach.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:1d168e08afd64888c30f59d422d60a600e180aa7ff2d792cdf87e1e642873f6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - Cobs
  - PAC
  - seqkit
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# kmer-sharing-count-interpretation

## Summary

Interpret PAC query output lines, which report per-query kmer-sharing counts between a query sequence and each indexed dataset (e.g. '29719 50 701'), to decide which indexed datasets plausibly contain the query.

## When to use

Use when running PAC queries against an index of thousands of datasets and you need to turn the raw per-line kmer-sharing counts (e.g. '29719 50 701') into a ranked list of datasets that may contain the query sequence.

## When NOT to use

- You need exact containment calls rather than an approximate membership query — PAC lists datasets that may contain a query and can return false positives.
- You are indexing a single dataset or a handful of files where a direct containment test is cheaper than building an index.
- Queries are raw reads with many single-occurrence kmers and you did not build the index with the -u option, which filters kmers seen only once — shared-count signal will be diluted by error kmers.

## Inputs

- PAC index folder previously built over a collection of datasets (e.g. index_test)
- Query FASTA file (plain or gzipped), e.g. batches of 100–500,000 human RefSeq transcript sequences
- Index construction provenance: the file-of-file listing the indexed datasets, which fixes the positional meaning of each output count

## Outputs

- Per-query output lines of shared-kmer counts, one per indexed dataset (format: '<count1> <count2> ...' under each query FASTA header)
- Parsed candidate-dataset list per query, ranking indexed datasets by shared-kmer count
- Optional benchmark table (e.g. pac_query_benchmark_timings.tsv) recording CPU time, wall-clock time, and peak RAM per batch size

## How to apply

Run PAC in query mode ('./PAC -l <index_folder> -q <batch_fasta> -c 16'); for each query FASTA header, PAC emits one line per indexed dataset whose kmer counts are aggregated by the index. Each output line contains the number of kmers the query shares with a given indexed entry — e.g. '29719 50 701' means the SARS-CoV-2 Tor2 genome shares 29,719 kmers with the first entry, 50 with the second, 701 with the third. Parse these counts by position (order of datasets follows the file-of-file used at construction) and treat high shared-kmer counts as candidate-containing datasets, since PAC is an approximate membership query (AMQ) structure that may return false positives but lists datasets that may contain the query. Optionally increase Bloom filter size via '-b' at construction time to lower the false positive rate if shared-count noise from spurious hits is problematic. Batch queries (100 to 500,000 human RefSeq transcripts) can be multithreaded with '-c 16'; validate throughput against the reported envelope (~5 min for 100 transcripts, <1 h for 500,000).

## Related tools

- **PAC** (Performs the index query and emits per-dataset shared-kmer count lines that this skill interprets; supports multithreaded queries with '-c 16'.) — https://github.com/Malfoy/PAC
- **seqkit** (Used to sample query batches (100, 10,000, 100,000, 500,000 human RefSeq transcripts) and write them as FASTA files before querying.)
- **Cobs** (Related AMQ structure based on Bloom filters; used as a comparator for PAC's query approach.)

## Examples

```
./PAC -l index_test -q test/GCF_000864885.1_ViralProj15500_genomic.fna.gz -c 16
```

## Evaluation signals

- Output format check: each query FASTA header is followed by a line of space-separated integer counts, one per indexed dataset, matching the documented format '29719 50 701'.
- Positive control: querying the SARS-CoV-2 Tor2 genome (NC_004718.3) against an index containing it should yield a dominant shared-kmer count (29,719 in the paper's example) for the matching entry.
- Count ordering: the i-th count must correspond to the i-th dataset in the construction file-of-file; misalignment invalidates the candidate list.
- Throughput envelope: benchmark timings should be consistent with ~5 min wall-clock for 100 transcripts, ~40 min wall-clock (~6.5 CPU hours) for 500,000 transcripts, and <1 h completion for the largest batch with up to ~2 GB RAM at 10,000 transcripts.
- False-positive awareness: datasets with low nonzero counts (e.g. 50) should be treated as possible false positives, consistent with the Bloom-filter AMQ design.

## Limitations

- PAC is an approximate membership query structure: it lists datasets that may contain a query, so nonzero shared-kmer counts do not guarantee true containment.
- By default PAC indexes all kmers; single-occurrence kmers (likely sequencing errors) are only filtered if the index was built with the -u option, affecting count interpretability for noisy queries.
- Larger Bloom filters (via -b) lower the false positive rate but increase index size — a size/specificity trade-off to consider when interpreting borderline counts.
- Memory usage during (multithreaded) query operations can increase with thread count; the reported benchmark used 16 threads.

## Evidence

- [readme] 29719 50 701 example meaning: "Meaning that the query share 29719 kmers with the first entry, 50 with the second and 701 with the third."
- [intro] AMQ scope statement: "PAC is a scalable structure able to list datasets that may contain a query sequence among thousands of indexed files."
- [methods] -u filter option: "PAC will only index kmers seen at least two times in a given dataset (kmers seen only once are filtered)"
- [readme] Bloom filter size trade-off: "Larger  Bloom filters decrease false positive rate but increase the index size."
- [abstract] 500k query performance: "We also showed that PAC's ability to query 500 000 transcript sequences in less than an hour."
