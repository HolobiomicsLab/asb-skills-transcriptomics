---
name: minimizer-sharing-probability-estimation
description: Use when building or recalibrating a pre-filter over (w,k)-minimizer
  matches (e.g. an IBF binning directory) and you need, for each query minimizer count
  m, the tau distinguishing true error-tolerant matches from random sharing.
when_to_use_negative:
- The query uses w<=k minimizers (the model is derived for (w,k)-minimizers with w>k)
- You need a pre-built index/search tool rather than threshold calibration — use the
  raptor build/search commands directly
- The input is bin-packing or clustering of sequences into bins — that is TaxSBP's
  role, not threshold estimation
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Raptor
  role: Reference implementation of winnowing minimizers and probabilistic thresholding;
    source of Equations 1–5 and Lemma 2 being reconstructed
  repo: github.com/seqan/raptor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:4109a7aa69a747959b9510991355369a0f453f40ca756e28d37638edd8279095
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Raptor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# minimizer-sharing-probability-estimation

## Summary

Reconstructs Raptor's probabilistic thresholding model (Equations 1–5 and Lemma 2) as a standalone generator of minimizer-sharing thresholds for (w,k)-minimizers with w>k, where the number of query minimizers is sequence-dependent. It computes, per maximal error count, the threshold tau below which a match is attributable to random minimizer sharing.

## When to use

Use when building or recalibrating a pre-filter over (w,k)-minimizer matches (e.g. an IBF binning directory) and you need, for each query minimizer count m, the tau distinguishing true error-tolerant matches from random sharing.

## When NOT to use

- The query uses w<=k minimizers (the model is derived for (w,k)-minimizers with w>k)
- You need a pre-built index/search tool rather than threshold calibration — use the raptor build/search commands directly
- The input is bin-packing or clustering of sequences into bins — that is TaxSBP's role, not threshold estimation

## Inputs

- Parameter grid: window size w, k-mer size k (with w>k), maximal number of errors
- Query minimizer count m in the range 15–35 (e.g. a 100 bp query with 20 minimizers)
- Significance level 0.99 for the random-sharing hypothesis

## Outputs

- threshold_table.csv: thresholds tau over the (w, k, errors, m) parameter grid
- Estimated P(Cn) values per parameter setting (Monte Carlo)

## How to apply

Formalize Equations 1–5 and Lemma 2 of the Raptor paper as a binomial model over the probability that a query minimizer is directly affected by errors, parameterized by window size w, k-mer size k, and number of errors. Estimate the indirectly affected minimizer term P(Cn) by Monte Carlo sampling, since the authors state no tractable closed form exists and leave it as an open problem. For each query minimizer count m in the 15–35 range, numerically solve for the threshold tau such that a match sharing fewer minimizers is attributable to random sharing at significance level 0.99. Tabulate thresholds over the (w, k, errors, m) grid and write the result to threshold_table.csv.

## Related tools

- **Raptor** (Reference implementation of winnowing minimizers and probabilistic thresholding; source of Equations 1–5 and Lemma 2 being reconstructed) — github.com/seqan/raptor

## Examples

```
conda install -c bioconda -c conda-forge raptor && python scripts/threshold_model.py --w 23 --k 19 --errors 0 --m 15-35 --out threshold_table.csv
```

## Evaluation signals

- A 100 bp query with 20 minimizers must yield threshold tau = 6 for the paper's example parameter setting
- All computed thresholds for the validated grid fall within the paper's reported 4–13 range
- Monte Carlo estimates of P(Cn) stabilize (low variance across repeated runs) for each (w, k, errors) setting
- Solving for tau at significance level 0.99 must be monotone in the number of errors (more errors -> lower threshold)
- Rows in threshold_table.csv cover the full m = 15–35 range without gaps

## Limitations

- P(Cn) has no tractable formulation; the authors 'tried to find a tractable formulation of Cn, but did not succeed and leave this as an open problem', so it must be estimated experimentally by sampling
- More involved models accounting for two errors in close vicinity were developed but omitted from the paper (not shown)
- The model's applicability inherits Raptor's sensitivity limits, e.g. susceptibility to uneven bin sizes in the underlying binning directory

## Evidence

- [abstract] Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom Filters (IBFs) as a set membership data structure and probabilistic thresholding for: [evidence span withheld — anchor status: not_found]
- [discussion] We tried to find a tractable formulation of Cn, but did not succeed and leave this as an open problem.: "We tried to find a tractable formulation of Cn, but did not succeed and leave this as an open problem."
- [readme] Each command will have a respective help page that can be shown via, e.g., `raptor build --help`.: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
