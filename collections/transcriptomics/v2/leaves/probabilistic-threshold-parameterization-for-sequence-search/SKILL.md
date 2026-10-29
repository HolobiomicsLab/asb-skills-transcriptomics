---
name: probabilistic-threshold-parameterization-for-sequence-search
description: Use when approximate sequence search needs a significance cutoff on shared
  minimizers — e.g. a query read with a sequence-dependent number of (w,k)-minimizers
  must be judged against random minimizer sharing at a fixed confidence level.
when_to_use_negative:
- The query length and minimizer count are fixed and known a priori so a single hand-computed
  threshold suffices — the table machinery is for sequence-dependent per-read minimizer
  counts.
- The search does not use (w,k)-minimizer representation (e.g. full-text FM-index
  or k-mer-exact tools like COBS/Mantis without winnowing), where the Lemma 2 error
  model does not apply.
- 'You need a provenly exact threshold: the indirectly affected term P(Cn) is only
  estimable by sampling, so results carry Monte Carlo error.'
tools:
- name: Raptor
  role: Source system whose probabilistic thresholding model (Lemma 2) is re-implemented
    as a standalone threshold-table generator; thresholds feed Raptor's minimizer-based
    pre-filter queries
  repo: github.com/seqan/raptor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:d527a28434ec17c78af48477f4210bafe9c8caf6bb6bfcc9b825fe6ee5f26a1f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Raptor
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# probabilistic-threshold-parameterization-for-sequence-search

## Summary

Reconstructs Raptor's probabilistic thresholding model (Equations 1–5 and Lemma 2) as a standalone generator of minimizer-sharing thresholds tau for (w,k)-minimizer queries, parameterized by window size w, k-mer size k, number of errors, and per-query minimizer count m.

## When to use

Use when approximate sequence search needs a significance cutoff on shared minimizers — e.g. a query read with a sequence-dependent number of (w,k)-minimizers must be judged against random minimizer sharing at a fixed confidence level.

## When NOT to use

- The query length and minimizer count are fixed and known a priori so a single hand-computed threshold suffices — the table machinery is for sequence-dependent per-read minimizer counts.
- The search does not use (w,k)-minimizer representation (e.g. full-text FM-index or k-mer-exact tools like COBS/Mantis without winnowing), where the Lemma 2 error model does not apply.
- You need a provenly exact threshold: the indirectly affected term P(Cn) is only estimable by sampling, so results carry Monte Carlo error.

## Inputs

- Window size w and k-mer size k parameters for (w,k)-minimizers
- Maximal number of errors e (indel/substitution budget) per query
- Grid of query minimizer counts m (15–35 as in the paper)
- Significance level (0.99) for the random-sharing test
- Monte Carlo sample size for estimating P(Cn)

## Outputs

- threshold_table.csv with columns (w, k, errors, m, tau) giving the minimizer-sharing threshold per parameter combination
- Monte Carlo estimates of P(Cn) for indirectly affected minimizers

## How to apply

Formalize Raptor's binomial model (Equations 1–5 and Lemma 2) over the probability that a query minimizer is directly affected by errors, parameterized by window size w, k-mer size k, and the number of errors. Because P(Cn) — the probability of indirectly affected minimizers — has no tractable closed form (the authors 'leave this as an open problem'), estimate it by Monte Carlo sampling. For each query minimizer count m in the 15–35 range, numerically solve for the threshold tau such that a match sharing fewer than tau minimizers is attributable to random sharing with significance level 0.99. Tabulate tau over the (w, k, errors, m) grid and validate against the paper's anchor example: a 100 bp query with 20 minimizers must yield threshold 6, with all table values falling in the reported 4–13 range. Write the result as threshold_table.csv for downstream use by the Raptor pre-filter, which uses winnowing minimizers plus an interleaved Bloom filter (IBF) binning directory.

## Related tools

- **Raptor** (Source system whose probabilistic thresholding model (Lemma 2) is re-implemented as a standalone threshold-table generator; thresholds feed Raptor's minimizer-based pre-filter queries) — github.com/seqan/raptor

## Examples

```
raptor build --help
```

## Evaluation signals

- A 100 bp query with 20 minimizers must produce threshold tau = 6 at significance level 0.99.
- All tabulated thresholds fall within the paper's reported range of 4–13.
- The threshold monotonically behaves as expected in w, k, and error count (more errors -> lower required tau; larger m -> larger tau).
- Monte Carlo estimates of P(Cn) are stable across independent runs (low variance across seeds).
- threshold_table.csv covers the full (w, k, errors, m) grid with m spanning 15–35 and has a consistent schema.

## Limitations

- P(Cn), the probability of indirectly affected minimizers, has no tractable formulation — the authors tried and 'leave this as an open problem', so it must be estimated experimentally by sampling.
- Minimizer counts are sequence-dependent, so the model must be evaluated per minimizer count m rather than per query length alone.
- More involved models for two errors in close vicinity were developed but omitted from the paper, so adjacent-error scenarios may be under-modeled.

## Evidence

- [abstract] Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for"
- [discussion] open problem P(Cn): "We tried to find a tractable formulation of Cn, but did not succeed and leave this as an open problem."
- [discussion] two errors close vicinity: "We also developed more involved models to account for two errors being in close vicinity (not shown)"
- [readme] raptor help page: "`raptor --help` will show available commands. Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [methods] raptor install: "See the [documentation](https://docs.seqan.de/raptor/main/setup.html) for detailed information on how to download, install, and compile Raptor."
