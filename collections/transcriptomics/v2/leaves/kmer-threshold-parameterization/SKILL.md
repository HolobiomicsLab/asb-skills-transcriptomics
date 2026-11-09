---
name: kmer-threshold-parameterization
description: Use when building or benchmarking approximate sequence search pre-filters
  over many bins of nucleotide sequences, and you must choose k (e.g. k=19), minimizer/threshold
  settings, and evaluate sensitivity (false positives) against known bin memberships.
when_to_use_negative:
- The reference collection has highly uneven bin sizes or a very high number of bins,
  since Raptor's method is susceptible to uneven bin sizes and slows down moderately
  at several ten thousand bins
- You need edit-distance-sensitive full-text search rather than approximate k-mer
  membership pre-filtering, where word-based indices are inappropriate but FM-index-based
  approaches are required
- Only one tool is being evaluated — the skill's core value is the controlled cross-tool
  comparison against known bin memberships
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Raptor
  role: Builds the binning directory index using winnowing minimizers, x-partitioned
    interleaved Bloom filters, and probabilistic thresholding; queried as the fast
    pre-filter
  repo: https://github.com/seqan/raptor
- name: COBS
  role: Competing k-mer index built with a 53% k-mer-lemma threshold for the benchmark
    comparison
  repo: https://github.com/bingmann/cobs
- name: Mantis
  role: Competing tool (with Squeaker preprocessing adapted to FASTQ quality scores)
    built at both bin counts for the benchmark; expected to segfault at 64 bins
  repo: https://github.com/splatlab/mantis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:8002510c7c439623bfe4c8f4a83c53714cfa204216893ec103f50afc0d8e7092
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - COBS
  - Mantis
  - Raptor
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# k-mer threshold parameterization for approximate sequence search indices

## Summary

Parameterizes k-mer based search indices (Raptor's binning directory with winnowing minimizers, interleaved Bloom filters, and probabilistic thresholding; COBS with a 53% k-mer-lemma threshold; Mantis) and benchmark-tunes these settings so construction time, search time, memory/disk usage, and false positives can be compared fairly against a known ground truth.

## When to use

Use when building or benchmarking approximate sequence search pre-filters over many bins of nucleotide sequences, and you must choose k (e.g. k=19), minimizer/threshold settings, and evaluate sensitivity (false positives) against known bin memberships.

## When NOT to use

- The reference collection has highly uneven bin sizes or a very high number of bins, since Raptor's method is susceptible to uneven bin sizes and slows down moderately at several ten thousand bins
- You need edit-distance-sensitive full-text search rather than approximate k-mer membership pre-filtering, where word-based indices are inappropriate but FM-index-based approaches are required
- Only one tool is being evaluated — the skill's core value is the controlled cross-tool comparison against known bin memberships

## Inputs

- Binned collection of nucleotide reference sequences (e.g. artificial dataset with 64 and 1,024 bins)
- Query read set with known bin memberships (e.g. 2^20 simulated reads, FASTQ)
- Tool parameters: k-mer size (k=19), COBS k-mer-lemma threshold (53%), Raptor IBF/minimizer settings

## Outputs

- Built indices per tool and bin count (COBS, Mantis, Raptor)
- Comparison table of construction time, search time, peak RAM, disk space, and false-positive counts per tool/bin-count configuration
- Recorded failure modes (e.g. Mantis/Squeaker segmentation fault at 64 bins)

## How to apply

Build each index over the same binned reference collection (e.g. 64 and 1,024 bins) using a fixed k-mer size (k=19 in the Raptor paper) and tool-specific threshold settings: Raptor uses winnowing minimizers with probabilistic thresholding over an extension of interleaved Bloom filters (x-partitioned IBF), while COBS applies its k-mer-lemma threshold (53% in the reproduced Table 4 comparison). Record construction time, peak RAM, and resulting index disk space during each build. Query all indices with an identical query set (2^20 simulated reads in the paper) and count false positives per tool/bin-count configuration against the known bin memberships of the queries. Assemble the measurements into a single comparison table so trade-offs between run time and main memory consumption — and the option to trade run time for memory — are directly visible across tools.

## Related tools

- **Raptor** (Builds the binning directory index using winnowing minimizers, x-partitioned interleaved Bloom filters, and probabilistic thresholding; queried as the fast pre-filter) — https://github.com/seqan/raptor
- **COBS** (Competing k-mer index built with a 53% k-mer-lemma threshold for the benchmark comparison) — https://github.com/bingmann/cobs
- **Mantis** (Competing tool (with Squeaker preprocessing adapted to FASTQ quality scores) built at both bin counts for the benchmark; expected to segfault at 64 bins) — https://github.com/splatlab/mantis

## Examples

```
raptor build --help
```

## Evaluation signals

- The comparison table contains a measurement row for each tool at 64 and 1,024 bins, with Mantis recorded as failing (segmentation fault) at 64 bins
- Raptor search times fall in the reported range (≈0.9 s at 64 bins, ≈1.6 s at 1,024 bins, k=19), i.e. far faster than COBS and Mantis
- COBS/Mantis construction times are at least three times longer than Raptor's construction
- False-positive counts per tool/bin-count configuration are reported against known query bin memberships and are comparable in magnitude to Table 4
- Construction time, peak RAM, and index disk space are recorded for every tool/bin-count combination

## Limitations

- Raptor's method is susceptible to uneven bin sizes; an adaptive IBF for uneven bin sizes was still under development
- Processing several ten thousand bins leads to a moderate slowdown
- P(Cn) has no tractable formulation and must be estimated experimentally by sampling
- Mantis (Squeaker) segfaulted at 64 bins in the reproduced comparison, so not all tool/bin-count configurations may complete
- Indices built with an older version of raptor cannot be used with version 2.0.0, so index and tool versions must be matched

## Evidence

- [abstract] Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for"
- [intro] Our comparison shows that Raptor is up to times faster than the competitors: "compare it with the state-of-the-art methods Mantis and COBS. Our comparison shows that Raptor is up to times faster than the competitors"
- [discussion] The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins.: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
- [methods] Each command will have a respective help page that can be shown via, e.g., `raptor build --help`.: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
