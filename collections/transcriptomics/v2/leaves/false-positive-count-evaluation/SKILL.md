---
name: false-positive-count-evaluation
description: Use when benchmarking approximate sequence search or binning tools (e.g.,
  Raptor vs. COBS vs. Mantis) where search results are probabilistic and false-positive
  rates must be counted against known ground-truth bin memberships.
when_to_use_negative:
- Queries have unknown bin membership (no ground truth), so false positives cannot
  be identified — use sensitivity against a validated reference tool instead.
- The index returns exact memberships (no probabilistic false positives), making FP
  counting uninformative.
- Datasets have highly uneven bin sizes or very high bin counts, where the paper notes
  Raptor's main limiting factors lie and the comparison would be confounded.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Raptor
  role: Pre-filter under evaluation; built with `raptor build` and queried over artificial
    dataset bins
  repo: https://github.com/seqan/raptor
- name: COBS
  role: Competitor index built with a 53% k-mer-lemma threshold and queried for timing
    and FP comparison
  repo: https://github.com/bingmann/cobs
- name: Mantis
  role: Competitor index built via Squeaker preprocessing (FASTQ quality scores);
    segfaulted at 64 bins in the paper
  repo: https://github.com/splatlab/mantis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:5169a9c9b0ef53557d4a440cd8fa6f85e551554b53fd3802e14dc42414dd5923
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - COBS
  - Mantis
  - Raptor
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# False-positive count evaluation for approximate sequence search indices

## Summary

Quantifies the specificity of pre-filter sequence search tools (Raptor, COBS, Mantis) by querying built indices with reads of known bin membership and counting false positives, alongside construction time, search time, RAM, and disk measurements, as done for the Raper/iScience Table 4 comparison on artificial data.

## When to use

Use when benchmarking approximate sequence search or binning tools (e.g., Raptor vs. COBS vs. Mantis) where search results are probabilistic and false-positive rates must be counted against known ground-truth bin memberships.

## When NOT to use

- Queries have unknown bin membership (no ground truth), so false positives cannot be identified — use sensitivity against a validated reference tool instead.
- The index returns exact memberships (no probabilistic false positives), making FP counting uninformative.
- Datasets have highly uneven bin sizes or very high bin counts, where the paper notes Raptor's main limiting factors lie and the comparison would be confounded.

## Inputs

- Artificial dataset with 64 and 1,024 bins (github.com/seqan/raptor)
- Built indices from Raptor, COBS (53% k-mer-lemma threshold), and Mantis/Squeaker at k=19
- Simulated query read set of 2^20 reads with known bin memberships (ground truth)
- FASTQ query files with quality scores (needed for Squeaker preprocessing in Mantis)

## Outputs

- False-positive counts per tool and bin-count configuration (64 and 1,024 bins)
- Search times per tool per bin count
- Index construction times and peak RAM per tool
- Index disk space usage per tool
- A consolidated comparison table matching the layout of Table 4

## How to apply

Build each tool's index over the same dataset — for the paper's artificial benchmark this means 64 and 1,024 bins with k=19 (COBS with a 53% k-mer-lemma threshold; Mantis with Squeaker preprocessing adapted to FASTQ quality scores). Query every index with the identical set of 2^20 simulated reads so that results are directly comparable. Since the true bin membership of each query is known for the artificial dataset, classify each returned hit as true positive or false positive and count false positives per tool and bin-count configuration. Record construction time, peak RAM, index disk usage, and search time in the same run so all dimensions of the comparison (like Table 4) come from identical inputs and hardware. Treat tool failures as data: in the paper, Mantis (Squeaker) segfaulted at 64 bins, which is recorded as a failure rather than excluded. Validation requires a measurement row per tool per bin count, with magnitudes comparable to the published Table 4 values (e.g., Raptor search ~0.9 s at 64 bins and ~1.6 s at 1,024 bins with k=19, roughly 144x faster than COBS and ~30x faster than Mantis at 1,024 bins).

## Related tools

- **Raptor** (Pre-filter under evaluation; built with `raptor build` and queried over artificial dataset bins) — https://github.com/seqan/raptor
- **COBS** (Competitor index built with a 53% k-mer-lemma threshold and queried for timing and FP comparison) — https://github.com/bingmann/cobs
- **Mantis** (Competitor index built via Squeaker preprocessing (FASTQ quality scores); segfaulted at 64 bins in the paper) — https://github.com/splatlab/mantis

## Examples

```
raptor build --help
```

## Evaluation signals

- A measurement row exists for each tool (Raptor, COBS, Mantis) at both 64 and 1,024 bins, with Mantis recorded as failing (segmentation fault) at 64 bins.
- All tools were queried with the identical 2^20 simulated read set, so search times and FP counts are directly comparable.
- Recorded search times are of the expected magnitude (Raptor ~0.9 s at 64 bins and ~1.6 s at 1,024 bins, k=19; ~144x faster than COBS, ~30x faster than Mantis).
- COBS/Mantis construction times are at least three times longer than Raptor's, consistent with Table 4.
- False-positive counts are computed strictly against the known bin memberships of the queries, not inferred post hoc.

## Limitations

- False positives are only countable on artificial/simulated data where bin memberships of queries are known; real-data benchmarks cannot yield exact FP counts.
- Mantis (Squeaker) segfaulted at 64 bins in the paper, so one cell of the comparison is a failure rather than a measurement.
- Raptor is susceptible to uneven bin sizes and a high number of bins; processing several ten thousand bins leads to a moderate slowdown, which can skew comparative timings.
- Raptor indices built with an older version cannot be used with version 2.0.0, so tool versions must match across the comparison.

## Evidence

- [intro] comparison scope: "compare it with the state-of-the-art methods Mantis and COBS. Our comparison shows that Raptor is up to times faster than the competitors"
- [discussion] limiting factors: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
- [discussion] version incompatibility: "Indices built with an older version of raptor cannot be used with version 2.0.0."
- [methods] build help: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [readme] compile from source: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
