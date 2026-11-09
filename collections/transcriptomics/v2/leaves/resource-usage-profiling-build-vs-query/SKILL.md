---
name: resource-usage-profiling-build-vs-query
description: Use when comparing index-based sequence search tools (e.g., Raptor vs
  Mantis) on a large experiment collection and you must quantify build resources and
  query speed, memory, and specificity per query set.
when_to_use_negative:
- The input is a small collection where exact-search tools like Mantis are already
  fast enough and pre-filter overhead provides no benefit
- The collection has highly uneven bin sizes or a very high number of bins, where
  Raptor's known limitations dominate
- You need edit-distance-sensitive full-text search rather than approximate k-mer/minimizer
  membership pre-filtering
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: Raptor
  role: Builds (20,20)- and (40,20)-minimizer binning directories via 'raptor build'
    and searches query sets via 'raptor search'; its time/RAM/index size are the measured
    quantities
  repo: https://github.com/seqan/raptor
- name: Mantis
  role: Exact-search comparator whose construction cost and query results serve as
    baseline resources and as ground truth for computing Raptor's false-positive rate
  repo: https://github.com/splatlab/mantis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:e0f54d98bf00a9d133bed309ea0d5907598eb4e2c89560554ae52153929c9895
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Mantis
  - Raptor
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# resource-usage-profiling-build-vs-query

## Summary

Benchmark protocol for profiling construction cost (wall-clock time, peak RAM, on-disk index size) and query performance (search time, RAM, specificity/false-positive rate) of sequence-search pre-filter indices, as demonstrated by comparing Raptor binning directories against a Mantis index on 1,742 human RNA-Seq experiments.

## When to use

Use when comparing index-based sequence search tools (e.g., Raptor vs Mantis) on a large experiment collection and you must quantify build resources and query speed, memory, and specificity per query set.

## When NOT to use

- The input is a small collection where exact-search tools like Mantis are already fast enough and pre-filter overhead provides no benefit
- The collection has highly uneven bin sizes or a very high number of bins, where Raptor's known limitations dominate
- You need edit-distance-sensitive full-text search rather than approximate k-mer/minimizer membership pre-filtering

## Inputs

- Collection of nucleotide sequence experiments (human RNA-Seq files, e.g., Zenodo DOI 10.5281/zenodo.1186393)
- Minimizer/binning parameters for each index (e.g., (20,20) and (40,20) minimizers, k=20)
- Randomly sampled query sets of transcripts (100, 1,000, and 10,000 queries)
- Reference/exact-search index results (Mantis) used as ground truth for FPR computation

## Outputs

- Construction resource table: wall-clock build time, peak RAM, and on-disk index size per tool/parameter setting
- Query performance table: per-query-set search time, RAM, and false-positive rate relative to Mantis ground truth

## How to apply

Download the target experiment collection (here the human RNA-seq collection, Zenodo DOI 10.5281/zenodo.1186393) and build each index over the same input: two Raptor binning directories via 'raptor build' with (20,20)- and (40,20)-minimizer schemes at k=20, plus a Mantis index, recording wall-clock time, peak RAM, and on-disk size for every construction. Randomly sample three query sets of 100, 1,000, and 10,000 human transcripts, then search each set against all indices ('raptor search' and Mantis's search command), recording per-set search time and RAM. Treat the exact-search tool's results (Mantis) as ground truth and compute the false-positive rate of each Raptor index at every query scale. Assemble the measurements into two tables — construction resources (time/RAM/space) and query performance (search time, RAM, FPR) — and check them against the paper's reported ranges (e.g., Raptor (40,20) is 12-58x faster than Mantis with FP rate about 0.017).

## Related tools

- **Raptor** (Builds (20,20)- and (40,20)-minimizer binning directories via 'raptor build' and searches query sets via 'raptor search'; its time/RAM/index size are the measured quantities) — https://github.com/seqan/raptor
- **Mantis** (Exact-search comparator whose construction cost and query results serve as baseline resources and as ground truth for computing Raptor's false-positive rate) — https://github.com/splatlab/mantis

## Examples

```
raptor build --help
```

## Evaluation signals

- Construction table entries (time, peak RAM, on-disk size) recorded for every index build are internally consistent and comparable to the paper's benchmark ranges for the same minimizer settings
- Query search time and RAM are reported per query-set scale (100 / 1,000 / 10,000 transcripts) for every index
- False-positive rate is computed against Mantis ground-truth results at every query scale; the (40,20) Raptor index should show a low FPR around 0.017
- Speedup of Raptor over Mantis falls in the reported 12-58x range on the real RNA-Seq dataset
- Index/parameters match exactly what was built (k=20 with (20,20) and (40,20) minimizers); indices from incompatible versions are rejected (pre-2.0.0 indices cannot be used with version 2.0.0)

## Limitations

- The main limiting factor of the method is its susceptibility to uneven bin sizes as well as a high number of bins; processing several ten thousand bins leads to a moderate slowdown
- Compressed bitvector construction requires both uncompressed and compressed bitvectors in main memory at some point, capping memory savings during build
- Indices built with an older version of Raptor cannot be used with version 2.0.0, so tool version must be fixed across the comparison

## Evidence

- [intro] compare it with the state-of-the-art methods Mantis and COBS. Our comparison shows that Raptor is up to times faster than the competitors: "compare it with the state-of-the-art methods Mantis and COBS. Our comparison shows that Raptor is up to times faster than the competitors"
- [intro] memory trade-off: "In addition, we only use a fraction of the memory and can further trade run time for main memory consumption."
- [methods] raptor CLI help: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [discussion] limiting factors: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
- [readme] compilation benefits: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
