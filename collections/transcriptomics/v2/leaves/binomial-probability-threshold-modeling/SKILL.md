---
name: binomial-probability-threshold-modeling
description: Use when computing minimizer-sharing thresholds for (w,k)-minimizer queries
  whose minimizer count varies per read (e.g. m in 15-35), so that random sharing
  can be separated from true matches at significance level 0.99.
when_to_use_negative:
- Queries use fixed-length exact k-mer indexing rather than sequence-dependent (w,k)-minimizers,
  so the minimizer count does not vary per read and a fixed threshold suffices.
- Raptor's susceptibility to uneven bin sizes or a very high number of bins is the
  actual bottleneck — threshold modeling will not address these index-level limits.
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: Raptor
  role: source tool whose probabilistic thresholding model (Equations 1-5, Lemma 2)
    is being reconstructed as a standalone generator; obtained via bioconda or compiled
    from source
  repo: github.com/seqan/raptor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:b99d284f7f17da74b61d0c200d92ef69be855dcafd71f88f77a4e04d154369c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Raptor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# binomial-probability-threshold-modeling

## Summary

Reconstructs Raptor's probabilistic thresholding model (Equations 1-5 and Lemma 2) as a standalone threshold-table generator: a binomial model over the probability that a query minimizer is affected by errors, parameterized by window size w, k-mer size k, number of errors, and the query's minimizer count.

## When to use

Use when computing minimizer-sharing thresholds for (w,k)-minimizer queries whose minimizer count varies per read (e.g. m in 15-35), so that random sharing can be separated from true matches at significance level 0.99.

## When NOT to use

- Queries use fixed-length exact k-mer indexing rather than sequence-dependent (w,k)-minimizers, so the minimizer count does not vary per read and a fixed threshold suffices.
- Raptor's susceptibility to uneven bin sizes or a very high number of bins is the actual bottleneck — threshold modeling will not address these index-level limits.

## Inputs

- window size w and k-mer size k of the (w,k)-minimizer scheme
- maximal number of sequencing errors e
- query length (e.g. 100 bp)
- query minimizer count m (per-read, sequence-dependent)

## Outputs

- threshold_table.csv: thresholds tau over the (w, k, errors, m) parameter grid

## How to apply

Formalize Raptor's Equations 1-5 and Lemma 2 as a binomial model over the probability that a query minimizer is directly affected by errors, parameterized by w, k, and the maximal number of errors. Because the term for indirectly affected minimizers P(Cn) has no tractable closed form (the authors explicitly left it as an open problem), estimate it by Monte Carlo sampling. For each query minimizer count m (tabulated over the 15-35 range), numerically solve for the threshold tau such that a match sharing fewer minimizers is attributable to random sharing at significance level 0.99. Write the results as a threshold table over the (w, k, errors, m) grid to threshold_table.csv.

## Related tools

- **Raptor** (source tool whose probabilistic thresholding model (Equations 1-5, Lemma 2) is being reconstructed as a standalone generator; obtained via bioconda or compiled from source) — github.com/seqan/raptor

## Examples

```
conda install -c bioconda -c conda-forge raptor
```

## Evaluation signals

- Validation against the paper's example: a 100 bp query with 20 minimizers must yield threshold 6.
- All tabulated thresholds must fall within the paper's reported 4-13 range.
- P(Cn) Monte Carlo estimates should be checked for convergence across independent samples before trusting thresholds.
- Schema check: threshold_table.csv must cover the full (w, k, errors, m) grid with m spanning 15-35.
- Thresholds must be monotone in the number of errors e (more tolerated errors -> lower required shared-minimizer count).

## Limitations

- P(Cn) (indirectly affected minimizers) has no tractable formulation; the authors 'did not succeed and leave this as an open problem', so only Monte Carlo estimates exist.
- More involved models for two errors in close vicinity were developed but omitted from the paper.
- The threshold depends on the number and distribution of minimizers, which is sequence-dependent and thus varies per read — a single table entry is only an approximation for a given read.

## Evidence

- [abstract] Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for"
- [discussion] We tried to find a tractable formulation of Cn, but did not succeed and leave this as an open problem.: "We tried to find a tractable formulation of Cn, but did not succeed and leave this as an open problem."
- [readme] Each command will have a respective help page that can be shown via, e.g., `raptor build --help`.: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [readme] The subdirectory `util` contains applications and scripts related to papers.: "The subdirectory `util` contains applications and scripts related to papers."
